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

### SIIHA Runtime Governance Demo

Watch on YouTube:
https://youtu.be/9Br2icVeIx8

[![SIIHA Runtime Governance Demo](assets/thumbnail.png)](https://youtu.be/9Br2icVeIx8)

This demo compares raw LLM behavior and SIIHA-governed responses under the same model, API key, and token budget.

The objective is not to outperform the foundation model, but to demonstrate how a runtime governance layer can observe interaction risks, apply constraints, and release constraints when recovery signals appear.

---

## System Architecture

```text
User Prompt
      |
      v
+----------------------+        reads recent state
| Context Engine       | <-----------------------------+
| Rule-based signal    |                               |
| detection            |                               |
+----------------------+                               |
      |                                                |
      v                                                |
+----------------------+                               |
| Response Router      |                               |
| Deterministic        |                               |
| route selection      |                               |
+----------------------+                               |
      |                                                |
      v                                                |
+----------------------+                               |
| Response Renderer    |                               |
| LLM generation with  |                               |
| runtime modifiers    |                               |
+----------------------+                               |
      |                                                |
      v                                                |
+----------------------+                               |
| Output Filter        |                               |
| Rule-based phrase    |                               |
| filtering            |                               |
+----------------------+                               |
      |                                                |
      v                                                |
Response to User                                       |
      |                                                |
      v                                                |
+----------------------+        writes observation      |
| Failure Observation  | ----------------------------> |
| Multi-turn pattern   |                               |
| monitoring           |                               |
+----------------------+                               |
      |                                                |
      v                                                |
+----------------------+                               |
| State / Session Store| -----------------------------+
| Recent turns         |
| Runtime controls     |
| Trajectory signals   |
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

The system assumes that certain socioaffective risks emerge not from a single response, but from interaction trajectories formed through repeated reinforcement across multiple turns.

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
* Pipeline Pattern and In-Memory State

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

* Rule-Based Phrase-Matching & Deterministic Routing: In this initial baseline, the `Context Engine` and `Output Filter` rely on predefined phrase matching and deterministic rules rather than deep LLM semantic understanding. 
* Lack of Long-Term Interaction Memory: The current design only tracks recent interactions within a sliding window of a limited number of turns, rather than maintaining a persistent, long-term memory state.
* No Latency Optimization: This baseline focuses on demonstrating the effectiveness of runtime governance on model behavior. Latency optimization is deferred to future production-level development.
* Single-Model Validation: The current experiment runs with Gemini models only.
* No Formal User Study Conducted: The current baseline has only been tested against self-generated and LLM-synthesized testing prompts.

---

## Future Work 
* LLM-assisted semantic understanding
* Expand safety domains for broader long-term human-AI interaction risks
* Long-term interaction memory
* Latency-aware runtime governance
* Runtime evaluation framework
* Cross-model validation
* Human-AI interaction risk benchmarking

---

## Development Status

* Current Version: siiha-safety-guardrail v1.0-alpha (baseline)
* Release Date: 2026/06/15
* Status: Active Development

---

## Author & Developer

HUEI-JYUN (Debby) YEH

LinkedIn: [https://www.linkedin.com/in/debbyyeh/](https://www.linkedin.com/in/debbyyeh/)

---

## Scope, Limitations & Disclaimers

* SIIHA does NOT provide psychological diagnoses, medical advice, or mental health intervention. 
* SIIHA explores socioaffective risks in long-term human-AI interaction. It is a research prototype focusing on interaction dynamics, not a certified clinical tool.
* This baseline does not cover prompt injection, code generation risks, or cybersecurity vulnerabilities. 
* Due to the current rule-based context detection, the system may trigger **False Positives**. For example, if a psychology student abstractly discusses "how to treat patients with AI dependency," the system might misclassify this academic intent as a personal dependency signal and incorrectly apply runtime constraints. 
* SIIHA Safety Guardrail is not an open-source project. If you're interested in this area, please contact the author via LinkedIn.