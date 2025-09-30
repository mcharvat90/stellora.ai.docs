[TOC]



# <img src="C:\Users\Michal Charvat\OneDrive\Documents\Stellora.AI\Logo\small.png" style="zoom:25%;" /> Stellora.AI  Documentation

## Overview

## What is Stellora.AI?

Stellora.AI is an “agentic, multilayer RAG engine” focused on speed, precision, and trust. It orchestrates multiple specialized agents that cross-check one another to return quick, hallucination-resistant answers grounded strictly in your data. 

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

    - or acttion defined by tje Flow

      