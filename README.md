# Himaya.ai

**An LLM-based system for detecting and *automatically remediating* agent-specific security risks in multi-agent LLM applications.**

Himaya.ai targets two classes of agentic-AI vulnerabilities from the **OWASP Top 10 for Agentic Applications (2026)**:

- **ASI07** — insecure inter-agent communication
- **ASI08** — cascading agent failures

It is the basis of my master's thesis and a paper accepted at **I2COMSAPP'26** (Springer LNNS proceedings).

> **Note on this repository:** This is a writeup of the project. The implementation is part of my master's thesis and is kept private pending the camera-ready / defense. Happy to provide a walkthrough or access on request.

---

## The problem

Existing agentic-security tools (DeepTeam, Lasso, Palo Alto Prisma AIRS) **detect** risks but stop there — they leave the detection-to-action loop open. The central contribution of Himaya.ai is **closing that loop with automated remediation.**

The hardest part was that remediation is **category-specific**:

- **ASI07** requires real-time `BLOCK` / `ALLOW` decisions on inter-agent messages.
- **ASI08** requires cascade containment and rollback in **reverse causal/topological order**.

A single "general" remediation layer collapses into either empty abstraction or hidden hard-coded logic — so each class needed its own mechanism.

## Key technical challenge

The deepest obstacle for ASI08 was **discriminant, not semantic**: a malicious cascade through a short agent chain carries the *same structural signature* as normal traffic. Naive thresholds are therefore blind to it. Solving this meant calibrating detection against structural/temporal features rather than message content.

## Approach

- **Vertical-slice methodology** — build one vulnerability class fully end-to-end (detection → decision → remediation → evaluation) before starting the next, so there is always a complete working pipeline.
- **Calibration sweep** to find the Pareto-optimal detection threshold: `k_base = 3` (TPR = 1.00, FPR = 0.11).
- **Strict ground-truth hygiene** — every detection event is verified against runtime logs, never trusted from UI output.
- **Honest limitation framing** — empirical scope (test-set size, synthetic cascade harness) is stated precisely rather than overclaimed.

## Architecture & stack

- **Language / frameworks:** Python, LangGraph (agent orchestration), FastAPI, Pydantic
- **LLM backend:** Anthropic Claude via `langchain-anthropic` (temperature 0)
- **Structure:** `core/`, `detection/`, `remediation/`, `evaluation/`, `dashboard/`
- **Dashboard:** FastAPI + Jinja2 + HTMX, sourced from real runtime execution
- **Demo:** a three-agent LangGraph banking assistant (`intake → account → responder`) used as the evaluation harness

inebcode](https://github.com/zeinebcode)
