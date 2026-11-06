# Awesome Agent Sandbox

> A curated list of code-execution sandboxing solutions for AI/LLM agents, along with related tools, frameworks, and resources.

AI coding agents are powerful because they can read files, run commands, install packages, and modify code—but this also makes them risky. This list focuses on projects that reduce that risk with OS sandboxes, virtual machines, containers, policy engines, and related guardrails.

---

## Contents

- [Cloud Sandboxes](#cloud-sandboxes)
- [Self-hosted / Open Source](#self-hosted--open-source)
- [Local Desktop Sandboxes](#local-desktop-sandboxes)
- [Container-based Solutions](#container-based-solutions)
- [MicroVM Platforms](#microvm-platforms)
- [Policy & Security Layers](#policy--security-layers)
- [Foundational Primitives](#foundational-primitives)
- [MCP Integration](#mcp-integration)
- [Related Awesome Lists](#related-awesome-lists)

---

## Cloud Sandboxes

> Managed sandbox services with SDK support and scalable infrastructure.

| Project | Description | Stars |
|--------|-------------|-------|
| [E2B](https://github.com/e2b-dev/E2B) | Open-source cloud runtime with Linux OS and SDK support | ⭐ 13.4k |
| [AgentSphere](https://www.agentsphere.run/) | MicroVM sandboxes with MCP integration | - |
| [Runloop](https://www.runloop.ai/) | Fast devboxes with snapshots and repo connections | - |
| [Sandbox0](https://sandbox0.ai/) | Managed cloud computers with gVisor isolation and persistent writable RootFS checkpoints | - |
| [Modal Sandboxes](https://modal.com/docs/guide/sandboxes) | Programmatic sandboxes at massive scale, sub-second startup | - |
| [Deno Sandboxes](https://docs.deno.com/sandboxes/) | Millisecond boot times | - |
| [Sprites](https://sprites.dev/) | Persistent Firecracker VMs with exec sessions | - |
| [Cognitora](https://www.cognitora.dev/) | Firecracker microVMs with hardware-level isolation | - |
| [exe.dev](https://exe.dev/) | 2-second Ubuntu VMs for coding agents | - |
| [YepCode Run](https://yepcode.io/run) | Serverless JS/Python with enterprise-grade sandboxing | - |
| [shellbox.dev](https://shellbox.dev/) | SSH-accessible Linux boxes | - |
| [Novita Sandbox](https://novita.ai/docs/guides/sandbox-agent-runtime-introduction) | Agent deployment with SDK decorators | - |
| [BlueRock Agent Sandbox](https://www.bluerock.io/products/agent-sandbox) | Commercial sandbox with full MCP visibility | - |
| [Sandstorm](https://github.com/tomascupr/sandstorm) | Run Claude agents in secure cloud sandboxes via API/CLI | ⭐ |
| **[OpenSandbox](https://github.com/opensandbox-group/OpenSandbox)** | **通用沙箱平台，多语言SDK，Docker/K8s运行时，支持gVisor/Kata/Firecracker** | **⭐ 12.7k** |
| [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) | 腾讯云：即时、并发、安全轻量 agent 沙箱 | ⭐ 11.1k |
| [Cloudflare Sandbox SDK](https://github.com/cloudflare/sandbox-sdk) | 在 Cloudflare 边缘网络运行沙箱化代码 | ⭐ 1.1k |
| [AgentBay SDK (无影)](https://github.com/agentbay-ai/wuying-agentbay-sdk) | 阿里云无影：为 AI Agent 打造的云沙箱 | ⭐ 1.1k |
| [SuperServe](https://github.com/superserve-ai/superserve) | AI Agent 沙箱基础设施 | ⭐ 453 |
| [ClickHouse code-interpreter](https://github.com/ClickHouse/code-interpreter) | 沙箱化代码执行 API，驱动 LibreChat | ⭐ 95 |
| [EVE (Vercel)](https://github.com/vercel/eve) | Vercel 开源 agent 框架，执行层构建于 Vercel Sandbox 之上 | ⭐ 4.6k |
| [Coding Agent Template (Vercel)](https://github.com/vercel-labs/coding-agent-template) | 多智能体 AI 编程平台，由 Vercel Sandbox + AI Gateway 驱动 | ⭐ 1.8k |

---

## Self-hosted / Open Source

> Self-deployable sandbox solutions for local or private cloud deployment.

### All-in-One Solutions

| Project | Description | Stars |
|--------|-------------|-------|
| [AIO Sandbox](https://github.com/agent-infra/sandbox) | All-in-one container with browser, terminal, VS Code, Jupyter, MCP | ⭐ 5.7k |
| [Kubernetes Agent Sandbox](https://github.com/kubernetes-sigs/agent-sandbox) | Kubernetes CRD with gVisor/Kata isolation | ⭐ 2.4k |
| **[OpenSandbox](https://github.com/opensandbox-group/OpenSandbox)** | **通用沙箱平台，多语言SDK(Python/JS/Java/C#)，Docker/K8s运行时，支持gVisor/Kata/Firecracker** | **⭐ 12.7k** |
| [agent-sandbox](https://github.com/agent-sandbox/agent-sandbox) | E2B-compatible, enterprise-grade runtime | ⭐ |
| [EdgeBox](https://github.com/BIGPPWONG/EdgeBox) | GUI-powered local sandbox with MCP protocol support | ⭐ |
| [Kilntainers](https://github.com/Kiln-AI/Kilntainers) | MCP server for ephemeral Linux sandboxes | ⭐ |
| [Daytona](https://www.daytona.io/docs/en/sandboxes/) | Open-source infrastructure with SDK/CLI | - |
| [Tensorlake](https://github.com/tensorlakeai/tensorlake) | Serverless runtime for sandboxes and agents | ⭐ |
| [AgentScope Runtime](https://github.com/agentscope-ai/agentscope-runtime) | Production-ready with secure tool sandboxing | ⭐ |
| [SWE-ReX](https://github.com/SWE-agent/SWE-ReX) | 大规模并行沙箱化代码执行，驱动 SWE-agent | ⭐ 570 |
| [agentos](https://github.com/rivet-dev/agentos) | 以库形式给 agent 一个操作系统（WASM/V8 isolate，无容器无 VM） | ⭐ 4.4k |
| [secure-exec](https://github.com/rivet-dev/secure-exec) | 安全的 Node.js 执行，无需沙箱/容器/VM | ⭐ 970 |
| [axern](https://github.com/cofy-x/axern) | 开源 AI agent 沙箱，可跑不可信代码与持久化服务 | ⭐ 271 |
| [kweaver Sandbox](https://github.com/kweaver-ai/sandbox) | 为 AI Agent 设计的高性能代码执行服务，多层隔离 | ⭐ 8 |
| [agentbox (Michaelliv)](https://github.com/Michaelliv/agentbox) | A computer for your agent - agent 的沙箱化代码执行 | ⭐ 47 |
| [omnigent](https://github.com/omnigent-ai/omnigent) | agent 元框架，可编排 Claude Code/Codex/Cursor，内置策略与沙箱化 | ⭐ 8.8k |
| [moltis](https://github.com/moltis-org/moltis) | Rust 写的个人 agent 服务器：单二进制 + 沙箱化执行 | ⭐ 2.8k |

### Lightweight / Experimental

| Project | Description | Stars |
|--------|-------------|-------|
| [Sandboxer](https://github.com/ammmir/sandboxer) | Forkable server for LLMs and agents | ⭐ |
| [Arrakis](https://github.com/abshkbh/arrakis) | MicroVM isolation with snapshots | ⭐ |
| [Bouvet](https://github.com/vrn21/bouvet) | Rust-based sandbox | ⭐ |
| [Microsandbox](https://github.com/superradcompany/microsandbox) | VM-level isolation, <200ms boot | ⭐ 7.5k |
| [SandboxAI](https://github.com/substratusai/sandboxai) | Multi-cloud infrastructure for AI code | ⭐ |
| [Fence](https://github.com/fencesandbox/fence) | Lightweight CLI with OS-native tools | ⭐ 898 |
| [nono](https://huggingface.co/blog/lukehinds/nono-agent-sandbox) | Capability-based shell | - |
| [Flintlock](https://github.com/liquidmetal-dev/flintlock) | MicroVM lifecycle management | ⭐ |
| [Volant](https://github.com/0xchasercat/volant) | MicroVM orchestration, ~200ms cold start | ⭐ |
| [Capsule](https://github.com/bots-garden/capsule) | WASM runners | ⭐ |
| [Enclave](https://github.com/agentfront/enclave) | JavaScript sandbox preventing injection | ⭐ |
| [Sandbox Agent](https://github.com/rivet-dev/sandbox-agent) | Universal API for Claude Code, Codex, OpenCode | ⭐ |
| [pctx](https://github.com/portofcontext/pctx) | Execution layer with type-checked Deno | ⭐ |

---

## Local Desktop Sandboxes

### macOS

| Project | Description | Stars |
|--------|-------------|-------|
| [Agent Safehouse](https://github.com/eugene1g/agent-safehouse) | Deny-first macOS Seatbelt profile | ⭐ 2.8k |
| [SandVault](https://github.com/webcoyote/sandvault) | Separate macOS user account + sandbox-exec | ⭐ |
| [vibebox](https://github.com/robcholz/vibebox) | Fast local macOS sandbox | ⭐ |
| [yolobox](https://github.com/finbarr/yolobox) | Protect home directory | ⭐ |
| [sandbox-shell](https://github.com/agentic-dev3o/sandbox-shell) | macOS Seatbelt shell wrapper | ⭐ |
| [Chamber](https://github.com/cirruslabs/chamber) | Ephemeral Tart-based macOS VM | ⭐ |
| [ClodPod](https://github.com/webcoyote/clodpod) | macOS VM workflow | ⭐ |
| [lima-devbox](https://github.com/recodelabs/lima-devbox) | Lima-based VM dev sandbox | ⭐ |

### Linux

| Project | Description | Stars |
|--------|-------------|-------|
| [Fence](https://github.com/fencesandbox/fence) | Native command sandbox | ⭐ 898 |
| [Matchlock](https://github.com/jingkaihe/matchlock) | Securing AI agent workloads | ⭐ |
| [Microbox](https://github.com/hqarroum/microbox) | Lightweight ephemeral sandboxes | ⭐ |
| [Nono](https://github.com/nolabs-ai/nono) | Capability-oriented kernel sandbox | ⭐ 3.6k |
| [sandbox-run](https://codeberg.org/Grauwolf/sandbox-run) | bubblewrap-based wrapper | ⭐ |
| [treebeard](https://github.com/divmain/treebeard) | Ephemeral Git worktree sandbox | ⭐ |
| [shai](https://github.com/colony-2/shai) | Sandboxing shell for agents | ⭐ |
| [sucoder](https://github.com/ligon/sucoder) | Unix-permissions-based approach | ⭐ |
| [Greywall](https://github.com/greyhavenhq/greywall) | Live network-control sandbox | ⭐ |

### Cross-platform

| Project | Description | Stars |
|--------|-------------|-------|
| [agent-sandbox.nix](https://github.com/archie-judd/agent-sandbox.nix) | Nix wrappers for constrained execution | ⭐ |
| [cco](https://github.com/nikvdp/cco) | Thin launcher with sandbox backends | ⭐ |
| [nixcage](https://github.com/hamidr/nixcage) | Nix environment sandboxing | ⭐ |
| [Pent](https://github.com/valentinradu/pent) | Native OS process sandbox | ⭐ |
| [Sandbox Runtime](https://github.com/anthropic-experimental/sandbox-runtime) | OS-level filesystem/network restrictions | ⭐ |
| [sandnix](https://github.com/srid/sandnix) | Nix module with Landlock | ⭐ |
| [workmux](https://github.com/raine/workmux) | Git worktree + tmux isolation | ⭐ |
| [yoloAI](https://github.com/kstenerud/yoloAI) | Multi-backend (Seatbelt, Tart, Docker) | ⭐ |
| [open-cowork](https://github.com/OpenCoworkAI/open-cowork) | Windows/macOS agent 桌面应用，一键装 Claude Code + 沙箱隔离 | ⭐ 2k |

---

## Container-based Solutions

| Project | Description | Stars |
|--------|-------------|-------|
| [littlebox](https://github.com/gbrindisi/littlebox) | Containerized with network firewalling（原 agentbox，已改名） | ⭐ |
| [Amazing Sandbox](https://github.com/ashishb/amazing-sandbox) | General local sandbox | ⭐ |
| [ClaudeBox](https://github.com/RchGrav/claudebox) | Docker-based Claude Code env | ⭐ |
| [codex-lockbox](https://github.com/paulux84/codex-lockbox) | Docker sandbox for Codex | ⭐ |
| [conch](https://github.com/sd2k/conch) | Wasm/bash-style sandbox | ⭐ |
| [packnplay](https://github.com/obra/packnplay) | Docker-backed command sandbox | ⭐ |
| [sandclaude](https://github.com/binwiederhier/sandclaude) | Docker wrapper for Claude Code | ⭐ |
| [Sculptor](https://github.com/imbue-ai/sculptor) | Desktop tooling for isolated containers | ⭐ |
| [vibebin](https://github.com/jgbrwn/vibebin) | Incus/LXC platform | ⭐ |
| [runbox](https://github.com/sahilb315/runbox) | Minimal container-like sandbox in C | ⭐ |

---

## MicroVM Platforms

| Project | Description | Stars |
|--------|-------------|-------|
| [BoxLite](https://github.com/boxlite-ai/boxlite) | Embeddable VM with snapshots | ⭐ 2.3k |
| [boxed](https://github.com/akshayaggarwal99/boxed) | Docker, Firecracker, Wasm backends | ⭐ |
| [coderunner](https://github.com/instavm/coderunner) | Hosted-style sandbox runner | ⭐ |
| [K7](https://github.com/katakate/k7) | Self-hosted lightweight VM infrastructure | ⭐ |
| [smolVM](https://github.com/smol-machines/smolvm) | Local microVM manager | ⭐ 4.9k |
| [Cleanroom](https://github.com/buildkite/cleanroom) | Self-hosted microVM with egress control | ⭐ |
| [bunkervm](https://github.com/ashishgituser/bunkervm) | Tiny Linux VM for agents | ⭐ |
| [zeroboot](https://github.com/zerobootdev/zeroboot) | Sub-millisecond VM via copy-on-write | ⭐ 2.4k |
| [astrid](https://github.com/astrid-runtime/astrid) | 可移植、能力安全的 agent 操作系统 | ⭐ 10.3k |
| [forkd](https://github.com/deeplethe/forkd) | 为 agent 微VM 提供 fork()：100ms 起 100 个子实例 | ⭐ 2.8k |

---

## Policy & Security Layers

| Project | Description | Stars |
|--------|-------------|-------|
| [Cupcake](https://github.com/eqtylab/cupcake) | OPA/Rego-based hook enforcement | ⭐ |
| [deepclause-sdk](https://github.com/deepclause/deepclause-sdk) | Policy/runtime SDK | ⭐ |
| [nah](https://github.com/manuelschipper/nah) | Allow/ask/block guard for Claude Code | ⭐ |
| [predicate-secure](https://github.com/PredicateSystems/predicate-secure) | Policy-based authorization | ⭐ |
| [punkgo-jack](https://github.com/PunkGo/punkgo-jack) | Audit via Merkle-logged hooks | ⭐ |
| [shannot](https://github.com/corv89/shannot) | Human-in-the-loop approval flow | ⭐ |
| [yolo-cage](https://github.com/borenstein/yolo-cage) | Anti-exfiltration sandbox | ⭐ |
| [AgentFence](https://github.com/agentfence/agentfence) | Security testing for prompt injection | ⭐ |
| [Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit) | 微软：策略执行、零信任身份、执行沙箱化，覆盖 OWASP Agentic Top 10 | ⭐ 5.9k |
| [SecGPT](https://github.com/llm-platform-security/SecGPT) | LLM 多智能体系统的执行隔离架构（学术论文项目） | ⭐ 121 |
| [hazmat](https://github.com/dredozubov/hazmat) | AI agent 遏制：用户隔离、沙箱化执行、网络控制、备份/回滚，TLA+ 验证 | ⭐ 124 |
| [KubeArmor](https://github.com/kubearmor/KubeArmor) | 容器运行时安全执行系统，LSM/AppArmor 最小权限沙箱化 | ⭐ 2.6k |

---

## Foundational Primitives

### Linux

| Project | Description | Stars |
|--------|-------------|-------|
| [bubblewrap](https://github.com/containers/bubblewrap) | Core Linux namespace sandbox | ⭐ |
| [bVisor](https://github.com/butter-dot-dev/bvisor) | Embedded bash sandbox | ⭐ |
| [Firejail](https://github.com/netblue30/firejail) | Mature desktop sandbox | ⭐ |
| [gVisor](https://github.com/google/gvisor) | User-space kernel | ⭐ |
| [island](https://github.com/landlock-lsm/island) | Landlock-powered CLI | ⭐ |
| [libkrun](https://github.com/containers/libkrun) | Lightweight virtualization | ⭐ |
| [Minijail](https://github.com/google/minijail) | ChromeOS/Android containment | ⭐ |
| [syd](https://git.sr.ht/~alip/syd) | Userspace syscall-intercepting sandbox | ⭐ |
| [sacre_bleu](https://github.com/hsaliak/sacre_bleu) | Seccomp and Landlock suite | ⭐ |
| [Firecracker](https://github.com/firecracker-microvm/firecracker) | 微VM 运行时，agent 沙箱的基础设施（E2B/Sprites 等底层） | ⭐ 36k |
| [Firecracker Go SDK](https://github.com/firecracker-microvm/firecracker-go-sdk) | Go SDK for Firecracker | ⭐ |
| [nanos](https://github.com/nanovms/nanos) | 单应用微内核（unikernel），虚拟化环境运行单一程序 | ⭐ 3.2k |

### WASM

| Project | Description | Stars |
|--------|-------------|-------|
| [Capsule](https://github.com/bots-garden/capsule) | WASM runners | ⭐ |
| [wasmtime](https://github.com/bytecodealliance/wasmtime) | 轻量 WASM 运行时，快速安全，agent 代码沙箱常用底层 | ⭐ 18.5k |
| [wasm3](https://github.com/wasm3/wasm3) | 最快的 WASM 解释器，通用运行时 | ⭐ 8k |
| [wasmi](https://github.com/wasmi-labs/wasmi) | 嵌入式系统的 WASM 解释器（Rust） | ⭐ 2.2k |

### JavaScript

| Project | Description | Stars |
|--------|-------------|-------|
| [vm2](https://github.com/patriksimek/vm2) | Node.js 高级 VM/沙箱 | ⭐ 4.1k |
| [JS-Interpreter](https://github.com/NeilFraser/JS-Interpreter) | 用 JS 实现的沙箱化 JS 解释器 | ⭐ 2.2k |

---

## MCP Integration

> Tools with Model Context Protocol support.

| Project | Description |
|---------|-------------|
| [AgentSphere](https://www.agentsphere.run/) | MicroVM + MCP |
| [AIO Sandbox](https://github.com/agent-infra/sandbox) | Full-featured container with MCP |
| [EdgeBox](https://github.com/BIGPPWONG/EdgeBox) | GUI sandbox with MCP support |
| [Kilntainers](https://github.com/Kiln-AI/Kilntainers) | MCP server for ephemeral sandboxes |
| [VibeKit](https://docs.vibekit.sh/) | SDK supporting E2B, Daytona, Modal |
| [forgemax](https://github.com/postrv/forgemax) | MCP gateway with sandboxed execution |
| [Composio](https://github.com/ComposioHQ/composio) | 1000+ toolkits with MCP and sandboxed workbench |
| [Apify Actor AI Code Sandbox](https://github.com/apify/actor-ai-code-sandbox) | AI 生成代码安全执行沙箱，预配置 Claude Code/Codex/OpenCode |

---

## Related Awesome Lists

- [awesome-agent-sandboxes](https://github.com/arjan/awesome-agent-sandboxes) - Original curated list
- [awesome-AI-sandbox](https://github.com/webcoyote/awesome-AI-sandbox) - Comprehensive security-focused list
- [awesome-agentic-hardening](https://github.com/AgenticHardening/awesome-agentic-hardening) - Prompt injection defense & security
- [awesome-sandbox](https://github.com/restyler/awesome-sandbox) - Broad sandboxing technologies

---

## Contributing

Contributions welcome! Please read the [contribution guidelines](CONTRIBUTING.md) first.

---

## License

[![CC0](https://licensebuttons.net/p/zero/1.0/80x15.png)](https://creativecommons.org/publicdomain/zero/1.0/)
