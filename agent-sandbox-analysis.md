# Kubernetes Agent Sandbox 深度技术分析

> 作者: Claude | 日期: 2025-03-24

---

## 目录

- [为什么需要 Agent Sandbox](#为什么需要-agent-sandbox)
- [整体架构](#整体架构)
- [CRD 设计 (API层)](#crd 设计 api层)
- [核心控制器流程](#核心控制器流程)
- [数据流动](#数据流动)
- [生命周期管理](#生命周期管理)
- [关键设计决策](#关键设计决策)
- [Extensions 扩展机制](#extensions-扩展机制)
- [与阿里 OpenSandbox 对比](#与阿里-opensandbox-对比)
- [总结](#总结)

---

## 为什么需要 Agent Sandbox

`★ Insight ─────────────────────────────────────`

**背景痛点：**

K8s 原生资源无法满足 AI Agent 场景：

- **Deployment**: 无状态副本模型，不适合长生命周期
- **StatefulSet**: 编号固定(Pod-0, Pod-1)，不适合 singleton 场景
- **Agent 需求**: 稳定身份 + 持久存储 + 生命周期管理 + 快速启停

**解决方案：**

设计一个 `Sandbox` CRD，提供：
- 稳定的主机名和网络身份
- 持久化存储
- 生命周期管理（创建、过期删除、暂停、恢复）
- 预热池（WarmPool）实现毫秒级启动

`─────────────────────────────────────────────────`

---

## 整体架构

### 架构图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           User / Agent                                       │
│                   (Create Sandbox/SandboxClaim)                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Kubernetes API Server                                  │
│  ┌──────────────┐  ┌──────────────────┐  ┌─────────────────────────────┐ │
│  │    Sandbox   │  │  SandboxTemplate │  │      SandboxClaim             │ │
│  │   (Core)     │  │   (Extension)   │  │       (Extension)             │ │
│  └──────────────┘  └──────────────────┘  └─────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌─────────────────────────────────┐   ┌─────────────────────────────────────────┐
│   Sandbox Controller            │   │   SandboxClaim Controller               │
│   (Core)                       │   │   (Extension)                           │
│                                │   │                                         │
│  • reconcileChildResources()   │   │  • Template引用解析                     │
│  • reconcilePod()              │   │  • Sandbox资源创建                      │
│  • reconcileService()          │   │  • WarmPool Pod adoption               │
│  • reconcilePVCs()            │   │                                         │
└─────────────────────────────────┘   └─────────────────────────────────────────┘
                    │                               │
                    ▼                               ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Kubernetes Resources                                  │
│  ┌─────────┐   ┌──────────┐   ┌───────────┐   ┌────────────────────────┐  │
│  │   Pod   │◄──│ Service  │   │    PVC    │   │  SandboxWarmPool       │  │
│  │ (Agent) │   │(Headless)│   │(Persistent)│   │  (Pre-warmed Pods)    │  │
│  └─────────┘   └──────────┘   └───────────┘   └────────────────────────┘  │
│       │                                                      ▲              │
│       │                                                      │              │
│       └──────────────────────────────────────────────────────┘              │
│                     (Optional: Adopt from WarmPool)                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 类型 | 职责 |
|------|------|------|
| **Sandbox** | Core CRD | 核心资源，管理单个有状态的工作负载 |
| **SandboxTemplate** | Extension CRD | 可重用的 Sandbox 配置模板 |
| **SandboxClaim** | Extension CRD | 声明式创建 Sandbox 的高级 API |
| **SandboxWarmPool** | Extension CRD | 预热 Pod 池，实现快速分配 |
| **Sandbox Controller** | Controller | 核心控制器，管理 Pod/Service/PVC |
| **SandboxClaim Controller** | Controller | 处理 Claim 到 Sandbox 的转换 |

---

## CRD 设计 (API层)

### Core: Sandbox CRD

```yaml
apiVersion: agents.x-k8s.io/v1alpha1
kind: Sandbox
metadata:
  name: my-agent-sandbox
spec:
  replicas: 1                    # 只能是 0 或 1 (singleton)
  podTemplate:                   # Pod 模板
    spec:
      containers:
      - name: agent
        image: agent-image:latest
  volumeClaimTemplates:          # 持久化存储
  - metadata:
      name: workspace
    spec:
      resources:
        requests:
          storage: 10Gi
  lifecycle:
    shutdownTime: "2025-01-01T00:00:00Z"  # 过期时间
    shutdownPolicy: Retain       # Delete | Retain
```

#### 核心字段说明

| 字段 | 类型 | 作用 | 约束 |
|------|------|------|------|
| `replicas` | `*int32` | 限制为 singleton | 只能是 0 或 1 |
| `podTemplate` | `PodTemplate` | 定义 Agent 运行环境 | 必填 |
| `volumeClaimTemplates` | `[]PVC` | 持久化存储 | 可选 |
| `lifecycle.shutdownTime` | `*metav1.Time` | 过期时间 | 可选 |
| `lifecycle.shutdownPolicy` | `ShutdownPolicy` | Delete 删除 / Retain 保留 | 可选，默认 Retain |

#### Status 状态

```go
type SandboxStatus struct {
    ServiceFQDN string            // 完整域名: sandbox-name.ns.svc.cluster.local
    Service   string              // Service 名称
    Conditions []metav1.Condition // 状态条件 (Ready)
    Replicas  int32               // 实际副本数 (0 或 1)
    LabelSelector string          // Pod 选择器
}
```

### Extensions: SandboxClaim CRD

```yaml
apiVersion: agents.x-k8s.io/v1alpha1
kind: SandboxClaim
metadata:
  name: user-claim-001
spec:
  templateRef:
    name: agent-template      # 引用 SandboxTemplate
  # 或直接内联 SandboxSpec
```

**作用**: 提供声明式的高级 API，用户只需引用模板，控制器自动创建 Sandbox

### Extensions: SandboxTemplate CRD

```yaml
apiVersion: agents.x-k8s.io/v1alpha1
kind: SandboxTemplate
metadata:
  name: agent-template
template:
  spec:
    replicas: 1
    podTemplate: ...
    volumeClaimTemplates: ...
```

**作用**: 模板化配置，支持大规模复用

### Extensions: SandboxWarmPool CRD

```yaml
apiVersion: agents.x-k8s.io/v1alpha1
kind: SandboxWarmPool
metadata:
  name: warm-pool-001
spec:
  replicas: 3                 # 预热 Pod 数量
  template:                   # Sandbox 模板
    spec:
      podTemplate: ...
```

**作用**: 预创建 Pod，实现**快速分配**（毫秒级）

---

## 核心控制器流程

### Reconciliation Loop

```go
// 简化版 Reconcile 流程
func (r *SandboxReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // 1. 获取 Sandbox 对象
    sandbox := &sandboxv1alpha1.Sandbox{}
    r.Get(ctx, req.NamespacedName, sandbox)

    // 2. 检查是否被删除
    if !sandbox.DeletionTimestamp.IsZero() {
        return ctrl.Result{}, nil  // 正在删除，不做处理
    }

    // 3. 检查是否已过期
    if sandboxMarkedExpired(sandbox) {
        return ctrl.Result{}, nil  // 已过期，不做处理
    }

    // 4. 检查过期时间
    expired, requeueAfter := checkSandboxExpiry(sandbox)
    if expired {
        return r.handleSandboxExpiry(ctx, sandbox)
    }

    // 5. 调谐子资源 (Pod/Service/PVC)
    err = r.reconcileChildResources(ctx, sandbox)

    // 6. 更新状态
    r.updateStatus(ctx, oldStatus, sandbox)

    return ctrl.Result{RequeueAfter: requeueAfter}, err
}
```

### 子资源调谐流程

```
reconcileChildResources()
    │
    ├──► reconcilePVCs()
    │        │
    │        └── 遍历 volumeClaimTemplates
    │              ├── Get PVC (已存在?) → Continue
    │              └── Create PVC (不存在)
    │
    ├──► reconcilePod()
    │        │
    │        ├── CASE 1: replicas == 0
    │        │        └── Delete Pod
    │        │
    │        ├── CASE 2: Pod 已存在
    │        │        ├── Update labels/annotations
    │        │        ├── Set ControllerReference
    │        │        └── Return existing Pod
    │        │
    │        └── CASE 3: 创建新 Pod
    │                 ├── 生成 nameHash (FNV-1a)
    │                 ├── 注入 label: agents.x-k8s.io/sandbox-name-hash
    │                 ├── 注入 volume mounts
    │                 └── Create Pod
    │
    └──► reconcileService()
             │
             └── 创建 Headless Service (ClusterIP: None)
                   └── Selector: sandboxLabel = nameHash
```

### Pod 创建核心逻辑

```go
func (r *SandboxReconciler) reconcilePod(ctx context.Context, sandbox *sandboxv1alpha1.Sandbox, nameHash string) (*corev1.Pod, error) {
    // 1. 根据 nameHash 查询已有 Pod
    labelSelector := labels.SelectorFromSet(labels.Set{
        sandboxLabel: nameHash,
    })
    r.List(ctx, podList, &client.ListOptions{LabelSelector: labelSelector})

    // 2. CASE: replicas == 0，删除 Pod
    if *sandbox.Spec.Replicas == 0 {
        return nil, r.Delete(ctx, pod)
    }

    // 3. CASE: Pod 已存在，更新并返回
    if pod != nil {
        // 更新 labels
        pod.Labels[sandboxLabel] = nameHash
        // 设置 OwnerReference
        ctrl.SetControllerReference(sandbox, pod, r.Scheme)
        return pod, nil
    }

    // 4. CASE: 创建新 Pod
    labels := map[string]string{
        sandboxLabel: nameHash,
    }
    // 注入 volume
    for _, pvcTemplate := range sandbox.Spec.VolumeClaimTemplates {
        pvcName := pvcTemplate.Name + "-" + sandbox.Name
        mutatedSpec.Volumes = append(mutatedSpec.Volumes, corev1.Volume{
            Name: pvcTemplate.Name,
            VolumeSource: corev1.VolumeSource{
                PersistentVolumeClaim: &corev1.PersistentVolumeClaimVolumeSource{
                    ClaimName: pvcName,
                },
            },
        })
    }

    return r.Create(ctx, pod)
}
```

---

## 数据流动

### 创建流程

```
User/App
   │
   ▼
POST /api/v1/namespaces/{ns}/sandboxes
   │
   ▼
Kubernetes API Server
   │
   ▼ (Watch Event)
Sandbox Controller
   │
   ├─► 1. Create PVC (if defined)
   │      - Name: {pvcTemplate.Name}-{sandbox.Name}
   │      - OwnerReference: Sandbox
   │
   ├─► 2. Create Pod
   │      - Name: sandbox.Name
   │      - Labels: agents.x-k8s.io/sandbox-name-hash={hash}
   │      - Volumes: from PVC templates
   │      - OwnerReference: Sandbox
   │
   ├─► 3. Create Headless Service
   │      - Name: sandbox.Name
   │      - ClusterIP: None
   │      - Selector: agents.x-k8s.io/sandbox-name-hash={hash}
   │      - OwnerReference: Sandbox
   │
   └─► 4. Update Sandbox Status
          - Status.Service = "sandbox-name"
          - Status.ServiceFQDN = "sandbox-name.ns.svc.cluster.local"
          - Status.Replicas = 1
          - Status.Conditions[Ready] = True
```

### 访问流程

```
Agent / External Client
   │
   ▼
DNS: {sandbox-name}.{namespace}.svc.cluster.local
   │
   ▼
Kubernetes Service (Headless)
   │
   ▼
Sandbox Pod (Agent Container)
   │
   ├──► Agent Process (Claude Code, Codex, etc.)
   │
   └──► Persistent Volume (workspace)
```

---

## 生命周期管理

### 状态机

```
                    ┌─────────────┐
                    │  Creating   │
                    └──────┬──────┘
                           │ Pod Created
                           ▼
                    ┌─────────────┐
          ┌────────►│   Running   │◄────────┐
          │         └──────┬──────┘         │
   Pod Deleted│                │            │ Pod Deleted
          │         ┌─────────┴─────────┐    │
          │         │                   │    │
          │    Ready=True          Ready=False
          │         │                   │    │
          │         └─────────┬─────────┘    │
          │                   │             │
          │                   ▼             │
          │            ┌─────────────┐       │
          │            │   Running   │       │
          │            │  (Ready)   │       │
          │            └──────┬──────┘       │
          │                   │               │
          │            shutdownTime reached   │
          │                   │               │
          │                   ▼               │
          │            ┌─────────────┐        │
          └────────────│   Expired   │────────┘
                       └─────────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
         shutdownPolicy=Delete      shutdownPolicy=Retain
                │                           │
                ▼                           ▼
         Resource Cleanup           Mark Expired + Keep
                │                           │
                └─────────────┬─────────────┘
                              │
                              ▼
                       Final State
```

### 过期检查逻辑

```go
func checkSandboxExpiry(sandbox *sandboxv1alpha1.Sandbox) (bool, time.Duration) {
    // 无过期时间
    if sandbox.Spec.ShutdownTime == nil {
        return false, 0
    }

    // 已过期
    if time.Now().After(sandbox.Spec.ShutdownTime.Time) {
        return true, 0
    }

    // 计算下次检查时间
    remainingTime := time.Until(sandbox.Spec.ShutdownTime.Time)
    // 至少 2 秒后再次检查
    return false, max(remainingTime, 2*time.Second)
}
```

### 过期处理逻辑

```go
func (r *SandboxReconciler) handleSandboxExpiry(ctx context.Context, sandbox *sandboxv1alpha1.Sandbox) (bool, error) {
    // 1. 删除 Pod
    r.Delete(ctx, pod)

    // 2. 删除 Service
    r.Delete(ctx, service)

    // 3. 根据策略处理 Sandbox 资源
    if *sandbox.Spec.ShutdownPolicy == ShutdownPolicyDelete {
        // 删除 Sandbox 资源本身
        r.Delete(ctx, sandbox)
        return true, nil  // 标记为已删除
    }

    // 4. Retain 策略：标记为过期
    meta.SetStatusCondition(&sandbox.Status.Conditions, metav1.Condition{
        Type:               string(SandboxConditionReady),
        Status:             metav1.ConditionFalse,
        Reason:             SandboxReasonExpired,
        Message:            "Sandbox has expired",
    })

    return false, nil  // 未删除，仅标记
}
```

---

## 关键设计决策

| 决策点 | 实现方式 | 原因 |
|--------|----------|------|
| **Pod 命名** | `sandbox.Name` 直接作为 Pod Name | 简化 DNS 访问，保持稳定身份 |
| **Service** | Headless (ClusterIP: None) | 支持 DNS 定点访问，无需负载均衡 |
| **Label 映射** | FNV-1a Hash 生成 `sandboxLabel` | 避免特殊字符问题，支持查询 |
| **OwnerReference** | Pod/Service/PVC 指向 Sandbox | 级联删除，垃圾回收 |
| **PVC 命名** | `{pvcTemplate.Name}-{sandbox.Name}` | 保证唯一性，支持关联 |
| **Replicas** | 限制 0 或 1 | 强制 singleton 语义 |
| **ControllerReference** | 所有子资源设置所有者 | 确保生命周期同步 |

### NameHash 生成算法

```go
func NameHash(objectName string) string {
    h := fnv.New32a()
    h.Write([]byte(objectName))
    return fmt.Sprintf("%08x", h.Sum32())
}
```

示例：
- `my-sandbox` → `8a2f8c1d`
- `agent-001` → `7b3e2a4f`

---

## Extensions 扩展机制

### WarmPool 快速分配

```
┌─────────────────┐         ┌─────────────────┐
│ SandboxClaim    │         │ SandboxWarmPool │
│                 │         │                 │
│ spec:           │         │ spec:           │
│   templateRef:  │────────►│   replicas: 5   │
│     name: tmpl  │ adopts  │   template: ... │
└─────────────────┘         └────────┬────────┘
                                     │
                    ┌────────────────┴────────────────┐
                    │     Pre-warmed Pods               │
                    │  ┌─────┐ ┌─────┐ ┌─────┐          │
                    │  │Pod-1│ │Pod-2│ │Pod-3│ ...      │
                    │  └─────┘ └─────┘ └─────┘          │
                    └─────────────────────────────┘
```

#### WarmPool 工作流程

1. **预热阶段**: WarmPool Controller 持续维护 N 个预创建的 Pod
2. **请求阶段**: 用户创建 SandboxClaim
3. **分配阶段**: Claim Controller 从 WarmPool 中选取一个 Pod
4. ** Adopt 阶段**: 修改 Pod 的 OwnerReference，从 WarmPool 转向 Sandbox
5. **就绪阶段**: Pod 已被使用，不再参与预热

**优势**:
- 预创建 Pod，用户请求时**直接 adopt** 无需等待镜像拉取
- 冷启动从分钟级降至毫秒级
- 减少资源浪费，提高利用率

---

## 与阿里 OpenSandbox 对比

| 特性 | kubernetes-sigs/agent-sandbox | Alibaba/OpenSandbox |
|------|-------------------------------|---------------------|
| **架构** | K8s CRD + Controller | 独立 Server + SDK |
| **隔离** | 依赖底层运行时 (gVisor/Kata) | 内置多种运行时 |
| **复杂度** | 中 (需 K8s 集群) | 低 (Docker/K8s 皆可) |
| **适用场景** | K8s 原生集成 | 多云/混合部署 |
| **SDK** | Python | Python/JS/Java/C# |
| **部署方式** | Kubernetes Operator | Docker / K8s |
| **多租户** | 原生支持 (Namespace + RBAC) | 需要额外配置 |
| **维护者** | Kubernetes SIG Apps | 阿里云 |

### 架构差异图

```
┌─────────────────────────────────────────────────────────┐
│           kubernetes-sigs/agent-sandbox                 │
│                                                         │
│   User → Sandbox CRD → API Server → Controller         │
│                            │                            │
│                            ▼                            │
│                      Pod / Service / PVC               │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                 Alibaba/OpenSandbox                     │
│                                                         │
│   User → SDK (Python/JS/Java) → REST API → Server      │
│                            │                            │
│                            ▼                            │
│                 Docker Container / K8s Pod              │
└─────────────────────────────────────────────────────────┘
```

### 选型建议

| 场景 | 推荐方案 |
|------|----------|
| 已有 K8s 集群 | kubernetes-sigs/agent-sandbox |
| 需要多语言 SDK | Alibaba/OpenSandbox |
| 轻量级/本地开发 | Alibaba/OpenSandbox |
| 需要强隔离 (gVisor/Kata) | 两者都支持 |
| 企业级多租户 | kubernetes-sigs/agent-sandbox |
| 快速原型验证 | Alibaba/OpenSandbox |

---

## 总结

`★ Insight ─────────────────────────────────────`

### 架构哲学

1. **声明式 API**: 用户描述"想要什么"，控制器负责"实现什么"
2. **控制器模式**: 经典的 K8s Operator 模式，易于理解和扩展
3. **资源池化**: WarmPool 预热机制，解决冷启动延迟痛点
4. **生命周期**: 内置过期/清理机制，适合"临时 Agent"场景

### 适用场景

✅ **推荐使用**:
- 已使用 K8s 的企业
- 需要原生 K8s 集成 (RBAC, NetworkPolicy, etc.)
- 多租户 Agent 平台
- 需要与现有 K8s 生态集成

❌ **不推荐**:
- 轻量级 / 本地开发 (推荐 OpenSandbox)
- 简单的一次性任务

### 核心价值

- **稳定身份**: 持久的主机名和 DNS 访问
- **状态持久**: 支持 PVC 实现数据持久化
- **生命周期**: 自动过期清理，资源回收
- **快速启动**: WarmPool 实现毫秒级分配
- **K8s 原生**: 完全遵循 K8s 设计模式

---

## 参考资料

- [Kubernetes Agent Sandbox 官方仓库](https://github.com/kubernetes-sigs/agent-sandbox)
- [官方文档](https://agent-sandbox.sigs.k8s.io/docs/)
- [DeepWiki 分析](https://deepwiki.com/kubernetes-sigs/agent-sandbox)

---

*文档生成时间: 2025-03-24*
