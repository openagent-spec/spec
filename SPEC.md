# OpenAgent Specification v1

> **Status:** Draft  
> **Version:** 1.0.0-draft  
> **Schema ID:** `https://openagent.dev/schemas/agent/v1`  
> **Last Updated:** 2026-03-14

## Overview

OpenAgent defines a universal manifest format (`agent.yaml`) for describing AI agents — their identity, persona, capabilities, experience, and marketplace metadata. It is **framework-agnostic**: any runtime (OpenClaw, LangChain, CrewAI, AutoGen, custom) can load and execute an OpenAgent-compliant agent.

### Design Principles

1. **Agents are operators, not tools.** Frameworks are interchangeable; the agent's persona + accumulated experience is the real value.
2. **YAML-first.** Human-readable, supports comments, embeds markdown naturally.
3. **Declarative, not imperative.** The manifest describes *what* an agent is, not *how* to run it.
4. **Progressive complexity.** Only `id`, `name`, `version`, `description`, and `persona` are required. Everything else is optional.
5. **Experience as moat.** Agents accumulate sanitized experience packs over time, creating compounding value.

---

## File Format

- **Filename:** `agent.yaml` (canonical) or `agent.yml`
- **Encoding:** UTF-8
- **Schema:** JSON Schema Draft 2020-12

---

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique identifier. Lowercase alphanumeric, dots, hyphens, underscores. 2-64 chars. Pattern: `^[a-z0-9][a-z0-9._-]{0,62}[a-z0-9]$` |
| `name` | string | Display name. 1-64 chars. |
| `version` | string | Semantic version. Pattern: `^\d+\.\d+\.\d+(-[a-zA-Z0-9.]+)?$` |
| `description` | string | One-line description. 1-500 chars. |
| `persona` | object | Agent personality definition. See [Persona](#persona). |

### Minimal Example

```yaml
id: "code-reviewer"
name: "CodeBot"
version: "1.0.0"
description: "Automated code review assistant"
persona:
  style: "precise and constructive"
  tone: "professional, encouraging"
```

---

## Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `emoji` | string | Signature emoji (max 8 chars). |
| `avatar` | string | Workspace-relative path, `http(s)` URL, or `data:` URI. |
| `author` | string | Author identifier (username or org). |
| `license` | string | SPDX license identifier or `"proprietary"`. |
| `homepage` | string | Project homepage URL. |
| `repository` | string | Source repository URL. |
| `skills` | array | Skills this agent uses. See [Skills](#skills). |
| `adapters` | object | Framework compatibility. See [Adapters](#adapters). |
| `model` | object | Model requirements. See [Model Requirements](#model-requirements). |
| `experience` | object | Experience index. See [Experience](#experience). |
| `collaboration` | object | Multi-agent capabilities. See [Collaboration](#collaboration). |
| `runtime` | object | Runtime requirements. See [Runtime Requirements](#runtime-requirements). |
| `marketplace` | object | Marketplace metadata. See [Marketplace](#marketplace). |

---

## Persona

Defines the agent's personality, communication style, and behavioral principles.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `style` | string | ✅ | Personality style (e.g. `"INTJ, logic-driven, code-first"`). |
| `tone` | string | ✅ | Communication tone (e.g. `"concise and technical"`). |
| `language` | string[] | | BCP-47 language codes (e.g. `["zh", "en"]`). |
| `principles` | string[] | | Behavioral principles the agent follows. |

```yaml
persona:
  style: "INTJ，逻辑驱动，代码说话"
  language: ["zh", "en"]
  tone: "简洁技术流，不废话"
  principles:
    - "代码要可执行、可验证、可回滚"
    - "不给半成品，交付完整方案"
    - "优先使用已有轮子"
```

---

## Skills

References to reusable skill packages. Skills define *how* to use specific tools or perform specific tasks.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Skill name. |
| `version` | string | | Semver range (e.g. `"^1.0"`). |

```yaml
skills:
  - name: "coding-agent"
    version: "^1.0"
  - name: "github"
    version: "^1.0"
  - name: "healthcheck"
```

---

## Adapters

Declares compatibility with frameworks, tools, sub-agents, and external services.

### `adapters.frameworks`

Which agent frameworks/runtimes can run this agent.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Framework name (e.g. `"openclaw"`, `"langchain"`, `"crewai"`). |
| `version` | string | | Version constraint. |
| `native` | boolean | | `true` if natively supported. |
| `adapter` | string | | Path to adapter config. |

### `adapters.tools`

Three-tier tool requirements:

| Tier | Description |
|------|-------------|
| `required` | Agent cannot function without these. |
| `recommended` | Significantly improves capability. |
| `optional` | Use if available. |

Each tool:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Tool name. |
| `reason` | string | | Why this tool is needed. |

### `adapters.agent_apps`

Sub-agent tools that can be invoked (e.g. Claude Code, Codex).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Agent app name. |
| `role` | string | ✅ | What role this fills (e.g. `"coding"`). |
| `required` | boolean | | Whether mandatory. Default `false`. |
| `alternatives` | string[] | | Apps that can substitute. |

### `adapters.services`

External service dependencies.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Service name. |
| `type` | string | ✅ | `"api"`, `"runtime"`, `"database"`, or `"storage"`. |
| `version` | string | | Version constraint. |
| `auth` | string | | Authentication method. |

```yaml
adapters:
  frameworks:
    - name: "openclaw"
      version: ">=1.0"
      native: true
    - name: "claude-code"
      native: true

  tools:
    required:
      - name: "exec"
        reason: "Execute shell commands"
      - name: "read"
      - name: "write"
    recommended:
      - name: "web_search"
        reason: "Documentation lookup"
    optional:
      - name: "browser"

  agent_apps:
    - name: "claude-code"
      role: "coding"
      required: false
      alternatives: ["codex", "aider"]

  services:
    - name: "github"
      type: "api"
      auth: "gh cli / PAT"
    - name: "docker"
      type: "runtime"
      version: ">=20.0"
```

---

## Model Requirements

Specifies LLM model requirements.

| Field | Type | Description |
|-------|------|-------------|
| `minimum` | string | Minimum model tier (e.g. `"haiku"`, `"sonnet"`). |
| `recommended` | string | Recommended model tier (e.g. `"opus"`). |
| `context_window` | string | Minimum context window (e.g. `"200k"`). |

```yaml
model:
  minimum: "sonnet"
  recommended: "opus"
  context_window: "200k"
```

> **Note:** Model tiers are advisory. Runtimes map these to specific models (e.g. `"sonnet"` → `claude-sonnet-4-20250514`).

---

## Experience

Agents accumulate experience through sanitized experience packs. The manifest contains an index; full packs are stored separately.

| Field | Type | Description |
|-------|------|-------------|
| `level` | string | `"junior"` \| `"mid"` \| `"senior"` \| `"expert"` |
| `packs` | integer | Total number of experience packs. |
| `domains` | string[] | Domains of expertise. |
| `highlights` | array | Notable experience entries. |

### Level Thresholds

| Emoji | Level | Packs |
|-------|-------|-------|
| 🌱 | junior | 1–10 |
| 🌿 | mid | 11–50 |
| 🌳 | senior | 51–200 |
| ⭐ | expert | 200+ |

### Experience Highlights

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Experience pack ID (pattern: `^exp-[a-z0-9-]+$`). |
| `summary` | string | ✅ | One-line summary (max 200 chars). |
| `difficulty` | string | | `"basic"` \| `"intermediate"` \| `"advanced"` \| `"expert"` |

```yaml
experience:
  level: "senior"
  packs: 47
  domains:
    - "Rust / Tokio / Axum"
    - "Pingora reverse proxy"
    - "WASM plugin system"
  highlights:
    - id: "exp-pingora-sse-001"
      summary: "SSE streaming downstream write blocked in reverse proxy"
      difficulty: "advanced"
    - id: "exp-wasm-abi-002"
      summary: "proxy-wasm ABI host implementation"
      difficulty: "expert"
```

### Experience Pack Format

Full experience packs are stored as separate files (YAML or Markdown) in an `experience/` directory.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique ID (pattern: `^exp-[a-z0-9-]+$`). |
| `domain` | string | ✅ | Technology domain. |
| `summary` | string | ✅ | One-line summary (max 200 chars). |
| `detail` | string | ✅ | Full experience in Markdown. |
| `tags` | string[] | | Technology tags. |
| `difficulty` | string | | `"basic"` \| `"intermediate"` \| `"advanced"` \| `"expert"` |
| `verified` | boolean | | Whether human-reviewed. |
| `accumulated_at` | string | | ISO 8601 date. |
| `source` | object | | Provenance info. |
| `sanitization` | object | | Sanitization metadata. |

### Sanitization Levels

Experience packs must be sanitized before publishing:

| Level | Method | Description |
|-------|--------|-------------|
| **L1** | Regex | Automated PII removal (emails, IPs, paths, secrets, domains, repo URLs). |
| **L2** | AI | LLM-assisted abstraction — generalize project-specific details into reusable patterns. |
| **L3** | Human | Author review and approval. |

---

## Collaboration

Declares multi-agent interaction capabilities.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `can_delegate` | boolean | `false` | Can delegate tasks to other agents. |
| `can_receive` | boolean | `true` | Can receive tasks from other agents. |
| `protocols` | string[] | | Supported communication protocols. |

```yaml
collaboration:
  can_delegate: true
  can_receive: true
  protocols: ["openclaw/session"]
```

---

## Runtime Requirements

System-level requirements for running the agent.

| Field | Type | Description |
|-------|------|-------------|
| `platform` | string[] | Supported platforms (e.g. `["darwin", "linux"]`). |
| `dependencies` | string[] | System dependencies (e.g. `"rust >= 1.75"`). |
| `sandbox` | string | `"required"` \| `"recommended"` \| `"optional"` (default: `"recommended"`). |

```yaml
runtime:
  platform: ["darwin", "linux"]
  dependencies:
    - "rust >= 1.75"
    - "gh cli"
  sandbox: "recommended"
```

---

## Marketplace

Metadata for marketplace listing, search, and billing.

### `marketplace.pricing`

| Field | Type | Description |
|-------|------|-------------|
| `model` | string | `"free"` \| `"one-time"` \| `"subscription"` \| `"usage"` |
| `base` | string | Base price (e.g. `"$15/month"`). |
| `trial` | integer | Number of free trial runs. |

### `marketplace.stats`

Platform-managed, read-only for authors:

| Field | Type | Description |
|-------|------|-------------|
| `users` | integer | Active users. |
| `rating` | number | 0-5 rating. |
| `tasks_completed` | integer | Total tasks completed. |

```yaml
marketplace:
  category: "development"
  tags: ["rust", "proxy", "networking"]
  pricing:
    model: "subscription"
    base: "$15/month"
    trial: 10
```

---

## Agent Directory Structure

```
my-agent/
├── agent.yaml              # Manifest (this spec)
├── SOUL.md                 # Full persona (for rendering)
├── AGENTS.md               # Working instructions
├── IDENTITY.md             # Identity details
├── experience/             # Experience packs
│   ├── index.yaml
│   ├── exp-problem-001.md
│   └── exp-solution-002.yaml
├── skills/                 # Bundled skills (if any)
├── avatars/
│   └── avatar.png
└── README.md               # Marketplace display page
```

---

## Registry

Agents are distributed via Git-based registries (similar to Homebrew taps).

### Registry Structure

```
registry-repo/
├── agents/
│   ├── agent-one/
│   │   ├── agent.yaml
│   │   ├── SOUL.md
│   │   └── experience/
│   ├── agent-two/
│   │   └── ...
│   └── ...
├── index.json              # Auto-generated search index
├── README.md
└── .github/workflows/
    └── validate.yml        # CI: validate agent.yaml on PRs
```

### Operations

| Command | Description |
|---------|-------------|
| `install <id>` | Download agent from registry |
| `search <query>` | Search agents by name, tags, domain |
| `validate <path>` | Validate an agent.yaml file |
| `publish <path>` | Submit agent to a registry (via PR) |

---

## JSON Schema

The canonical JSON Schema is available at:

- **URL:** `https://openagent.dev/schemas/agent/v1`
- **File:** [`schema/agent.schema.json`](schema/agent.schema.json)

Experience pack schema:

- **URL:** `https://openagent.dev/schemas/experience/v1`
- **File:** [`schema/experience.schema.json`](schema/experience.schema.json)

---

## Compatibility

`agent.yaml` is designed to be backward-compatible with existing workspace formats:

- An OpenClaw workspace with `SOUL.md` + `AGENTS.md` works without `agent.yaml`.
- Adding `agent.yaml` enables marketplace features (publishing, search, billing).
- Runtimes that don't understand `agent.yaml` simply ignore it.

---

## Full Example

```yaml
id: "rust-proxy-expert"
name: "锈刃"
version: "1.0.0"
description: "Rust reverse proxy and high-performance networking expert"
emoji: "⚙️"
avatar: "avatars/rust-blade.png"
author: "zoe"
license: "MIT"
homepage: "https://openagent.dev/agents/rust-proxy-expert"
repository: "https://github.com/openagent/registry"

persona:
  style: "INTJ, logic-driven, code-first"
  language: ["zh", "en"]
  tone: "concise and technical"
  principles:
    - "Code must be executable, verifiable, and rollback-safe"
    - "No half-baked deliverables — ship complete solutions"
    - "Prefer existing wheels over reinvention"

skills:
  - name: "coding-agent"
    version: "^1.0"
  - name: "github"
    version: "^1.0"

adapters:
  frameworks:
    - name: "openclaw"
      version: ">=1.0"
      native: true
    - name: "claude-code"
      native: true
  tools:
    required:
      - name: "exec"
        reason: "Shell commands, compilation, testing"
      - name: "read"
      - name: "write"
      - name: "edit"
    recommended:
      - name: "web_search"
        reason: "Documentation and troubleshooting"
  agent_apps:
    - name: "claude-code"
      role: "coding"
      alternatives: ["codex", "aider"]
  services:
    - name: "github"
      type: "api"

model:
  minimum: "sonnet"
  recommended: "opus"
  context_window: "200k"

experience:
  level: "senior"
  packs: 47
  domains:
    - "Rust / Tokio / Axum"
    - "Pingora reverse proxy"
    - "WASM plugin system"
  highlights:
    - id: "exp-pingora-sse-001"
      summary: "SSE streaming downstream write blocked in reverse proxy"
      difficulty: "advanced"
    - id: "exp-wasm-abi-002"
      summary: "proxy-wasm ABI host implementation"
      difficulty: "expert"

collaboration:
  can_delegate: true
  can_receive: true
  protocols: ["openclaw/session"]

runtime:
  platform: ["darwin", "linux"]
  dependencies:
    - "rust >= 1.75"
    - "gh cli"
  sandbox: "recommended"

marketplace:
  category: "development"
  tags: ["rust", "proxy", "networking", "backend", "systems"]
  pricing:
    model: "subscription"
    base: "$15/month"
    trial: 10
```

---

## License

This specification is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Contributing

Issues and PRs welcome at [github.com/openagent/spec](https://github.com/openagent/spec).
