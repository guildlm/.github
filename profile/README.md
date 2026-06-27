<div align="center">

# GuildLM ⚔️

### Specialist Models. Mastered.

**One master per domain. One brain to coordinate them all.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Local-First](https://img.shields.io/badge/local-first-C5A55A.svg)](#six-principles)
[![Open Source](https://img.shields.io/badge/open-source-2ea44f.svg)](https://github.com/guildlm)

</div>

---

GuildLM is an open-source platform that trains **small, specialist LLMs (SLMs)**,
groups them into domain **guilds**, and routes every request through a central
**brain**. It is the antithesis of the "one giant model that knows everything"
approach: instead of a single 500B generalist, GuildLM runs dozens of sharp
3B–8B masters — each fine-tuned for one job — and a router that decides which one
should answer.

> **Is this Go code? → Code Guild. A contract? → Legal Guild.**
> The brain never answers itself; it classifies, routes, and orchestrates.

Smaller specialists are cheaper to train, faster to serve, easier to evaluate,
and — because every adapter is a LoRA over a shared base — they hot-swap inside a
single VRAM budget. The whole system is designed to run on **one machine**.

---

## Architecture

```
                          ┌──────────────┐
                          │     USER      │
                          └──────┬───────┘
                                 │
                          ┌──────▼───────┐
                          │  🧠 BRAIN     │   classify · route · orchestrate
                          │   (router)    │   (never answers itself)
                          └──────┬───────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                         │
 ┌──────▼──────┐          ┌──────▼──────┐           ┌──────▼──────┐
 │ ⚔️  CODE     │          │ ⚖️  LEGAL    │           │ 🏥 MEDICAL   │
 │   GUILD      │          │  (planned)   │           │  (planned)   │
 │ live ✅       │          │              │           │              │
 │ go_generator │          │ contract     │           │ diagnosis    │
 │ go_reviewer  │          │ compliance   │           │ literature   │
 │ go_tester    │          │ case_law     │           │ drug_interac │
 │ go_explainer │          │ ip_law       │           │ radiology    │
 └─────────────┘          └─────────────┘           └─────────────┘

       Each specialist is built by the same four-tool pipeline:

       forge ──▶ anvil ──▶ crucible ──▶ brain
       (data)    (train)    (evaluate)    (serve & route)
```

The four core tools are **domain-agnostic** — they know nothing about Go, SQL or
law. All domain knowledge lives in the **guild repos** (configs, prompts, eval
suites). Adding a new domain means adding a new `guild-*` repo, not touching the
engine.

---

## Repositories

| Repo | What it is | Type |
| --- | --- | --- |
| [**forge**](https://github.com/guildlm/forge) | Data pipeline — discover → download → process → generate → build. Turns sources (GitHub, arXiv, …) into SFT datasets via teacher models. | 🟡 Core |
| [**anvil**](https://github.com/guildlm/anvil) | Training infra — QLoRA SFT, DPO, LoRA merge, and quantization (GGUF/GPTQ/AWQ) for any base model. | 🟡 Core |
| [**crucible**](https://github.com/guildlm/crucible) | Evaluation — pluggable evaluators (sandboxed `go_functional`, `llm_judge`, `safety`) with JSON/Markdown reports. | 🟡 Core |
| [**brain**](https://github.com/guildlm/brain) | The router/orchestrator — classifies intent, routes to a specialist, hot-swaps LoRA adapters, runs multi-step pipelines. | 🟡 Core |
| [**guild-code**](https://github.com/guildlm/guild-code) | ⚔️ The first guild: Go specialists (`go_generator`, `go_reviewer`, `go_tester`, `go_explainer`) with forge/anvil/crucible recipes. | 🔵 Guild |
| [**guild-template**](https://github.com/guildlm/guild-template) | 📋 Boilerplate + `new_guild.sh` generator for standing up a new guild in minutes. | 🟣 Template |
| [**guildlm.github.io**](https://github.com/guildlm/guildlm.github.io) | The landing site and public face of the project. | ⚪ Site |

> **Status:** the four core tools are production-grade (pyproject packaging,
> pytest, CI, rich docs). **guild-code** is the first complete guild spec.

---

## How it flows

A single request through the Code Guild's `bug_fix` pipeline:

```
User: "Find and fix the race condition in this Go code"
  │
  ▼  🧠 brain.classify()  →  domain: code · language: go · task: bug_fix
  │  🧠 brain.route()     →  guild-code
  ▼
  1. go_reviewer   analyze →  diagnoses the data race
  2. go_generator  fix     →  applies a mutex / channel fix
  3. go_reviewer   verify  →  confirms no regressions
  ▼
Returned to the user.
```

The same loop builds every specialist:

1. **forge** — a data recipe (`go/forge/<id>.yaml`) builds an SFT dataset.
2. **anvil** — a training recipe (`go/anvil/<id>.yaml`) QLoRA-tunes an adapter.
3. **crucible** — an eval suite (`go/crucible/<id>.yaml`) scores it.
4. **brain** — the adapter is registered and served behind the router.

---

## Six principles

| Principle | Meaning |
| --- | --- |
| **Guild System** | Every domain is a guild containing multiple specialist SLMs. |
| **Brain First** | The brain identifies the task and routes it; it never answers itself. |
| **Domain Isolation** | Each SLM knows its own domain only — sharper, smaller, faster. |
| **Composable** | Adding a domain = adding a guild. The core engine never changes. |
| **Local-First** | The whole system runs on a single machine. |
| **Open Source** | Everything is open and community-driven. |

---

## Roadmap

- **Phase 1 — Foundation.** Landing site + core repo scaffolding. ✅
- **Phase 2 — forge.** Domain-agnostic data pipeline. ✅
- **Phase 3 — anvil.** Training infra (SFT/DPO/QLoRA/merge/quantize). ✅
- **Phase 4 — crucible.** Pluggable evaluation + sandboxes. ✅
- **Phase 5 — brain.** Router, registry, multi-step pipelines. ✅
- **Phase 6 — guild-code.** First guild: the Go specialists. ✅ (spec complete)
- **Phase 7 — guild-template.** One-command new-guild boilerplate. ✅
- **Next — more guilds.** SQL, Rust, Legal, Medical… built from the template.

---

## Get involved

- 🚀 **Build a guild:** start from [guild-template](https://github.com/guildlm/guild-template)
  — `./new_guild.sh guild-sql sql sql_writer`.
- ⚔️ **Sharpen the Code Guild:** add specialists or datasets in
  [guild-code](https://github.com/guildlm/guild-code).
- 🛠️ **Improve the core:** issues and PRs welcome on
  [forge](https://github.com/guildlm/forge),
  [anvil](https://github.com/guildlm/anvil),
  [crucible](https://github.com/guildlm/crucible) and
  [brain](https://github.com/guildlm/brain).
- 📖 Every repo ships a README and `CONTRIBUTING.md` — that's the place to start.

<div align="center">

**Everything is Apache-2.0. Come forge a master.**

</div>
