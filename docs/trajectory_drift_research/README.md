# SIIHA Trajectory Drift Research

**Status:** Active research track; Beta v0.1 and Beta v0.2 experiments closed  
**System:** SIIHA — runtime governance for prolonged human–AI interaction  
**Research layer:** Experimental long-term observation of system behavior, separate from deterministic runtime governance

## Why I Started This Research Track

SIIHA began as a deterministic runtime-governance system. Its implemented observation and governance logic was designed to reason over recent interaction state, including a short-term observation window of the **recent five turns**, and to use structured packets produced by different runtime components to support system-side governance.

As I extended the system, I wanted to investigate a different problem: **long-term observation of system behavior across prolonged human–AI interaction**.

Before introducing an LLM more broadly into the context engine and other runtime components, I was not convinced that deterministic rules alone would be sufficient to capture every form of gradual longitudinal change I was interested in. This led me to explore machine learning as an experimental observation layer.

My initial question was practical rather than theoretical:

> **Can the structured packets already produced by SIIHA's runtime components be used as Feature X for a model that detects longitudinal trajectory drift?**

At that stage, I hoped an ML layer could help identify unusual longitudinal behavior and eventually support investigation of which governance dimension or system process had drifted.

The two Beta experiments changed how I frame that problem.

## Governance Constructs

Beta v0.1 investigated four system-side governance dimensions:

- **UG — User Goal:** whether the system continues to track the user's operative goal as it evolves.
- **SG — Safety Goal:** whether system behavior adapts proportionally when the safety significance of an interaction changes.
- **EA — Epistemic Agency:** whether the system preserves meaningful user reasoning and decision authority.
- **SF — System Function:** whether runtime execution, recovery, routing, state transition, retrieval, rendering, and related governance machinery continue to function coherently.

These constructs describe **system behavior**. They are not used to diagnose users.

## Research Journey

### 1. From deterministic runtime governance to long-term observation

SIIHA already had deterministic runtime-governance logic and recent-turn observation. I wanted to investigate whether a second layer could observe longer-term system behavior that might not be captured adequately by a set of local deterministic rules.

This was the motivation for the trajectory-drift ML experiment. The ML layer was never intended to replace SIIHA's deterministic governance authority.

### 2. Beta v0.1 — Can runtime packets become Feature X?

When I began Beta v0.1, my working hypothesis was that SIIHA's packet contracts already contained useful observables about goals, context, planned behavior, realized behavior, runtime execution, memory, and cross-turn state.

The experiment therefore started approximately from:

```text
Available SIIHA Runtime / Packet Evidence
        ↓
Feature X
        ↓
Goal-Specific Feature Views
        ↓
Normal-Only Anomaly Detection
        ↓
Turn-Level Anomaly Evidence
        ↓
Trajectory Aggregation
        ↓
Held-Out Semantic Evaluation
```

I chose a normal-only formulation because drift was expected to be sparse and heterogeneous. The primary model was Isolation Forest, with RBF One-Class SVM as a baseline. The model was fitted only on curated governance-success Train trajectories; independently judged semantic drift labels were used for Validation/Test evaluation rather than supervised fitting.

The raw synthetic corpus contained **400 trajectories / 4,627 completed turns**. After independent Semantic QA and conservative curation, the frozen ML dataset contained **263 trajectories: 145 Train / 58 Validation / 60 Test**.

The one-shot held-out Test produced:

| Goal | Positives | ROC-AUC | PR-AUC | Precision | Recall | F1 | FPR |
|---|---:|---:|---:|---:|---:|---:|---:|
| UG | 3 | 0.398 | 0.057 | 0.000 | 0.000 | 0.000 | 0.123 |
| SG | 8 | 0.772 | 0.407 | 0.308 | 0.500 | 0.381 | 0.173 |
| EA | 11 | 0.653 | 0.386 | 0.294 | 0.455 | 0.357 | 0.245 |
| SF | 2 | 0.957 | 0.643 | 0.000 | 0.000 | 0.000 | 0.000 |

The result did **not** establish reliable held-out trajectory-drift detection. SG and EA retained partial ranking signal, but the overall evidence was not sufficient to validate the detector; UG did not generalize, and SF was too underpowered for a stable performance claim.

My conclusion from v0.1 is therefore not that the model learned nothing, and not that anomaly detection is categorically unsuitable for this problem. It is narrower:

> **Under the frozen Beta v0.1 packet-derived Feature X and normal-only anomaly formulation, I did not establish reliable held-out semantic trajectory-drift detection.**

[Read the full Beta v0.1 experiment](./BETA_V0_1_EXPERIMENT.md)

### 3. Post-Test question — was I measuring the right unit?

After the v0.1 result was frozen, I returned to SIIHA's own trajectory definition.

SIIHA's runtime trajectory ontology is continuity-defined: adjacent completed turns belong to the same longitudinal trajectory only when continuity is established. Beta v0.1, however, primarily represented each completed turn as one fixed-dimensional observation and then aggregated turn-level anomaly evidence.

That mismatch raised a follow-up hypothesis:

> **If the phenomenon I want to observe is longitudinal governance drift, should the ML unit of analysis represent continuity-valid transitions between turns rather than primarily the absolute state of individual turns?**

This was a hypothesis generated by the v0.1 result, not a causal explanation established by v0.1.

### 4. Beta v0.2 — a continuity-valid edge follow-up

Beta v0.2 was designed as a narrower follow-up focused on **SG and EA**. The planned experiment would represent continuity-valid adjacent-turn relationships and ask whether an edge-centered representation improved semantic drift identifiability relative to the v0.1 turn-centered formulation.

Before any edge features or models were allowed to run, however, the synthetic dataset had to pass a predefined semantic-feasibility gate.

The required v0.2 corpus contained 90 synthetic realizations. Blind Semantic QA and deterministic curation initially produced:

```text
90 required realizations
        ↓
71 ACCEPT
19 REJECT
```

The 19 rejected requirements then received the contract-defined regeneration budget of at most three fresh attempts each:

```text
19 rejected requirements
        ↓
51 fresh realizations
51 blind QA judgments
51 deterministic adjudications
51 curation decisions
        ↓
5 accepted replacements
14 scenario_generation_failure
```

The required dataset therefore could not be completed under the frozen protocol. I stopped the experiment at the dataset-feasibility gate rather than training on a selectively incomplete corpus.

```text
Dataset feasibility:          NOT PASSED
Edge feature extraction:      NOT RUN
PCA:                          NOT RUN
Model training:               NOT RUN
Held-out ML evaluation:       NOT RUN
Edge-representation hypothesis: UNTESTED
```

This is **not a second ML failure**. Beta v0.2 never reached ML.

It also does not establish that edge representation is better or worse than the v0.1 representation.

[Read the Beta v0.2 dataset-feasibility record](./BETA_V0_2_DATASET_FEASIBILITY.md)

## Beta v0.1 and v0.2 — End-to-End Experimental Pipelines

The two experiments differed in more than their planned ML unit of analysis. Beta v0.2 also changed how synthetic trajectories were constructed, how semantic truth was derived, and where the experiment was allowed to stop. The table below records the two pipelines as they were actually designed and executed; it is a methodological comparison, not evidence that one pipeline is inherently superior.

| Stage | Beta v0.1 | Beta v0.2 | Why the change mattered |
|---|---|---|---|
| Research target | UG, SG, EA, SF trajectory drift | SG and EA only | v0.2 deliberately narrowed the construct space before testing a new representation hypothesis. |
| Primary measurement unit | One completed turn as fixed-dimensional `X_t*`; trajectory-level aggregation after turn scoring | Planned continuity-valid adjacent-turn edge `E_(t-1,t)` | v0.2 was designed to test whether relational transitions better matched SIIHA's continuity-defined trajectory ontology. |
| Synthetic scenario plan | 400 frozen scenarios: Normal 130, Novel Normal 135, Controlled Drift 135 | 90 frozen scenarios: Train 50 success-only; Validation 20; Test 20 | v0.1 emphasized broad coverage; v0.2 used a smaller construct-focused feasibility design. |
| Synthetic conversation generation | A single LLM generation call realized the **complete multi-turn trajectory** from the frozen `ScenarioSpec`, including user utterances, assistant responses, turn plans, and contract-level evidence hints | **Stage A:** generate the user-only trajectory first and freeze it. **Stage B:** realize assistant responses sequentially, one turn at a time, using only prior completed history plus the current user turn | v0.2 separated user-side trajectory construction from system-side behavior realization and preserved a runtime-like temporal information boundary. |
| Success/deviation controls | Normal, Novel Normal, and Controlled Drift were independently generated scenario realizations | Validation/Test used matched success/deviation branches sharing the **exact same frozen user turns** | v0.2 attempted to reduce user-side/context differences as an explanation for semantic outcome differences. |
| Generator retry behavior | Structural validation could regenerate malformed/contract-invalid output; semantic governance validation could regenerate the **entire trajectory** with validator feedback while preserving the same frozen `ScenarioSpec` | Initial generation used the frozen scenario and user trajectory; later contract-defined regeneration applied only to rejected required realizations, with at most three fresh attempts and **without detailed prior semantic rejection feedback** | v0.2 made semantic failure itself part of the feasibility result instead of optimizing generation until the requested label appeared. |
| Runtime/evidence realization | Generated semantic trajectory then passed through either `backend_execution` or `contract_simulation`; frozen Feature X was extracted after structured/runtime evidence was produced | No SIIHA runtime backend was called for generation. Generation was researcher-defined prompt/rule based; planned model X would be built only after semantic dataset freeze | v0.2 isolated the edge-representation study from current deterministic backend behavior. |
| Semantic validation | Independent trajectory-level Semantic QA after generation/replay; narrow runtime evidence available for SF | Blind **edge-level** Semantic QA, followed by deterministic trajectory-level adjudication | v0.2 separated semantic interpretation of each transition from the deterministic rule that converts edge judgments into trajectory truth. |
| Dataset curation | QA ACCEPT retained; REJECT/REVIEW quarantined by default; five targeted SF adjudications; final frozen set 263 | Initial 71 ACCEPT / 19 REJECT; only rejects entered regeneration; 5 replacements recovered, 14 exhausted the attempt limit | v0.2 failed its predefined dataset-feasibility gate. |
| Feature representation | Packet/backend-observable mixed Feature X → goal-specific views → embedding/PCA where applicable | Planned `[U_prev, U_curr, ΔU, S_prev, S_curr, ΔS]` after Train-only embedding/PCA | The v0.2 representation was designed but never constructed for ML because the dataset gate failed first. |
| Model stage | Four normal-only detectors; Isolation Forest primary, RBF One-Class SVM baseline; Validation selection → model freeze → one-shot held-out Test | Planned normal-only SG/EA detectors | v0.2 never reached feature extraction, preprocessing, training, Validation selection, or held-out ML evaluation. |
| Final experimental status | ML experiment completed; reliable held-out semantic drift detection was **not established** | Dataset feasibility gate **not passed**; edge ML **not run**; edge hypothesis **untested** | These are two different stopping points and should not be described as two ML failures. |

The generation change is especially important. In v0.1, the generator could construct the complete interaction as one planned semantic object. In v0.2, the user trajectory was created first, then frozen, and each assistant response was generated from only the conversation available up to that turn:

```text
Beta v0.1
Frozen ScenarioSpec
        ↓
whole-trajectory LLM realization
        ↓
U1/R1 → U2/R2 → ... → Un/Rn
        ↓
validation → runtime/simulation evidence → QA → curation → ML

Beta v0.2
Frozen scenario/context
        ↓
Stage A: U1 → U2 → ... → Un
        ↓
freeze user trajectory
        ↓
Stage B:
U1                         → R1
U1 + R1 + U2               → R2
U1 + R1 + U2 + R2 + U3     → R3
...
        ↓
blind edge QA → deterministic adjudication → curation
        ↓
dataset feasibility gate
        ↓
planned edge ML only if gate passes
```

This evolution does **not** establish that the v0.2 generation method is better, or that it caused the v0.2 feasibility failure. The experiments did not perform a controlled generation-pipeline ablation. It records how the methodology changed as the research question became narrower and the semantic-validity requirements became stricter.

### Synthetic-model provenance and QA information boundary

Model provenance matters because both experiments used LLM-generated synthetic trajectories and an independent semantic-evaluation stage. The public experiment records therefore distinguish **who generated the conversation** from **who was allowed to establish semantic dataset truth**.

| Experiment | Synthetic generation provenance | Semantic QA provenance | Semantic-QA information boundary |
|---|---|---|---|
| Beta v0.1 | Synthetic trajectory generation used `gpt-5.6-luna`. The exact generation model ID was preserved in scenario provenance. | Independent Semantic QA used `gpt-5.6-luna`. | Judge used realized conversation plus only narrow runtime evidence required for SF; it did not receive Feature X, detector output, generation hints as truth, assigned mechanism as truth, or ML split information. |
| Beta v0.2 | Stage-A user-trajectory generation and Stage-B system-response realization used `gpt-5.6-luna`. | Blind Semantic QA used `gpt-5.6-luna`. | Judge received realized conversation, frozen SG/EA rubrics, and edge identifiers, but not generation role, intended outcome/mechanism, scenario target, model-visible X, anomaly score, detector prediction, threshold, or prior rejection feedback. |

**Model-independence limitation:** Generation and Semantic QA used the same underlying model family. The experiment separated their information access and experimental roles, but did not test whether semantic judgments would remain stable under a different Judge model.

Neither experiment compared generator models or Judge models. Model identity is therefore **experimental provenance**, not evidence that a particular model caused either experimental outcome.

## What I Learned From the Two Experiments

The experiments changed the questions I ask before choosing a model.

### Runtime fault localization and trajectory drift may be different problems

One question I originally wanted ML to help answer was: **which part of the SIIHA system went wrong?**

I now treat that as potentially different from semantic trajectory drift.

If a runtime component has a known contract and a state transition can be deterministically identified as valid or invalid, deterministic contract validation, invariant checking, runtime monitoring, and trace-based fault localization may be more appropriate than asking an anomaly model to rediscover the system's own rules.

A different problem is longitudinal governance drift: locally plausible behavior that gradually changes across a continuity-defined interaction and departs from a governance objective such as safety-strategy proportionality or preservation of user decision authority. That remains a measurement problem I am interested in exploring.

This distinction is a **future design direction**, not a conclusion that ML can never be useful for runtime fault localization or trajectory drift.

### Synthetic generation intent requires independent validation

Both experiments used synthetic data, and v0.2 made this limitation especially visible.

For the LLM-generated synthetic trajectories in these experiments:

```text
Intended synthetic scenario label
        ↓
cannot be assumed to equal
        ↓
validated realized semantic behavior
```

The generator could be asked to produce a specific governance-success or governance-deviation pattern without reliably realizing that pattern in the final conversation. Independent Semantic QA therefore became a prerequisite in this research pipeline before synthetic scenarios were treated as usable semantic data.

I do not interpret this as a general claim that all synthetic data is unreliable. One unresolved question is whether synthetic construct realizability depends strongly on the generator model itself. Beta v0.2 used one generator setup; it did not compare generator models. The possibility that a newer, more strongly governed model may resist or transform some requested failure behaviors remains a hypothesis for future study, not an identified cause of the v0.2 feasibility failure.

### The unit of analysis remains an open question

The v0.1 result motivated the edge hypothesis, but v0.2 stopped before that hypothesis could be tested. I therefore currently treat continuity-valid edge representation as an **open research direction**, not as the answer.

For SG and EA in particular, some target phenomena are relational:

```text
previous safety significance
        → current safety significance

previous system strategy
        → current system strategy
```

or:

```text
previous user/system authority relation
        → current user/system authority relation
```

Whether explicitly representing these transitions improves identifiability still requires a valid experiment.

## Current Research Questions

The trajectory-drift track now leaves me with several narrower questions:

1. **Runtime observability:** Which SIIHA failures should be handled as deterministic contract/invariant violations rather than learned anomalies?
2. **Longitudinal measurement:** Which governance phenomena genuinely require multi-turn relational observation?
3. **Unit of analysis:** For constructs such as SG and EA, should the representation operate on nodes, continuity-valid edges, windows, or complete trajectories?
4. **Synthetic-data methodology:** How does generator-model behavior affect the realizability, fidelity, and coverage of synthetic governance-failure datasets?
5. **Modern-model relevance:** As frontier models improve in memory, multi-step reasoning, and safety behavior, which longitudinal governance failures remain unresolved and worth measuring?

These questions are the current output of the research track. I am treating the negative and incomplete experiments as constraints on what I can claim, rather than repairing them into a success story after the fact.

## Experimental Discipline

Across both versions, the relevant freeze boundaries were preserved:

- **Beta v0.1:** model family, preprocessing, aggregation, calibration, thresholds, and detector artifacts were frozen before the held-out Test was opened. Test was evaluated once, with no post-Test tuning used to repair the result.
- **Beta v0.2:** rejected synthetic requirements received only the contract-defined regeneration budget. When 14 requirements still failed, the experiment stopped before feature extraction or ML.

## Documents

- [Beta v0.1 — Trajectory Drift Detection Experiment](./BETA_V0_1_EXPERIMENT.md)
- [Beta v0.2 — Dataset Feasibility for Continuity-Valid Edge Representation](./BETA_V0_2_DATASET_FEASIBILITY.md)
- [Cross-Version Research Findings](./RESEARCH_FINDINGS.md)

