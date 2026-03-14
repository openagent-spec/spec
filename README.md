# OpenAgent Spec

<p align="center">
  <img src="logo.png" width="128" alt="OpenAgent" />
</p>

<p align="center">
  <strong>Universal manifest format for AI agents.</strong>
</p>

<p align="center">
  <a href="https://openagent.vercel.app">Website</a> ·
  <a href="SPEC.md">Spec</a> ·
  <a href="https://github.com/openagent-spec/registry">Registry</a> ·
  <a href="https://github.com/openagent-spec/sdk-go">Go SDK</a> ·
  <a href="https://github.com/openagent-spec/sdk-js">JS SDK</a>
</p>

---

Universal manifest format for AI agents.

## What is this?

`agent.yaml` is a declarative manifest that describes an AI agent — identity, persona, skills, experience, and marketplace metadata. It's **framework-agnostic**: any runtime can load and execute an OpenAgent-compliant agent.

## Quick Start

```yaml
# agent.yaml — minimal
id: "my-agent"
name: "My Agent"
version: "1.0.0"
description: "A helpful assistant"
persona:
  style: "friendly and precise"
  tone: "professional"
```

## Specification

📖 **[Read the full spec →](SPEC.md)**

## Schema

- [`schema/agent.schema.json`](schema/agent.schema.json) — Agent manifest schema
- [`schema/experience.schema.json`](schema/experience.schema.json) — Experience pack schema

## Examples

- [`examples/rust-proxy-expert.yaml`](examples/rust-proxy-expert.yaml) — Full-featured agent

## SDKs

| Language | Package | Status |
|----------|---------|--------|
| Go | [`openagent/sdk-go`](https://github.com/openagent/sdk-go) | 🔜 |
| JavaScript/TypeScript | [`openagent/sdk-js`](https://github.com/openagent/sdk-js) | 🔜 |

## Registry

Agents are distributed via Git-based registries: [`openagent/registry`](https://github.com/openagent/registry)

## License

Spec: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
