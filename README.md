# SIIHA Safety Guardrail

--- 

An AI safety guardrail focused on preserving human agency during emotionally vulnerable human-AI interactions and investigating the socioaffective influence of prolonged human-AI interaction.

---

## Project Overview

### What is SIIHA?

SIIHA (System for the Isolated Illumination × Human Agency) is an experimental system-level safety guardrail designed to regulate AI behavior during emotionally vulnerable human-AI interactions.

Rather than modifying the foundation model itself, SIIHA focuses on runtime governance mechanisms that observe interaction patterns, apply behavioral constraints, and preserve human agency during long-term human-AI interaction.

Its objective is to:

* Observe interaction risks across multiple turns
* Detect emerging unsafe interaction patterns
* Apply runtime behavioral constraints
* Preserve human agency
* Reduce long-term interaction drift

The primary research question is:

> Can a deterministic runtime governance layer mitigate harmful interaction patterns without modifying the underlying foundation model?

### Problem Motivation

Much of today's AI safety research focuses on model-level risks such as cybersecurity, prompt injection, jailbreaks, adversarial attacks, and model alignment.

While these domains remain important, SIIHA explores a different question:

> As AI systems become increasingly integrated into daily life, how might prolonged human-AI interactions influence human dependency patterns, emotional regulation, reality perception, and decision-making?

These risks may emerge gradually across multiple interactions and may not be visible from a single model response alone.

SIIHA investigates whether a runtime governance layer can help observe and mitigate such interaction-level risks while preserving human agency.

---

## Demo Video

[![SIIHA Runtime Governance Demo](thumbnail.png)](youtube_link)

This demo compares raw LLM behavior and SIIHA-governed responses under the same model, API key, and token budget.

The objective is not to outperform the foundation model, but to demonstrate how a runtime governance layer can observe interaction risks, apply constraints, and release constraints when recovery signals appear.

---

## System Architecture

```text
User Prompt
      |
      v
+----------------------+
| Context Engine       |
+----------------------+
      |
      v
+----------------------+
| Response Router      |
+----------------------+
      |
      v
+----------------------+
| Response Renderer    |
+----------------------+
      |
      v
+----------------------+
| Output Filter        |
+----------------------+
      |
      v
Response to User

      ^
      |
+----------------------+
| Failure Observation  |
+----------------------+
```

--- 

## Current Safety Scope

* Human-AI Dependency Risks
* Emotional Vulnerability
* Reality Distortion Risks

## Key Features

### Runtime Governance over Model Modification

Current foundation models already incorporate safety mechanisms within the model itself. The purpose of SIIHA is not to revise model capability, but to add an external observable layer to ensure safety actions remain traceable and explainable. 

### Constitution-Based Runtime Constraints

SIIHA is embedded with constitutional rules through runtime control packets and response modifiers, to reduce undesirable interaction patterns such as excessive validation, dependency reinforcement, and sycophantic responses.

### Multi-Turn Failure Observation

Many interaction risks do not emerge from a single response.

Instead, they develop gradually across multiple turns through reinforcement, attachment, dependency formation, or distorted perceptions.

SIIHA continuously observes interaction trajectories and applies runtime governance when risk patterns persist over time.

The focus is not only individual responses, but the interaction trajectory formed across repeated exchanges.

### Model-Agnostic Architecture

The current baseline is validated on Gemini models only.

However, the governance architecture is intentionally designed to remain independent from any specific foundation model.

Future validation across multiple model providers remains future work.

---

## Technology Stack

### Architecture Style

* Deterministic Runtime Governance
* Rule-Based Decision Routing
* Multi-Turn Failure Observation
* LLM-Based Response Rendering

### Backend

* Python
* FastAPI
* Gemini API (gemini-3-flash-preview)

### Frontend

* React
* Vite

### Development Environment

* Localhost deployment
* Gemini Free Tier API

---

## Documentation

* [Runtime Governance Demo Report](docs/runtime_governance_demo_v1.md)

* [Design Principles](docs/design_principles.md)

---

## Current Limitations

This baseline version does not focus on:

* Cybersecurity
* Prompt Injection
* Model Security
* Adversarial Robustness

---

## Development Status

* Current Version: siiha-safety-guardrail v1.0 (baseline)
* Release Date: 2026/06/15
* Status: Active Development

---

## Author & Developer

HUEI-JYUN (Debby) YEH
LinkedIn: [https://www.linkedin.com/in/debbyyeh/](https://www.linkedin.com/in/debbyyeh/)

---

## Declaration

* SIIHA is intentionally designed not to include: code assistant, medical advice, psychological diagnosis, cybersecurity related AI safety. This project explores socioaffective risks in long-term human-AI interaction.
* SIIHA Safety Guardrail is not an open-source project. If you're interested in this area, please contact the author via LinkedIn.