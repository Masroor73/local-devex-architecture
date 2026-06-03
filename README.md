# Production-Grade Local Development Ecosystem

A polyglot, event-driven engineering environment designed to eliminate the "works on my machine" anti-pattern and minimize developer cognitive load. This spec treats hardware as Infrastructure as Code, enforces defence-in-depth security with no plaintext credential exposure, and aggressively replaces JVM-heavy tools with compiled-language alternatives to reduce idle RAM consumption.

**Target Hardware:**
- Windows 11 — MSI Raider (16GB RAM)
- macOS — MacBook Air M4 (16GB RAM)

---

## Table of Contents

1. [Core Engineering Principles](#1-core-engineering-principles)
2. [Unified Toolchain & Scope Matrix](#2-unified-toolchain--scope-matrix)
3. [Cross-OS Boundary Management](#3-cross-os-boundary-management)
4. [Declarative Schemas & AI Tool Allocation](#4-declarative-schemas--ai-tool-allocation)
5. [Automated Workflows, Maintenance & Observability](#5-automated-workflows-maintenance--observability)
6. [Team Onboarding Sequence](#6-team-onboarding-sequence)
7. [Failure Modes & Recovery Runbooks](#7-failure-modes--recovery-runbooks)
8. [RAM Budget Reference](#8-ram-budget-reference)
9. [Code Quality & Cross-Service Testing](#9-code-quality--cross-service-testing)

---

## 1. Core Engineering Principles

**Pragmatism Over Purism** — Tools are selected based on objective utility, performance, and cognitive ease. If a GUI reduces error rate and review time compared to a CLI for a given task, the GUI is the standard.

**Best Tool for the Purpose** — Unified cross-platform tools are only used if they provide a measurable performance boost. Otherwise, the apex native tool for each OS is used (e.g., Ghostty on Metal for macOS; Windows Terminal on DirectX for Windows).

**Architectural Fluidity** — The environment supports both high-throughput Microservices (Go/C#) and Modular Monoliths (Laravel/Django). The toolchain adapts to the project's optimal pattern without forcing a one-size-fits-all approach.

---

## 2. Unified Toolchain & Scope Matrix

**Scope Legend:**
- `UNIVERSAL` — Deployed identically on both Windows and macOS nodes
- `CONTEXTUAL` — Task-specific or platform-specific; deployed only when the relevant workload is active

| Architectural Layer | Core Tooling | Scope | Windows | macOS M4 |
|---|---|---|---|---|
| Package Management (OS) | Automated software installation | UNIVERSAL | WinGet | Homebrew |
| Package Management (JS) | Fast, hard-linked I/O operations | UNIVERSAL | pnpm (via Corepack) | pnpm (via Corepack) |
| Execution Engine | Underlying OS for code compilation | UNIVERSAL | WSL2 (Ubuntu) | Native Unix |
| Terminal | GPU-accelerated command interface | UNIVERSAL | Windows Terminal | Ghostty |
| Runtime Orchestrator | Global language version management | UNIVERSAL | mise | mise |
| Delegated Runtimes | Best-in-class language compilers | UNIVERSAL | uv (Python), rustup (Rust) | uv (Python), rustup (Rust) |
| Event Streaming | Kafka-compatible C++ broker | CONTEXTUAL | Redpanda (via Docker Desktop) | Redpanda (via OrbStack) |
| Polyglot IDE | AI-assisted development context | UNIVERSAL | Cursor (with WSL extension) | Cursor |
| Agentic Host IDE | Base window for agent tools | CONTEXTUAL | VS Code (with WSL extension) | VS Code |
| AI Coding Agent | Agentic, context-deep task execution | CONTEXTUAL | Claude Code CLI | Claude Code CLI |
| Database IDE | Heavy-duty backend & DB UI | CONTEXTUAL | JetBrains Rider | JetBrains Rider |
| API Contract Code-Gen | Strict typing across polyglot boundaries | UNIVERSAL | OpenAPI, Orval, Zod | OpenAPI, Orval, Zod |
| API Client | Version-controlled REST/gRPC testing | UNIVERSAL | Bruno | Bruno |
| Version Control GUI | Visual Git diffs and staging | UNIVERSAL | GitHub Desktop | GitHub Desktop |
| Web Browser (Dev) | Chromium-based local testing | UNIVERSAL | Brave Browser | Brave Browser |
| Containerisation | Virtualised project isolation | UNIVERSAL | Docker Desktop | OrbStack |
| Local Kubernetes | Lightweight cluster simulation | CONTEXTUAL | k3d | k3d |
| Database Migrations | Declarative schema management | UNIVERSAL | Atlas (ariga.io) | Atlas (ariga.io) |
| Local Observability | OpenTelemetry trace visualisation | CONTEXTUAL | .NET Aspire Dashboard | .NET Aspire Dashboard |
| Secrets & Identity | SSH Agent & dynamic secret injection | UNIVERSAL | Bitwarden & BWS CLI | Bitwarden & BWS CLI |
| Local Security Scanner | Container & dependency CVE checks | UNIVERSAL | Trivy (Aqua Security) | Trivy (Aqua Security) |
| Local & Public Ingress | Secure webhooks & HTTPS routing | CONTEXTUAL | Caddy & Cloudflare Tunnels | Caddy & Cloudflare Tunnels |
| Mesh Networking | Cross-device secure VPN | UNIVERSAL | Tailscale | Tailscale |
| CI/CD & Automation | Pipeline execution & dependency updates | CONTEXTUAL | GitHub Actions, Renovate Bot | GitHub Actions, Renovate Bot |
| Infrastructure as Code | Deterministic cloud provisioning | CONTEXTUAL | Pulumi | Pulumi |
| Production Observability | Error grouping & long-term metrics | CONTEXTUAL | Sentry, Prometheus + Grafana | Sentry, Prometheus + Grafana |

---

## 3. Cross-OS Boundary Management

To prevent I/O throttling, polyglot formatting errors, and system crashes on Windows nodes, the architecture mandates strict boundary protocols.

### The NTFS Bypass

Repositories are cloned into the **native WSL ext4 filesystem**, not the Windows filesystem. Cloning to the Windows filesystem causes severe I/O slowdowns on Linux compilation benchmarks due to virtualization cross-filesystem overhead.

```
# Target path — substitute <your-username> with your WSL username
\\wsl.localhost\Ubuntu\home\<your-username>\Projects
```

### WSL2 Memory Cap

To prevent WSL2 from starving the host machine during intense concurrent builds, enforce a `.wslconfig` in your Windows user profile directory (`C:\Users\<your-username>\.wslconfig`):

```ini
[wsl2]
memory=10GB
processors=4
autoMemoryReclaim=dropcache
```

This allocates 10GB to Linux, preserving ~6GB headroom for native host processes.

### Line Ending Enforcer

Configure Git globally to force LF line endings, preventing Windows from injecting carriage return breaks into scripts destined for Linux containers:

```bash
git config --global core.autocrlf false
```

---

## 4. Declarative Schemas & AI Tool Allocation

### Schema & API Contract Management

The centralized relational database schema is managed declaratively via **Atlas** rather than language-specific ORM migrations colliding across polyglot service boundaries.

For inter-service boundaries, backends output **OpenAPI** or **Protobuf** specifications. **Orval** reads these local specs automatically to compile type-safe client repositories, React Query hooks, and Zod schemas — converting backend changes into instant frontend compile-time errors.

### The Centaur Heuristic: AI Engine Allocation

AI execution is explicitly allocated to two engines based on the contextual depth and boundary scope of the task:

**Cursor** *(Tab Autocomplete + Localised Scaffolding)*
Invoked for greenfield code construction, real-time typing completion loops, and single-package component generation. Operations are guided by rules hardcoded in the project's `.cursorrules` file.

**Claude Code + VS Code** *(Agentic Cross-Service Refactoring — Optional)*
Invoked via the terminal wrapper inside a dedicated VS Code window when an edit or dependency update impacts multiple decoupled microservices or database definitions. Because Cursor isolates its environment from the native Anthropic terminal daemon, VS Code serves as the secondary execution interface for deep multi-file refactoring runs. Claude Code's 1M-token context horizon holds the entire multi-language project state simultaneously, preventing silent contract breakages across service boundaries. Requires a Claude Pro or Max subscription — engineers without one should default to Cursor for all tasks, which covers standard daily workflows.

---

## 5. Automated Workflows, Maintenance & Observability

### Commit Message Optimisation

The developer writes a concise, 1-sentence draft containing the subjective "Why" in GitHub Desktop. A Git hook intercepts the raw diff and reformats the string to comply with the [Conventional Commits](https://www.conventionalcommits.org/) standard automatically.

### Supply-Chain Automation & Dependency Gating (Renovate)

Renovate Bot runs via GitHub Actions to scan lockfiles daily, run automated test pipelines against package variations, and auto-merge minor/patch iterations with a 3-day stability window on formatting utilities.

Critical runtime dependencies are isolated from auto-merge via `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "packageRules": [
    {
      "matchPackagePatterns": ["Microsoft.EntityFrameworkCore", "pg", "auth", "processor"],
      "automerge": false,
      "dependencyDashboardApproval": true
    }
  ]
}
```

Payment dependencies, database drivers, and authentication layers require manual approval before merging.

### Infrastructure as Code & Observability

Manual cloud modifications are strictly prohibited. **Pulumi** provisions all infrastructure resources using application-native TypeScript, executed exclusively via GitHub Actions CI/CD pipelines.

- **Local distributed tracing:** .NET Aspire Dashboard
- **Production error grouping:** Sentry
- **Long-term metrics baseline:** Prometheus + Grafana

---

## 6. Team Onboarding Sequence

Full workspace provisioning to operational parity in approximately 30 minutes.

### Prerequisites (Team Lead Setup)

- Developer's GitHub profile added to the core organization workspace
- Developer identity invited to the Bitwarden Secrets Manager team with minimal access boundaries
- A unique project access machine token generated via the BWS console and shared over a secure, ephemeral channel

### Step-by-Step Developer Execution

**Step 1 — Initialize Universal Environment Tools**

Fire the remote configuration sync engine from your native terminal profile to provision `mise`, `pnpm` via Corepack, `Lefthook`, `Trivy`, and the Bitwarden CLI. Replace `<org-dotfiles-repo>` with your organization's chezmoi dotfiles repository:

```bash
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply <org-dotfiles-repo>
```

**Step 2 — Authenticate Environment Identity Vaults**

```bash
bw login
bws login
```

Input the BWS organization machine token when prompted to activate runtime secret injection hooks.

**Step 3 — Initialize Local Filesystem Structures**

Before checking out code, manually initialize folder structures inside the native filesystem to bypass the Windows cross-filesystem virtualization barrier.

*Windows (WSL) — substitute `<your-username>` with your WSL username:*
```bash
mkdir -p /home/<your-username>/Projects && cd ~/Projects
```

*macOS:*
```bash
mkdir -p ~/Projects && cd ~/Projects
```

Open GitHub Desktop, click **Clone**, and target this initialized directory.

**Step 4 — Bridge IDE Runtime Communication**

Launch the cloned workspace in Cursor. On Windows, explicitly initialize the **"Connect to WSL"** bridge from the Command Palette before installing or running language server extensions.

**Step 5 — Secure Local Quality Gates**

```bash
lefthook install
trivy fs . --severity CRITICAL
```

**Step 6 — Bootstrap Agent Runtimes via VS Code Bridge (Optional)**

> **Prerequisite:** This step requires a Claude Pro or Max subscription. Skip this step if you do not hold one — Cursor covers all standard daily engineering workflows without it.

Open a vanilla VS Code instance mapped to your execution host. Install the Claude Code CLI using explicit global package manager parameters:

```bash
pnpm add -g @anthropic-ai/claude-code
```

Execute `claude` in your terminal to complete verification against the organization project dashboard.

---

## 7. Failure Modes & Recovery Runbooks

### A. chezmoi Partial Provisioning Failure

**Condition:** `chezmoi init` terminates unexpectedly mid-execution due to token dropouts or shell initialization script locks.

**Recovery:**

1. Do **not** re-execute the initialization command — this will corrupt partial state files.
2. Run `chezmoi diff` to locate what files were cleanly parsed.
3. Execute `chezmoi apply --dry-run` to trace remaining un-indexed assets safely.
4. If configuration states are broken, force a reset:
```bash
cd ~/.local/share/chezmoi && git reset --hard HEAD
chezmoi apply
```

---

### B. BWS Token Expiry Inside a DevContainer

**Condition:** Token credentials expire during a running terminal session, causing local startup scripts to return unauthorized credential errors.

**Recovery:**

```bash
bws logout
bws login
export BWS_ACCESS_TOKEN=<new-token-string>
bws run pnpm dev
```

---

### C. WSL2 Memory Exhaustion (16GB Host Nodes)

**Condition:** Simultaneous compilation runs across multiple microservices cap out the allocated Linux memory pool, resulting in silent OOM compilation drops or complete console unresponsiveness.

**Recovery:**

1. Open a native Windows administrative PowerShell window and force-kill the virtualization subsystem:
```powershell
wsl --shutdown
```
2. Wait 10 seconds for the host OS memory registers to clear.
3. Confirm recovery in Windows Task Manager — verify the `Vmmem` process footprint has dropped below 500MB.
4. Re-initialize Linux: `wsl`
5. Boot storage runtimes **sequentially** to prevent memory spikes:
   - Redpanda first
   - Database DevContainers second
   - Cursor project third

---

## 8. RAM Budget Reference

### Windows Node — MSI Raider (16GB Host, 10GB WSL2 Hard Cap)

| Component | Allocation |
|---|---|
| Windows Host OS + Native Background Daemons | 3.5 GB |
| WSL2 Linux Kernel Active Overhead | 0.8 GB |
| Cursor IDE + Active Language Servers | 1.3 GB |
| Active Polyglot DevContainer Instances (Node/PHP/Go) | 1.2 GB |
| PostgreSQL Relational Container | 0.5 GB |
| Redpanda Event Streaming Broker Container | 0.4 GB |
| Brave Browser (4–6 Active Documentation Tabs) | 1.1 GB |
| **Total Active System Consumption** | **~8.8 GB** |

### macOS Node — MacBook Air M4 (16GB Unified Memory)

| Component | Allocation |
|---|---|
| macOS Base System | 2.8 GB |
| Cursor IDE + Runtime Compilers | 1.0 GB |
| OrbStack Container Virtualisation | 1.5 GB |
| Redpanda Broker Container inside OrbStack | 0.35 GB |
| Brave Browser Research Profile (4–6 Active Tabs) | 1.0 GB |
| **Total Active System Consumption** | **~6.65 GB** |

---

## 9. Code Quality & Cross-Service Testing

### Testing Stack & Coverage

| Service | Test Framework | Minimum Coverage |
|---|---|---|
| Node.js | Vitest | 80% line coverage |
| .NET 10 | xUnit | 80% line coverage |
| Go | Native benchmarks + testify | 80% line coverage |
| PHP/Laravel | PHPUnit | 80% line coverage |

Integration checks run inside ephemeral DevContainers with direct access to live Postgres and Redpanda instances. Contract testing is driven by **Pact** to guarantee polyglot microservice schemas never drift out of alignment.

### Global Lefthook Pre-Commit Enforcement

To guarantee that cross-language serialization models (Protobuf schemas) never drift, the pre-commit configuration blocks integration whenever an interface change occurs. Lefthook mandates absolute contract testing across the entire system whenever a shared Protobuf schema file registers a change:

```yaml
# lefthook.yml
pre-commit:
  parallel: true
  commands:
    contract-validate:
      run: pnpm turbo test:contract
    lint:
      run: pnpm lint --filter=[HEAD^1]
    trivy:
      run: trivy fs . --severity CRITICAL --exit-code 1
```

This prevents broken API footprints from entering the repository at any point in the development cycle.
