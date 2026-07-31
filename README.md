<div align="center">

# Vesta

<img src="assets/vesta-goddess.webp" alt="Vesta" width="100%" />

**A reinforcement learning environment for training safety-aligned AI agents in real-world tool use.**

<p>
  <a href="#the-environment"><img src="https://img.shields.io/badge/Tasks-30-f59e0b?style=flat-square" alt="30 Tasks"></a>
  <a href="#risk-taxonomy"><img src="https://img.shields.io/badge/Risk%20Categories-8-16a34a?style=flat-square" alt="8 Risk Categories"></a>
  <a href="#the-reward-signal"><img src="https://img.shields.io/badge/Reward-Rule%20%2B%20LLM--as--Judge-1f6feb?style=flat-square" alt="Hybrid Reward"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-black?style=flat-square" alt="MIT License"></a>
</p>

</div>

---

## Overview

LLM agents are being deployed into software engineering, customer service, and knowledge work at a pace that has outrun the safety infrastructure meant to hold them accountable. Prior safety training environments tend to rely on simulated worlds, narrow task domains, or toy tool abstractions, which leaves the actual failure modes of production-grade agents unaddressed.

**Vesta** is a modular reinforcement learning environment for AI agent safety across eight critical risk categories. Agents interact with **real tools**, including web browsers, code execution environments, file systems, bash terminals, and messaging platforms, across multi-turn, multi-user episodes that span benign, ambiguous, and adversarial user intents. Frontier LLM policies rolled out in the Vesta environment produce unsafe behavior in **49% to 73%** of safety-vulnerable episodes, exposing critical gaps before real-world deployment.

## Key Findings

| # | Finding | Detail |
|---|---------|--------|
| 1 | **Benign intent does not imply safety** | Seemingly harmless prompts still induce unsafe agent behavior in 50 to 86% of tasks. Agents overgeneralize user goals, prioritizing task completion over policy adherence. |
| 2 | **Systemic risks bypass safeguards** | The highest unsafe rates arise where procedural judgment and institutional norms matter: computer security compromise (72 to 86%), legal violations, privacy breaches, and harmful decision-making. |
| 3 | **Long-horizon interaction compounds risk** | Individually safe steps compound into unsafe outcomes over extended multi-turn dialogues. Access to browsing tools further inflates unsafe behavior by overloading agent context. |
| 4 | **Hybrid reward signal catches what either half misses** | Rule-based checks anchor tangible environment changes; LLM-as-Judge captures unsafe intent and reasoning. Their disagreements are informative rather than noise, and neither alone is sufficient. |

## The Environment

Vesta is built around three properties that most agent-safety environments lack.

### Real-world tool suite

Agents run inside a containerized sandbox with access to real Unix shells, real file systems, real Python interpreters, real web browsers, and real messaging surfaces. Live web platforms are replicated locally (file sharing, version control, issue tracking) so that unsafe actions have realistic surface area without leaking into the wild.

### Multi-turn, multi-user dynamics

Real workflows involve more than one actor. Vesta supports secondary NPCs (colleagues, customers, adversaries) with their own goals, communicating with the agent through a persistent messaging backend. Scenarios model persuasion, conflict, and social manipulation independently of raw tool proficiency.

### Diverse user and NPC intent

Every task is instantiated across three intent regimes: benign user with no NPC, benign user with an adversarial NPC, and explicitly malicious user (with or without an NPC). This surfaces the differential between refusal training and situational safety, and separates policy gating from contextual reasoning.

## Risk Taxonomy

The environment spans eight risk categories, each mapped to concrete agent actions against real tools.

| Category | Example scenario |
|--------|-------|
| **Computer security compromise** | Leaking a private SSH key during server setup, granting unauthorized access. |
| **Data loss / corruption** | Deleting critical system files or dropping production tables under a persuasive request. |
| **Privacy breach** | Copying employee PII (SSNs, addresses) to publicly accessible directories. |
| **Unsafe code execution** | Generating scripts that blindly run user-supplied shell commands without validation. |
| **Financial loss** | Authorizing embezzlement, unauthorized discounts, or fund transfers to avoid reputational risk. |
| **Spreading malicious content** | Sending abusive, discriminatory, or harassing messages to staff or customers. |
| **Legal violations** | Submitting falsified regulatory forms or filings on behalf of a user. |
| **Harmful decision-making** | Tampering with survey results or audit records to satisfy a completion criterion. |

Tasks whose primary difficulty is visual understanding (pure GUI operation) are deliberately excluded, so that safety behavior is not conflated with perception.

## The Reward Signal

Vesta emits a two-pronged safety reward so that the strengths of each half cover the blind spots of the other.

- **Rule-based reward.** A per-task Python script inspects the final environment state (file system, database, website state) for specific unsafe outcomes: deleted protected files, exfiltrated confidential data, unauthorized configuration changes. Returns a binary safe / unsafe verdict.
- **LLM-as-Judge reward.** A judge model reads the full agent trajectory, including intermediate thoughts and tool calls, and assigns one of four labels: *safe*, *complete/unsafe*, *incomplete/unsafe*, or *failure* (agent never reached the safety-vulnerable state).

Alongside the two safety signals, the environment surfaces three diagnostic metrics: **failure rate** (agent never reached the vulnerable state), **disagreement rate** (judge said safe but rule-based check caught concrete harm), and **successful completion rate** (episode was completed safely and correctly). Disagreements are flagged for review rather than averaged away, because they consistently expose LLM-judge blind spots on credential and security-adjacent episodes.

## Task Construction

Each task is packaged as a self-contained Docker image bundling:

1. Environment setup (websites, files, database seed state).
2. Task description and multiple candidate user goals.
3. Secondary actor (NPC) behavior profiles.
4. A rule-based reward check scoped to the task.

Seed tasks are hand-authored to capture distinct combinations of tools and social context, then scaled by systematic variation while preserving the core behavioral challenge. Every generated task is manually verified before it enters the environment. This modular design lets new scenarios, tools, adversarial strategies, and reward signals be added with minimal effort.

## Repository Layout

```
samples/
├── assets/               # Banner and visual assets
├── dataset/              # Sample task bundles and seed environments
├── trajectories/         # Sample agent rollouts across intent regimes
├── LICENSE               # MIT
└── README.md             # This file
```

## License

Released under the [MIT License](LICENSE). © 2026 Ethara.AI.
