[TOC]



# <img src="C:\Users\Michal Charvat\OneDrive\Documents\Stellora.AI\Logo\small.png" style="zoom:25%;" /> Stellora.AI  Documentation

## Overview

## What is Stellora.AI?

Stellora.AI is an “agentic, multilayer RAG engine” focused on speed, precision, and trust. It orchestrates multiple specialized agents that cross-check one another to return quick, hallucination-resistant answers grounded strictly in your data. 

##  System Requirements

> These specs target a **single Stellora.AI instance with the on-prem orchestrator** solving a complex task (e.g., your “quantum problem” scenario). Stellora can run on smaller boxes for light workloads; the website notes cases as low as **1 CPU / 4 GB RAM / 8 GB storage** for minimal setups;

### Minimum (for the described orchestrator workload)

| Resource    | Spec                                                         |
| ----------- | ------------------------------------------------------------ |
| **CPU**     | 2 vCPU (≈ 2.5 GHz or better)                                 |
| **RAM**     | 8 GB                                                         |
| **Disk**    | 120 GB SSD (project data + vectorized twin + logs)           |
| **OS**      | Ubuntu **LTS** (22.04/24.04); 64-bit server install recommended. [documentation.ubuntu.com](https://documentation.ubuntu.com/server/reference/installation/system-requirements/?utm_source=chatgpt.com) |
| **Network** | Stable internet **< 50 Mbit/s** is sufficient; low latency helps with API calls. |
| **GPU**     | **Not required** (CPU-only works).                           |
| **Access**  | Outbound HTTPS to OpenAI/Azure/AWS endpoints (TLS; API-key auth). |

- Notes & rationale

- **Small-footprint mode:** The official site highlights efficiency—*“in some cases, it can run efficiently on just 1 CPU, 8 GB of storage, and 4 GB of RAM”*—useful for demos or tiny corpora. Scale up for agent orchestration and larger vector stores. 
- **Ubuntu LTS:** Chosen for stability and long support; baseline server requirements are modest, so Stellora’s needs are driven mainly by your corpus size and concurrency. [documentation.ubuntu.com](https://documentation.ubuntu.com/server/reference/installation/system-requirements/?utm_source=chatgpt.com)
- **Network:** Stellora communicates with OpenAI (or Azure/AWS OpenAI) via **secure HTTPS (TLS)**; bandwidth mainly impacts latency for API calls and model streaming rather than raw throughput.
- **No GPU required:** CPU-only is fine; GPUs mainly buy speed for local open-weight models, which Stellora does not require for the stated setup.

If you want, I can add a copy-pasteable `requirements.md` block to your repo and a simple capacity calculator (inputs: docs size, avg chunk count) to estimate disk/RAM headroom for the vectorized twin.

## Licensing

> **Short version:** Stellora.AI is proprietary. You may use it only within the scope and duration of a written agreement from the Author (below). It also includes third-party open-source libraries, each under its own license.

### Technology

The solution is based primarily on Python and secure Python scripts/programs, which are likely proprietary, with support for x86_64 and ARM devices

### Grant & Scope

- **License Type:** Non-exclusive, non-transferable, revocable license to install and use Stellora.AI strictly for the purposes stated in your **Agreement** (proposal, order form, or MSA/SOW) and for the **Term** defined there.
- **No Implied Rights:** Any usage not explicitly permitted is prohibited. All rights not expressly granted are reserved by the Author.

### Consent & Term

- **Author’s Consent Required:** Use of Stellora.AI requires the Author’s prior written consent.
- **Term-Limited:** Your rights expire at the end of the agreed **Term** (e.g., trial, PoC, subscription).
- **No Overrun:** Continued use **after expiry** or **outside scope** (e.g., additional environments, seats, or workloads) is not allowed **without new written consent** from the Author.

### Restrictions

- No sublicensing, renting, or hosting for third parties.
- No reverse engineering, decompiling, or circumventing technical limits except where mandatory by law.
- No use to build a substantially similar or competing product.
- No removal or alteration of notices, trademarks, or license headers.

### Ownership

- Stellora.AI, its components, documentation, and all improvements remain the **exclusive property of the Author**.
- You retain ownership of your data and configurations.

### Third-Party Open-Source Components

Stellora.AI includes third-party open-source libraries. These are **not** licensed under this proprietary license; they are licensed under their respective OSS licenses. Your use of Stellora.AI **constitutes acceptance of those OSS terms**.

- **No Warranty Transfer:** Third-party components are provided by their respective authors “as is”.

### Support & Updates

- Any support, updates, or security fixes are provided **only** as set out in your Agreement. No obligation exists outside that Agreement.

### Termination

- The Author may terminate the license for material breach (including unlicensed use or overrun) if not cured within the cure period stated in the Agreement. Upon termination or expiry, you must **cease all use** and delete/return copies.

### Warranty Disclaimer; Limitation of Liability

- Stellora.AI is provided **“as is”** except as expressly warranted in your Agreement.
- To the maximum extent permitted by law, the Author **disclaims all implied warranties** and **limits liability** as specified in the Agreement.

## Why it’s different

- **Multilayer agentic orchestration** — for complex queries, Stellora.AI cam spins up a swarm of task-specific agents that re-retrieve, validate, and synthesize results into a single, unified answer. This layered reasoning is designed to push beyond single-model RAG systems while maintaining guardrails. 
- **Hallucination control** — outputs are positioned as “hallucination-free” within the provided context, with optional internal validation steps.
- **Lean by design** — the engine emphasizes minimal resource usage and model calls; in some situations it can operate on very modest hardware. 
- **Sovereignty options** — built in the EU and compatible with models that can be hosted in sovereign data centers or tenants (e.g., AWS/Azure). 

## Core components

### Stellora.AI Core (Multilayer RAG Engine)

- Ingests your docs/code/logs, vectorizes them, and is able to work with them
- Keeps vector stores on-prem (when required) and emphasizes secure, structured retrieval. 

###  Stellora.AI Flows

- A Python-based framework built atop the core to automate AI-driven tasks and embed agentic reasoning into operational workflows.
- Ships with example flows such as **Term Shield**—a control layer that restricts and audits CLI/SSH-like actions within flows to enforce enterprise guardrails.

###  Stellora.AI webUI

- A “mission control” front-end for real-time interaction and debugging.
- Traces prompts, intermediate agent calls, and final outputs to give transparent, end-to-end visibility for teams. 

## High-level architecture

![schematic](C:\Users\Michal Charvat\Desktop\schematic.png)

### Stellora.AI — Dataflow (from the schematic)

- **Inputs**
  - Supported data (files, code, logs…) are fed in via **standard UNIX I/O** (stdin, file descriptors).
  - Optional **query / additional context** can be supplied the same way.
- **Core ingestion & twin**
  - Stellora.AI Core (SaaS or on-prem) **reads documents via UNIX I/O**.
  - It creates a **Vectorized Twin** of your corpus and **writes it to a predefined on-disk location using UNIX I/O** (no proprietary datastore required).
- **Retrieval & reasoning**
  - Stellora.AI **queries the Vectorized Twin using mathematical vector search** (similarity over embeddings).
  - For broader reasoning or tool use, Stellora.AI can **forward sub-queries to OpenAI GPT models**.
    - Communication with OpenAI is performed **over penAI API or cloud tenants (Azure OpenAI, AWS endpoints)** while keeping **your data and vectors local**/in controlled environment.
- **Hallucination prevention**
  - Retrieved context from the Vectorized Twin is **validated / cross-checked** before synthesis to reduce hallucinations.
- **Agentic orchestration (on-prem)**
  - A **multilayer agent orchestrator** runs **on-premises**.
  - It receives the user query and any follow-up tasks, then **spawns additional Stellora.AI daemons** as needed.
  - Each spawned daemon **reuses the same ingestion → retrieval → validation path** described above.
- **Flows (automation)**
  - **Stellora.AI Flow** (SaaS or on-prem) consumes validated answers and **executes follow-on actions** (scripts, tools, integrations) under policy.
- **Outputs**
  - Final results are exposed either:
    - on an **output UNIX pipe** (stream-friendly, CLI/ops use), **or**
    - via an **MCP server endpoint**, depending on the selected Stellora.AI mode.
    - or acttion defined by the Flo

# Usage

## Stellora.AI Core

TBC