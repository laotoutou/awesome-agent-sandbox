# OpenSandbox 深度技术分析

> 分析日期: 2026-03-24 | 来源: 阿里官方 GitHub 4.8k Stars

---

## 1. 概述

OpenSandbox 是阿里巴巴开源的**通用沙箱平台**，专为 AI 应用场景设计，提供多语言 SDK、标准化沙箱协议和灵活的运行时实现。

**核心定位:**
- 为 AI Agent 提供安全的代码执行环境
- 支持 Coding Agents、GUI Agents、Agent Evaluation、AI Code Execution、RL Training
- 已进入 CNCF Landscape

---

## 2. 整体架构

### 2.1 四层架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                      SDKs Layer                              │
│   Python | Java/Kotlin | JavaScript/TS | C#/.NET | Go       │
│   (客户端库，处理与 Sandbox 的通信)                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Specs Layer                             │
│   Sandbox Lifecycle API (OpenAPI)                           │
│   Sandbox Execution API (OpenAPI)                           │
│   (协议层，定义 SDK 与 Runtime 的契约)                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Runtime Layer                           │
│   Docker Runtime | Kubernetes Runtime | Custom Runtime       │
│   (沙箱生命周期管理)                                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Sandbox Instances                         │
│   Base Container + execd Daemon + User Entrypoint           │
│   (运行的沙箱实例)                                           │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 目录结构

| 目录 | 说明 |
|------|------|
| `sdks/` | 多语言 SDK (Python, Java, JS, C#) |
| `specs/` | OpenAPI 规范定义 |
| `server/` | FastAPI 生命周期服务器 |
| `kubernetes/` | K8s 部署配置 |
| `components/execd/` | 执行守护进程 (Go) |
| `components/ingress/` | 流量入口网关 |
| `components/egress/` | 出流量控制 |
| `sandboxes/` | 沙箱运行时实现 |
| `examples/` | 集成示例 |

---

## 3. 核心组件

### 3.1 SDK Layer

**多语言支持:**
- Python SDK
- Java/Kotlin SDK
- JavaScript/TypeScript SDK
- C#/.NET SDK
- Go SDK (Roadmap)

**核心 SDK 类:**

1. **Sandbox** - 沙箱生命周期管理
   - `create()` - 创建沙箱实例
   - `pause()` / `resume()` - 暂停/恢复
   - `renew_expiration()` - 延长 TTL
   - `kill()` - 销毁沙箱

2. **Filesystem** - 文件操作
   - `read_file()` / `write_files()`
   - `upload()` / `download()`
   - `search()` - Glob 模式搜索

3. **Commands** - 命令执行
   - `run()` - 同步执行
   - 后台进程支持
   - SSE 流式输出

4. **CodeInterpreter** - 代码解释器
   - 多语言支持 (Python, Java, JS, Go, Bash)
   - Jupyter 内核协议
   - 会话状态保持

### 3.2 Specs Layer

#### Sandbox Lifecycle API

定义沙箱实例的生命周期管理:

| 操作 | 端点 | 说明 |
|------|------|------|
| Create | POST /sandboxes | 从镜像创建沙箱 |
| List | GET /sandboxes | 列表查询 |
| Get | GET /sandboxes/{id} | 获取详情 |
| Delete | DELETE /sandboxes/{id} | 终止沙箱 |
| Pause | POST /sandboxes/{id}/pause | 暂停 |
| Resume | POST /sandboxes/{id}/resume | 恢复 |
| Renew | POST /sandboxes/{id}/renew-expiration | 续期 |
| Endpoint | GET /sandboxes/{id}/endpoints/{port} | 获取公网 URL |

#### Sandbox Execution API

定义与运行中沙箱实例的交互 (由 execd 实现):

- **Code**: 代码执行上下文创建/执行/中断
- **Command**: Shell 命令执行
- **Files**: 文件 CRUD 操作
- **Metrics**: 系统指标监控

### 3.3 Runtime Layer

#### Docker Runtime

```python
# 核心职责
1. Pull container image (支持私有仓库认证)
2. Create container with resource limits
3. Inject execd binary and start script
4. Monitor container state
5. Handle pause/resume
6. Cleanup on expiration
```

**两种网络模式:**
- **Host Mode**: 共享宿主机网络
- **Bridge Mode**: 隔离网络 + HTTP 路由

#### Kubernetes Runtime

支持两种部署模式:

1. **BatchSandbox** (阿里自研)
   - 沙箱池化
   - 高吞吐批量创建
   - 异构任务编排

2. **SIG agent-sandbox 兼容**
   - 与 kubernetes-sigs/agent-sandbox 集成
   - 使用其 CRD

### 3.4 execd - 核心创新

**位置**: `components/execd/`

execd 是注入到每个沙箱容器的 Go 编写的 HTTP 守护进程。

#### 注入机制

```
容器创建流程:
1. Pull execd 镜像
2. Extract execd binary → 临时目录
3. Volume mount 到目标容器
4. Override entrypoint
5. 启动顺序:
   start.sh → Jupyter Server → execd → user entrypoint
```

#### 启动脚本示例

```bash
# /opt/opensandbox/start.sh

# 1. 启动 Jupyter Server
jupyter notebook --port=54321 --no-browser --ip=0.0.0.0

# 2. 启动 execd daemon
/opt/opensandbox/execd --jupyter-host=http://127.0.0.1:54321 --port=44772

# 3. 执行用户入口点
exec "${USER_ENTRYPOINT[@]}"
```

#### 功能模块

| 模块 | 说明 |
|------|------|
| `pkg/flag/` | 配置和 CLI 参数 |
| `pkg/web/` | HTTP 层 (controllers, models, router) |
| `pkg/runtime/` | 执行调度器 |
| `pkg/jupyter/` | Jupyter 内核客户端 |
| `pkg/util/` | 工具函数 |

#### 支持的内核

- Python (IPython)
- Java (IJava)
- JavaScript (IJavaScript)
- TypeScript (ITypeScript)
- Go (gophernotes)
- Bash

---

## 4. 数据流动

### 4.1 沙箱创建流程

```
User/SDK
   │
   │ 1. POST /sandboxes (image, entrypoint, resources)
   ▼
Server (Lifecycle API)
   │
   │ 2. Pull container image
   │ 3. Extract & inject execd
   │ 4. Create container
   │ 5. Start container
   ▼
Sandbox Instance
   │
   │ 6. Start execd daemon
   │ 7. Start Jupyter Server
   │ 8. Execute user entrypoint
   ▼
Running
```

### 4.2 代码执行流程

```
User/SDK
   │
   │ 1. Create sandbox
   │ 2. Get execd endpoint
   ▼
CodeInterpreter SDK
   │
   │ 3. POST /code/context (创建会话)
   │ 4. POST /code (执行代码)
   ▼
execd (Execution API)
   │
   │ 5. Route to Jupyter runtime
   ▼
Jupyter Runtime
   │
   │ 6. WebSocket to Jupyter Server
   │ 7. Send execute_request
   ▼
Jupyter Kernel (Python/Java/etc.)
   │
   │ 8. Execute code
   │ 9. Stream output events
   ▼
execd
   │
   │ 10. Convert to SSE events
   │ 11. Stream to client
   ▼
CodeInterpreter SDK → User
```

---

## 5. 网络架构

### 5.1 Ingress Gateway

提供 HTTP/HTTPS 负载均衡到沙箱实例端口:

**功能:**
- 动态端点生成: `{domain}/sandboxes/{sandboxId}/port/{port}`
- 支持域名和通配符路由
- 反向代理到容器端口
- 自动清理

**使用场景:**
- 访问沙箱内的 Web 应用
- 连接开发服务器 (VS Code Server)
- VNC 和远程桌面访问

### 5.2 Egress Control

出流量控制:
- 按域名/IP 过滤
- 白名单/黑名单策略
- 审计日志

---

## 6. 安全特性

### 6.1 隔离技术

支持多种安全容器运行时:

| 运行时 | 类型 | 隔离级别 |
|--------|------|----------|
| gVisor | 用户态内核 | 高 |
| Kata Containers | 轻量 VM | 很高 |
| Firecracker | 微 VM | 极高 |

### 6.2 认证机制

- **Lifecycle API**: API Key 认证
- **Execution API**: Token 认证
- **Network**: Ingress/Egress 控制

### 6.3 资源限制

- CPU 配额
- 内存配额
- GPU 配额 (Roadmap)

---

## 7. 使用示例

### 7.1 Python SDK

```python
import asyncio
from datetime import timedelta
from code_interpreter import CodeInterpreter, SupportedLanguage
from opensandbox import Sandbox
from opensandbox.models import WriteEntry

async def main():
    # 1. 创建沙箱
    sandbox = await Sandbox.create(
        "opensandbox/code-interpreter:v1.0.2",
        entrypoint=["/opt/opensandbox/code-interpreter.sh"],
        timeout=timedelta(minutes=10),
    )

    async with sandbox:
        # 2. 执行 shell 命令
        result = await sandbox.commands.run("echo 'Hello'")
        print(result.logs.stdout)

        # 3. 写文件
        await sandbox.files.write_files([
            WriteEntry(path="/tmp/test.py", data="print('hi')")
        ])

        # 4. 代码解释器
        interpreter = await CodeInterpreter.create(sandbox)
        result = await interpreter.codes.run("2 + 2", language=SupportedLanguage.PYTHON)
        print(result.result[0].text)  # 4

    await sandbox.kill()

asyncio.run(main())
```

### 7.2 K8s 部署

```bash
# Helm 安装
helm install opensandbox ./charts/opensandbox
```

---

## 8. Roadmap

### SDK
- Sandbox client connection pool - 客户端连接池，预创建沙箱实现 X ms 获取
- Go SDK - Go 语言客户端

### Runtime
- Persistent volumes - 可挂载持久卷
- Local lightweight sandbox - PC 本地轻量沙箱
- Secure Container - 安全容器沙箱

### Deployment
- 自托管 K8s 集群部署指南

---

## 9. 总结

`★ Insight ─────────────────────────────────────`
OpenSandbox 的核心设计哲学:

1. **协议优先**: 通过 OpenAPI 定义清晰的契约，支持多语言实现
2. **运行时注入**: execd 守护进程动态注入，无需修改用户镜像
3. **多运行时**: Docker 本地运行 + K8s 大规模部署
4. **功能完备**: 内置代码解释器、文件操作、命令执行、指标监控

**适用场景:**
- AI Code Generation & Execution
- 交互式编码环境 (Jupyter)
- 浏览器自动化 (Chrome/Playwright)
- 远程开发环境 (VS Code Server)
- CI/CD 测试
`─────────────────────────────────────────────────`
