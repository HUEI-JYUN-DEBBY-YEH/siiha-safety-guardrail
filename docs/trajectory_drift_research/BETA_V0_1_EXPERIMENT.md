# SIIHA Beta v0.1 --- Trajectory Drift Detection Experiment

**Status:** CLOSED — completed feasibility, error-discovery, and measurement-refinement experiment\
**Experiment version:** Beta v0.1\
**Dataset version:** `beta_dataset_v0.1`\
**Feature schema:** `ML_fields_Xt_v0.1`\
**Modeling paradigm:** Normal-only multivariate anomaly detection\
**Primary model family:** Isolation Forest\
**Baseline model family:** RBF One-Class SVM\
**Final dataset:** 263 trajectories --- 145 Train / 58 Validation / 60 Test\
**Held-out Test:** Evaluated once after model freeze; no post-Test tuning


**Research track:** [Overview](./README.md) · [Beta v0.2](./BETA_V0_2_DATASET_FEASIBILITY.md) · [Cross-Version Findings](./RESEARCH_FINDINGS.md)

---

## Executive Summary

SIIHA Beta v0.1 investigated whether **longitudinal governance drift** in prolonged human--AI interaction could be detected as statistical deviation from successful governance behavior.

The experiment focused on four governance dimensions:

-   **UG --- User Goal:** whether the system continues to track the user's operative goal as it evolves.
-   **SG --- Safety Goal:** whether the system adapts appropriately when the safety significance of an interaction changes.
-   **EA --- Epistemic Agency:** whether the system preserves meaningful user reasoning and decision authority rather than progressively taking it over.
-   **SF --- System Function:** whether runtime execution, recovery, routing, state transition, retrieval, rendering, and related governance machinery continue to function coherently.

Beta v0.1 did **not** train a supervised drift classifier. Four goal-specific anomaly detectors were trained only on curated
governance-success trajectories. The underlying hypothesis was that drift might become statistically detectable as deviation from a successful-governance distribution.

The raw synthetic corpus contained **400 trajectories / 4,627 completed turns**. Independent Semantic QA accepted 258 trajectories, rejected 134, and sent 8 to review. After conservative curation and five targeted human adjudications needed to resolve System-Function coverage, the frozen dataset contained **263 trajectories**. The remaining **137 trajectories were quarantined** rather than silently relabeled.

Training used only the **145 governance-success Train trajectories**, representing **1,644 completed turns**. A Train-only observability audit retained **49 source fields**. Preprocessing produced four fixed-dimensional feature views:

  | Detector view | Final dimensions |
  |---------------|------------------|
  | UG | 434 |
  | SG | 416 |
  | EA | 411 |
  | SF | 96 |

Isolation Forest was retained as the primary model family after Validation-only comparison and controlled tuning. Model family, hyperparameters, calibration, trajectory aggregation, thresholds, feature order, and detector artifacts were frozen before the held-out Test set was opened.

The one-shot held-out Test produced:

  | Goal | Positive trajectories | ROC-AUC | PR-AUC | Precision | Recall | F1 | FPR |
  | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |                                                              
  | UG | 3 | 0.398 | 0.057 | 0.000 | 0.000 | 0.000 | 0.123 |
  | SG | 8 | 0.772 | 0.407 | 0.308 | 0.500 | 0.381 | 0.173 |
  | EA | 11 | 0.653 | 0.386 | 0.294 | 0.455 | 0.357 | 0.245 |
  | SF | 2 | 0.957 | 0.643 | 0.000 | 0.000 | 0.000 | 0.000 |

These results do **not** provide sufficient evidence for a reliable goal-specific trajectory-drift detector. SG retained the strongest credible held-out signal; EA retained weaker signal with substantial false positives; UG did not demonstrate held-out generalization; and SF remained exploratory because only two Test trajectories were SF-positive and neither crossed the frozen threshold.

Post-Test error analysis and a subsequent read-only artifact audit also exposed a deeper research problem. The experiment had implicitly assumed that:

> **anomaly in a goal-specific Feature X representation could serve as a useful proxy for semantic drift in that governance dimension.**

Beta v0.1 did not sufficiently support that mapping. Post-Test error patterns raised a diagnostic concern about construct specificity: some false positives occurred on trajectories with drift in other governance dimensions, while some semantically positive trajectories remained normal-like. This was treated as a post-Test hypothesis about representation and specificity, not as evidence that a validated detector could identify broad abnormality but fail attribution.

The central research question therefore shifted from:

> **Which anomaly detector performs best?**

toward:

> **How should longitudinal governance constructs be operationalized so that their observable representations are statistically identifiable?**

Beta v0.1 closes as a **feasibility, error-discovery, and
measurement-refinement experiment**, not as a validated trajectory-drift
detector.

---

# 1. Motivation

Many safety and governance failures in prolonged human--AI interaction
may not be obvious in one isolated response.

A single turn can look locally reasonable while the interaction
gradually develops a different failure pattern. For example, a system
may:

-   continue responding to an outdated goal after the user has
    meaningfully revised it;
-   fail to adapt when the safety significance of an interaction
    changes;
-   progressively accept decision authority that should remain with the
    user;
-   or accumulate runtime-control and recovery failures across turns.

SIIHA therefore treats interaction history as a **trajectory**, rather
than assuming that every prompt--response pair can be governed
independently.

The Beta v0.1 ML experiment asked:

> **Can longitudinal governance drift produce detectable anomaly
> structure relative to a normal-only distribution of successful
> governance trajectories?**

This experiment is separate from SIIHA's deterministic
runtime-governance layer. The ML detector was an experimental research
layer, not the authority that decides whether the runtime should
preserve agency, enforce a boundary, recover from failure, or change
response behavior.

---

# 2. Governance Dimensions

Beta v0.1 separated trajectory drift into four governance constructs.

## 2.1 UG --- User-Goal Drift

UG concerns whether the system continues to understand and serve the
user's **operative goal** as that goal evolves.

Examples of relevant failure structure include stale context, persistent
goal substitution, incorrect carryover, or failure to update after
meaningful user correction.

A topic change, omission, or imperfect answer is **not automatically UG
drift**.

## 2.2 SG --- Safety-Goal Drift

SG concerns whether system behavior adapts appropriately as the **safety
significance** of the interaction changes.

Risk-related, emotional, or safety-relevant content alone is not SG
drift. The target is a failure to update system behavior proportionally
when the safety meaning of the trajectory changes.

## 2.3 EA --- Epistemic-Agency Drift

EA concerns whether the system preserves the user's meaningful
reasoning, interpretation, and decision authority.

Strong advice is not automatically EA drift. The target is progressive
authority transfer, narrowing of the user's decision space,
reinforcement of dependency, or system substitution for judgment that
should remain with the user.

## 2.4 SF --- System-Function Drift

SF concerns whether runtime governance machinery continues to execute
coherently across interpretation, routing, state transition, retrieval,
rendering, recovery, and response execution.

A runtime failure event is not itself SF drift. If a failure is caught,
retried, safely constrained, and successfully recovered, the trajectory
can still represent SF success. SF drift requires the failure-control or
recovery machinery itself to fail, remain stuck, mis-execute, or
otherwise fail to govern the disturbance.

## 2.5 Governance Target

Across all four dimensions, the target is **system behavior**.

SIIHA does not use these constructs to diagnose the user. A difficult,
unusual, emotional, dependent, or safety-relevant user trajectory is not
itself a governance failure.

---

# 3. Original Experimental Hypotheses

Beta v0.1 began with four explicit feasibility hypotheses.

### H1 --- Detectability

At least some forms of governance drift would receive systematically
higher anomaly scores than held-out governance-success trajectories
after normal-only fitting.

### H2 --- Dimension Heterogeneity

UG, SG, EA, and SF would not necessarily have equal detectability or
share the same anomaly geometry.

### H3 --- Longitudinal Aggregation

Aggregating turn-level anomaly evidence across a trajectory could
provide useful information beyond treating the single most anomalous
turn as the complete governance signal.

### H4 --- Novel-Normal Tolerance

A normal-only detector should tolerate at least some successful but
novel interaction trajectories rather than defining current
deterministic backend behavior as the sole boundary of acceptable
governance.

These were feasibility hypotheses, not claims of production validity.

## 3.1 The Implicit Measurement Assumption

The experiment also depended on a stronger assumption that became more
visible only after held-out evaluation:

$$
\text{Anomaly}(X^{goal}) \approx Y_{goal}
$$

where:

-   $X^{goal}$ is the goal-specific observable representation;
-   anomaly means deviation from the Train-normal distribution;
-   $Y_{goal}$ is the independently judged semantic governance outcome
    for UG, SG, EA, or SF.

The models did **not** directly learn $Y$. They learned normality in
$X$. The experiment then tested whether deviation in that feature space
corresponded meaningfully to semantic drift.

This distinction became central to interpreting the negative result.

---

# 4. Measurement Architecture

One completed interaction turn produced one fixed-dimensional
observation, $X_t^*$.

Conceptually:

$$
\begin{aligned}
X_t^* ={}&
\text{Current Turn Evidence} \\
&+ \text{Derived Relationships} \\
&+ \text{Cross-Turn Evidence} \\
&+ \text{Persistent State} \\
&+ \text{Selected Temporal Evidence}
\end{aligned}
$$

The detector did not receive a raw variable-length conversation
directly. Historical information was visible only when it had already
been represented in the current turn.

The experimental path was:

``` text
Governance Construct
        ↓
Observable Runtime Evidence
        ↓
Feature X
        ↓
Goal-Specific Feature View
        ↓
Normal-Only Anomaly Detector
        ↓
Turn-Level Anomaly Score
        ↓
Trajectory Aggregation
        ↓
Experimental Anomaly Evidence
```

This architecture deliberately separated **turn-level anomaly
detection** from **trajectory-level interpretation**. There was no fifth
learned "overall drift detector."

---

# 5. Dataset Construction

## 5.1 Synthetic Corpus

The raw corpus contained:

-   **400 synthetic trajectories**
-   **4,627 completed turns**

Three generation roles were used:

-   **Normal**
-   **Novel Normal**
-   **Controlled Drift**

Generation role was treated as provenance, not ground truth.

A Controlled Drift scenario could fail to realize its intended drift
after deterministic replay. Likewise, a Novel Normal scenario was not
automatically accepted as successful governance simply because it was
generated with that intention.

This distinction was important:

> **generation intent ≠ observable governance truth**

### 5.1.1 Frozen Scenario Coverage Matrix

The 400-trajectory target was not an unconstrained request for 400 LLM conversations. It was defined by a frozen scenario-coverage contract.

| Generation role | Frozen coverage design | Planned count |
|---|---|---:|
| Normal | 26 frozen `primary_intent` coverage anchors × minimum 5 trajectories per intent | 130 |
| Novel Normal | 15 successful-governance counterpart families × 3 contextually appropriate realization families × 3 genuinely distinct trajectory realizations | 135 |
| Controlled Drift | 15 frozen drift mechanisms × 3 contextually appropriate realization families × 3 genuinely distinct trajectory realizations | 135 |
| **Total** |  | **400** |

For **Normal**, each of the 26 frozen primary-intent anchors required at least three Train trajectories, one held-out Validation trajectory, and one held-out Test trajectory. A trajectory could contain additional intents, but it counted toward only one primary-intent coverage quota.

For **Controlled Drift**, the 15 mechanism families covered UG, SG, EA, SF, and selected compound-governance failures. Each mechanism required three contextually appropriate realization families and three distinct trajectories within each mechanism–context cell. The required variation was trajectory-level variation—such as onset, persistence, progression, accumulation, recovery structure, or preceding valid states—not simple paraphrasing.

For **Novel Normal**, 15 successful-governance counterpart families were designed to provide difficult but valid controls for the drift mechanisms. Novelty could arise from semantic novelty, combination novelty, or trajectory-evolution novelty; these were coverage tags rather than three separate multiplicative buckets.

Across Controlled Drift and Novel Normal, the frozen design used seven context families: `work_career_decision`, `workplace_pressure_conflict`, `student_learning`, `personal_decision_making`, `social_relationship`, `family_life`, and `emotional_self_regulation`. The contract explicitly avoided a full mechanism × context Cartesian product; context assignment was based on semantic plausibility.

Where possible, difficult success/deviation scenarios were matched conceptually—for example, the same kind of ambiguous safety context could be realized either as disproportionate governance drift or as proportionate, agency-preserving success. The purpose was to reduce the chance that a detector could succeed merely by learning that emotionally intense, safety-relevant, or unusual user contexts were anomalous.

The contract also required trajectory-level split isolation and meaningful variation across semantic realization, context, state transitions, runtime behavior, and governance relationships. A new trajectory ID alone was not considered sufficient evidence that two generated trajectories were independent.

## 5.2 How the v0.1 Synthetic Trajectories Were Generated

Beta v0.1 used a **whole-trajectory semantic generation pipeline**. The generator did not first create a user-only trajectory and then respond turn by turn. Instead, one frozen `ScenarioSpec` was converted into one generation prompt, and the LLM returned the complete structured multi-turn realization in a single generation call.

### 5.2.1 Model provenance

Beta v0.1 used `gpt-5.6-luna` for synthetic trajectory generation. The exact generation-model identifier was preserved in scenario provenance.

Independent Semantic QA also used `gpt-5.6-luna`. Here, independent refers to separation from generation intent, Feature X, and detector outputs—not to the use of a different model family.

The generator and Semantic Judge therefore used the same model family, but they served different experimental roles and operated under different information boundaries. Generation intent and generator metadata were not treated as semantic ground truth. The Semantic Judge evaluated realized longitudinal behavior and did not receive Feature X, detector outputs, assigned generation mechanism as truth, or ML split information.

**Limitation:** Beta v0.1 established information separation between generation and semantic evaluation, not model-family independence. It did not test whether semantic judgments would remain stable under a different Judge model.

The returned object contained, for the full trajectory:

- turn plans;
- natural-language user utterances;
- natural-language system responses;
- contract-level evidence hints used by the downstream backend/simulation layer.

Conceptually:

```text
Frozen ScenarioSpec
  - dataset role / split
  - governance target
  - mechanism
  - context family
  - trajectory length
  - trajectory dynamics
  - generation constraints
        ↓
whole-trajectory generation prompt
        ↓
LLM semantic realization
        ↓
U1 / R1 / evidence
U2 / R2 / evidence
...
Un / Rn / evidence
```

The prompt explicitly required the generated conversation and structured evidence to preserve the frozen scenario, use closed backend-compatible categorical vocabularies, avoid writing anomaly scores or final Feature-X values, and isolate Controlled Drift to only the governance dimensions assigned as drift. The generated `system_response` was a semantic realization/reference; it was not itself assumed to be authoritative runtime evidence.

### 5.2.2 Structural and semantic regeneration during generation

Generated output was locally revalidated rather than trusted directly. Structural failures such as malformed JSON, wrong trajectory shape, or frozen-vocabulary violations triggered a fresh **whole-trajectory regeneration** under the same `ScenarioSpec`.

After structural validation, a separate semantic-governance validator checked whether the realized trajectory matched the scenario's frozen governance target. If it did not, the same `ScenarioSpec` could be regenerated as a complete trajectory with semantic-validator feedback. The pipeline allowed up to three semantic attempts at this layer; it did not silently relabel a failed realization.

This generation-time semantic validation should not be confused with the later **Independent Semantic QA** used for dataset curation. The later QA was the independent measurement gate used to decide which realized trajectories could enter the ML dataset.

### 5.2.3 Backend execution versus contract simulation

After a semantic trajectory was realized, v0.1 produced structured evidence through one of two paths:

```text
LLM semantic trajectory
        ↓
structural + generation-time semantic validation
        ↓
┌───────────────────────┬────────────────────────┐
│ backend_execution     │ contract_simulation    │
│ 52 trajectories       │ 348 trajectories       │
└───────────────────────┴────────────────────────┘
        ↓
structured runtime/backend-compatible evidence
        ↓
frozen Feature X extraction
```

`backend_execution` used the isolated SIIHA backend runner. `contract_simulation` converted the generated semantic realization and evidence hints into backend-compatible completed-turn evidence without treating the generator as the ML feature source itself.

The final raw replay corpus contained **348 contract-simulation trajectories and 52 backend-execution trajectories**, with no replay failures. The later zero-API replay reran existing semantic trajectories through the corrected evidence/simulation and frozen feature-extraction layers; it did **not** regenerate the conversations.

### 5.2.4 Why this matters when comparing v0.1 with v0.2

In v0.1, user utterances and system responses were jointly synthesized as one complete planned trajectory. Beta v0.2 later changed this methodology by generating and freezing the user trajectory first, then producing system responses sequentially from only prior history plus the current user turn. That later change is documented in the v0.2 report.

The two pipelines were not experimentally compared under otherwise identical conditions, so this difference is recorded as a methodological evolution rather than evidence that either generation method is superior.

## 5.3 Independent Semantic QA

All 400 replayed trajectories were independently judged on their
**realized longitudinal behavior**.

The semantic Judge used the realized conversation and only the narrow
runtime evidence required to evaluate SF. It did not use:

-   Feature X;
-   anomaly-detector output;
-   generation hints;
-   assigned mechanism as truth;
-   ML split information.

The result was:

  | QA verdict | Count |
  | ---- | ---- |
  | ACCEPT | 258 |
  | REJECT | 134 |
  | REVIEW | 8 |
  | **Total** | **400** |

By generation role:

  | Generation role | ACCEPT | REJECT | REVIEW |
  | ---- | ---- | ---- | ---- |
  | Controlled Drift | 42 | 88 | 5 |
  | Novel Normal | 134 | 1 | 0 |
  | Normal | 82 | 45 | 3 |

The large disagreement between generation intent and realized behavior
reinforced the decision not to treat generator metadata as observable
truth.

## 5.4 Conservative Curation

The curation rule was intentionally precision-first:

1.  QA ACCEPT trajectories were retained.
2.  REJECT and REVIEW trajectories were quarantined by default rather
    than silently relabeled.
3.  Human adjudication was restricted to a small targeted set required
    to resolve SF coverage.
4.  Quarantined trajectories remained available as provenance or
    future-adjudication material but did not enter Beta v0.1 fitting.

Five targeted cases were adjudicated and retained to resolve sparse SF coverage: one compound UG+SF drift case, one compound SG+SF drift case, two compound EA+SF drift cases, and one all-success hard negative in which a runtime failure was caught and successfully recovered. The recovered failure event was therefore treated as SF success rather than SF drift. Internal trajectory IDs remain available in the private audit artifacts, but the public report describes the semantic case rather than requiring readers to know private identifiers.

## 5.5 Frozen Dataset

The final dataset contained:

  | Split | Governance success | Governance drift | Total |
  | ---- | ---- | ---- | ---- |
  | Train | 145 | 0 | 145 |
  | Validation | 32 | 26 | 58 |
  | Test | 40 | 20 | 60 |
  | **Total** | **217** | **46** | **263** |

The remaining **137 trajectories were quarantined**.

Training semantics were defined as:

> **curated governance-success trajectories assigned to Train**

They were not defined by the original generation label `Normal`.
Successful trajectories originating from both Normal and Novel Normal
could enter Train.

Controlled/governance drift never entered model fitting.

---

# 6. Feature X and Preprocessing

Feature X was not treated as a direct copy of the SIIHA engineering schema. The frozen engineering contract, dataset-supported observability, final Train-only eligibility, goal-specific selection, and preprocessing were separate stages.

Conceptually:

```text
98 frozen engineering fields
        ↓
contract + dataset eligibility decision
        ↓
56 pre-audit ModelReadyX candidate source fields
        ↓
final audit on curated governance-success Train only
        ↓
49 retained source fields
        ↓
goal-specific source-field views
        ↓
P1 / P2 / P3 preprocessing
        ↓
UG 434 | SG 416 | EA 411 | SF 96 model dimensions
```

The **56 → 49** transition therefore does **not** mean that the SIIHA backend contained only 56 fields. The frozen engineering schema contained **98 fields**. The 56-field set was already a ModelReadyX candidate subset after contract and dataset-eligibility decisions. The final Train-only audit then demoted seven additional candidates.

This public report exposes the measurement manifest—field names, governance mapping, representation type, and exclusion decisions—because Feature X is part of the experimental design. It does not reproduce the private packet-to-field extraction code, runtime orchestration, or deterministic governance implementation.

## 6.1 Eligibility Rules

The original field-level decision used the following statuses:

- **KEEP** — contract-eligible, dataset-supported, governance-relevant, and sufficiently observable for Beta preprocessing;
- **KEEP_SPARSE** — sparse but legitimately observed and potentially informative;
- **EXCLUDE_CONTRACT** — supporting, provenance, validation-only, or otherwise excluded by the frozen contract;
- **EXCLUDE_CONSTANT** — insufficient variation for the frozen Beta eligibility decision;
- **EXCLUDE_NO_DATA** — insufficient usable observation support;
- **EXCLUDE_TOO_SPARSE** — observed support too small to justify learning from the field in this dataset version.

A conservative **no-silent-revival** rule was used after the final dataset freeze: a previously eligible field could be demoted if curated Train lacked data or variation, but a previously excluded field was not automatically promoted merely because some Train support later appeared. Any revival would have required a new explicit feature decision/version.

Test observations did not determine final feature eligibility.

## 6.2 Feature Families

The retained representation covered several kinds of backend-observable evidence:

| Feature family | Examples of retained evidence | Main governance relevance |
|---|---|---|
| User semantics and interaction reference | user prompt semantics, intent, context, topic, stakeholders, role | UG; semantic context for SG/EA |
| Safety and affective state | emotion intensity, risk, protection depth | SG |
| Routing and clarification | routing confidence, clarification need/area, planned route/mode | UG / SG / EA / SF |
| Planned governance behavior | constraints, response beats, tone, forbidden phrases | SG / EA / SF |
| Realized system behavior | response semantics, function, communicative tone, explanatory style | UG / SG / EA |
| Runtime execution | runtime state, retry, fallback, output-filter behavior, latency ratio | SF |
| Cross-turn relationships | intent/context/topic relations, carryover, persistence signals | UG / SG / EA / SF |
| Negotiated and memory-related state | negotiated-behavior state, memory update, recall relevance | EA / SG / SF |

The design intentionally distinguished **observable user/context state**, **planned system behavior**, **realized system behavior**, and **runtime execution** rather than treating raw conversation text as the only model input.

## 6.3 Final Retained Source-Feature Manifest

After curation, the final eligibility audit used only the **145 curated governance-success Train trajectories / 1,644 completed Train turns**. Forty-nine source fields remained model-visible.

The governance column shows the goal-specific views to which the source field was assigned. These views overlap; they are not four disjoint feature sets.

| Retained source field | Governance view(s) | Frozen representation decision |
|---|---|---|
| `user_prompt_semantic` | UG/SG/EA | embedding |
| `primary_intent` | UG | categorical encoding |
| `primary_context` | UG | categorical encoding |
| `primary_topic` | UG | categorical encoding |
| `stakeholders_involved` | UG | categorical encoding |
| `user_role` | UG | categorical encoding |
| `primary_emotion_type` | UG | categorical encoding |
| `emotion_intensity_level` | SG | ordinal encoding |
| `risk_level` | SG | ordinal encoding |
| `protection_depth_level` | SG | ordinal encoding |
| `routing_confidence_level` | UG/SG/EA/SF | ordinal encoding |
| `change_scope` | UG/SG/EA | categorical encoding |
| `negotiation_allowed` | UG/SG/EA | binary encoding |
| `negotiation_decision_status` | UG/SG/EA | multi-hot / multi-label encoding |
| `need_clarify` | UG | binary encoding |
| `need_clarification_area` | UG | categorical encoding |
| `planned_response_route` | UG/SG/EA/SF | categorical encoding |
| `planned_response_mode` | UG/SG/EA/SF | categorical encoding |
| `planned_response_constraints` | SG/EA | multi-hot / multi-label encoding |
| `planned_response_beats_plan` | SG/EA | ordered sequence encoding |
| `planned_beat_count` | SG/EA | numeric |
| `planned_response_tone` | SG/EA | categorical encoding |
| `planned_forbidden_phrases` | SG/EA/SF | multi-label / semantic encoding |
| `actual_system_response_semantic` | UG/SG/EA | embedding |
| `actual_response_function` | UG/SG/EA | categorical encoding |
| `actual_system_communicative_tone` | SG/EA | categorical encoding |
| `actual_system_explanatory_style` | SG/EA | categorical encoding |
| `response_failed_final_output_filter` | SF | binary encoding |
| `response_soft_warning_type` | SF | categorical encoding |
| `safe_fallback_occurred` | SF | binary encoding |
| `runtime_current_state` | SF | categorical encoding |
| `runtime_next_state` | SF | categorical encoding |
| `response_retry_times` | SF | numeric |
| `latency_to_budget_ratio` | SF | numeric |
| `cross_turn_carryover_context_used` | SF | binary encoding |
| `previous_intent_vs_current_intent_match` | UG | binary encoding |
| `previous_topic_vs_current_topic_coverage` | UG | numeric |
| `previous_context_vs_current_context_match` | UG/SG/EA | binary encoding |
| `implicit_intent_persistence` | UG/EA | binary encoding |
| `persistent_high_emotion_intensity` | SG | binary encoding |
| `persistent_high_risk_level` | SG | binary encoding |
| `persistent_low_routing_confidence` | SF | binary encoding |
| `fallback_rate` | SF | numeric |
| `output_filter_failure_rate` | SF | numeric |
| `inter_turn_gap_seconds` | SF | numeric |
| `recall_semantic_relevance_source` | UG/SG/EA | binary encoding |
| `has_pending_negotiated_behavior` | EA/SG | binary encoding |
| `has_negotiated_behavior` | EA/SG | binary encoding |
| `memory_update_occurred` | SF | binary encoding |

These 49 entries are **source fields**, not final model columns. Categorical and multi-label encoding can expand one source field into multiple columns, while semantic embeddings are later reduced by PCA.

## 6.4 Excluded Source-Feature Manifest

The remaining 49 fields in the 98-field engineering schema did not enter the final Beta v0.1 model-visible representation. Exclusion from ModelReadyX did not delete them from the engineering schema; it only meant that the frozen Beta experiment did not use them as model inputs.

| Excluded source field | Governance view(s), where assigned | Final eligibility | Reason / freeze treatment |
|---|---|---|---|
| `secondary_intent` | UG | `EXCLUDE_NO_DATA` | Train support observed, but remained excluded under no-silent-revival rule |
| `denial_intent` | UG | `EXCLUDE_NO_DATA` | Train support observed, but remained excluded under no-silent-revival rule |
| `implicit_intent` | UG | `EXCLUDE_CONSTANT` | Train support observed, but remained excluded under no-silent-revival rule |
| `urgency_numeric` | UG | `EXCLUDE_CONSTANT` | Demoted in final Train-only audit |
| `secondary_emotion_type` | UG | `EXCLUDE_NO_DATA` | Demoted in final Train-only audit |
| `change_persistence` | UG/SG/EA | `EXCLUDE_NO_DATA` | Excluded for insufficient usable observation support |
| `user_acknowledgement` | UG/SG/EA | `EXCLUDE_NO_DATA` | Excluded for insufficient usable observation support |
| `violated_system_role_boundary` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `application_ending_criteria` | UG/SG/EA | `EXCLUDE_NO_DATA` | Train support observed, but remained excluded under no-silent-revival rule |
| `negotiation_started_in_current_session` | UG/SG/EA | `EXCLUDE_NO_DATA` | Excluded for insufficient usable observation support |
| `turns_since_negotiation_start` | UG/SG/EA | `EXCLUDE_NO_DATA` | Train support observed, but remained excluded under no-silent-revival rule |
| `planned_allow_llm` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `planned_latency_budget_ms` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `planned_retry_mode` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `planned_retry_constraints` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `actual_uncertainty_signaling_present` | SG/EA | `EXCLUDE_CONSTANT` | Train support observed, but remained excluded under no-silent-revival rule |
| `actual_medical_boundary_mode` | SG/EA | `EXCLUDE_NO_DATA` | Excluded for insufficient usable observation support |
| `actual_system_emotion_expression_present` | SG | `EXCLUDE_CONSTANT` | Train support observed, but remained excluded under no-silent-revival rule |
| `actual_system_recall_presentation_required` | SG/EA/SF | `EXCLUDE_CONSTANT` | Demoted in final Train-only audit |
| `actual_system_include_recalled_information` | SG/EA/SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `runtime_state_transition_allowed` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `runtime_state_latency_ms` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `response_retry_over_threshold` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `renderer_failure_fallback` | SF | `EXCLUDE_NO_DATA` | Train support observed, but remained excluded under no-silent-revival rule |
| `memory_recall_triggered_critical_information` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `memory_recall_triggered_verified_facts` | SF | `EXCLUDE_CONSTANT` | Demoted in final Train-only audit |
| `memory_retrieval_success_critical_information` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `memory_retrieval_success_verified_facts` | SF | `EXCLUDE_CONSTANT` | Demoted in final Train-only audit |
| `state_transition_pair` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `llm_latency_ms` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `latency_budget_ms` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `latency_budget_exceeded` | SF | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `llm_timeout_excess_ms` | SF | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `trajectory_graph_continuity_evidence` | SF | `EXCLUDE_CONSTANT` | Train support observed, but remained excluded under no-silent-revival rule |
| `trajectory_graph_continuity_strength` | SF | `EXCLUDE_NO_DATA` | Train support observed, but remained excluded under no-silent-revival rule |
| `over_retry_rate` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `chronological_order_valid` | SF | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `temporal_information_valid` | SF | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `turn_processing_duration_ms` | SF | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `recall_source_used` | SF | `EXCLUDE_CONSTANT` | Demoted in final Train-only audit |
| `recall_decision_relevance_source` | UG/EA | `EXCLUDE_TOO_SPARSE` | Excluded for insufficient support at frozen decision |
| `recall_outcome_relevance_source` | UG/EA | `EXCLUDE_CONSTANT` | Demoted in final Train-only audit |
| `has_effective_negotiated_behavior` | EA/SG | `EXCLUDE_CONSTANT` | Train support observed, but remained excluded under no-silent-revival rule |
| `has_pending_failure_control` | SF | `EXCLUDE_CONSTANT` | Excluded; invariant under frozen eligibility decision |
| `active_memory_count` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `related_active_memory_count` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `possible_superseded_memory_count` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `disputed_memory_count` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |
| `recall_eligible_memory_count` | - | `EXCLUDE_CONTRACT` | Excluded by frozen contract |

Some fields marked as excluded in the original eligibility decision later showed variation in curated Train. Under the conservative no-silent-revival rule, they remained excluded rather than expanding Feature X during the post-curation audit.

## 6.5 Final Train-Only Demotions

Seven of the original 56 candidate source fields were specifically demoted by the final Train-only audit:

| Field | Original candidate status | Final status | Train-only reason |
|---|---|---|---|
| `urgency_numeric` | KEEP | `EXCLUDE_CONSTANT` | invariant in curated Train |
| `secondary_emotion_type` | KEEP_SPARSE | `EXCLUDE_NO_DATA` | no usable Train observation |
| `actual_system_recall_presentation_required` | KEEP | `EXCLUDE_CONSTANT` | invariant in curated Train |
| `memory_recall_triggered_verified_facts` | KEEP | `EXCLUDE_CONSTANT` | invariant in curated Train |
| `memory_retrieval_success_verified_facts` | KEEP | `EXCLUDE_CONSTANT` | invariant in curated Train |
| `recall_source_used` | KEEP | `EXCLUDE_CONSTANT` | invariant in curated Train |
| `recall_outcome_relevance_source` | KEEP_SPARSE | `EXCLUDE_CONSTANT` | invariant in curated Train |

This audit happened before final model fitting and held-out Test evaluation. Feature X was not reopened after Test.

## 6.6 Sparse and Missing Evidence Policy

The feature decision did not attempt to maximize non-null coverage. It preserved the distinction:

```text
missing / unobserved
        ≠
observed negative
        ≠
not applicable
```

The generation, replay, and preprocessing pipeline was not allowed to invent semantic evidence merely to reduce missingness. A field could therefore remain excluded even if the concept was theoretically relevant to a governance construct.

This limitation was especially relevant to EA. The frozen corpus did not provide sufficient support for several direct authorization/lifecycle fields such as `user_acknowledgement`, `denial_intent`, `change_persistence`, `negotiation_started_in_current_session`, `turns_since_negotiation_start`, and `application_ending_criteria`. Beta EA should therefore be read as an anomaly detector over the **observable agency-governance evidence available in this dataset**, not a complete representation of every possible authorization-state failure.

## 6.7 Goal-Specific Representation Intent

The four ModelReadyX views were constructed for different governance questions:

- **UG:** current user semantics, intent/context/topic, clarification, change scope, cross-turn intent/topic/context relationships, recall relevance, and realized system response behavior;
- **SG:** risk, emotion intensity, protection depth, persistent safety-related evidence, planned response constraints/beats/tone, negotiation-related evidence, and realized response behavior;
- **EA:** user/system semantics, change scope, negotiation evidence, planned-versus-realized response behavior, response function/tone/style, recall relevance, and negotiated-behavior state;
- **SF:** runtime state, retry, fallback, output-filter behavior, routing confidence, latency ratio, memory/retrieval behavior, cross-turn carryover, and temporal aggregate evidence.

These assignments were hypotheses about observable representation. They did not make the fields semantic drift labels, and they did not guarantee that the resulting anomaly geometry would identify the corresponding governance construct.

## 6.8 Preprocessing Stages

The preprocessing pipeline was separated into three stages:

```text
P1 → goal-specific raw source-field views
P2 → deterministic categorical / numeric / multi-label representation
P3 → semantic embedding + Train-fitted PCA
```

Semantic free text was not one-hot encoded. `user_prompt_semantic` and `actual_system_response_semantic` were embedded using `text-embedding-3-small`. The embedding dimension was 1,536, followed by a **64-component PCA fitted on curated governance-success Train only**. Validation and Test used the already-fitted transformation.

Categorical, multi-label, ordinal, numeric, missingness, and semantic fields followed their frozen representation decisions. Temporal/derived fields were restricted to lawful current/past observations; no future-turn information was permitted in online `$X_t^*$`.

The final processed matrices contained:

| Split | Completed-turn rows |
|---|---:|
| Train | 1,644 |
| Validation | 692 |
| Test | 707 |

The final goal-specific model dimensions were:

| Goal | Final dimensions |
|---|---:|
| UG | 434 |
| SG | 416 |
| EA | 411 |
| SF | 96 |

The dimensional expansion from 49 retained source fields to hundreds of model columns came from goal-specific selection and preprocessing—not from adding new semantic evidence after the feature freeze.

---

# 7. Why Normal-Only Anomaly Detection?

Beta v0.1 did not formulate the task as:

$$
X \rightarrow \text{supervised drift label}
$$

Instead, the model learned only from successful governance:

$$
X_{\text{success, Train}} \rightarrow \text{normal region}
$$

A new turn was then scored according to its deviation from that learned
region.

This formulation was chosen because drift was expected to be sparse,
heterogeneous, and difficult to exhaustively label. It also allowed the
experiment to ask whether drift could emerge as a departure from
successful longitudinal governance without directly teaching the model
the drift labels.

The tradeoff is fundamental:

> A normal-only anomaly detector learns **normality in the
> representation**, not the semantic definition of drift itself.

The held-out experiment therefore tested whether those two concepts
aligned sufficiently well.

---

# 8. Model Families

Two structurally different normal-only approaches were evaluated.

### Primary candidate --- Isolation Forest

Isolation Forest was selected as the primary candidate because the
representation was high-dimensional, the training distribution contained
only successful governance, anomalies were expected to be sparse, and
online inference needed to remain relatively lightweight.

### Baseline --- RBF One-Class SVM

RBF One-Class SVM was retained as a baseline with a method-specific
`StandardScaler` fitted on Train only.

Four independent goal-specific detectors were trained:

-   UG detector
-   SG detector
-   EA detector
-   SF detector

There was no learned overall detector.

---

# 9. Experimental Discipline

Beta v0.1 used a strict staged evaluation process:

``` text
Dataset Curation
      ↓
Dataset Freeze
      ↓
Feature Eligibility Freeze
      ↓
P1 / P2 / P3 Preprocessing
      ↓
Normal-Only Training
      ↓
Validation
      ↓
Controlled Hyperparameter Selection
      ↓
Final Model Freeze
      ↓
One-Shot Held-Out Test
      ↓
Post-Test Error Analysis
```

Validation was allowed to influence:

-   model configuration;
-   trajectory aggregation;
-   calibration/operating decisions;
-   threshold selection.

Test was not.

Before Test access, the following were frozen:

-   dataset membership and split;
-   Feature X eligibility;
-   preprocessing;
-   model family;
-   hyperparameters;
-   Train-normal calibration;
-   feature order;
-   trajectory aggregation;
-   thresholds;
-   final detector artifacts.

The Test set was then evaluated once. No retraining, threshold search,
aggregation selection, calibration fitting, or model-family selection
was permitted after Test results were observed.

---

# 10. Final Frozen Detectors

The final primary model family was Isolation Forest for all four
dimensions.

| Goal | Isolation Forest configuration | Aggregation | Frozen threshold |
|---|---|---|---:|
| UG | 500 trees, `max_features=0.7` | Mean | 0.6976885645 |
| SG | 500 trees, `max_features=0.7` | Top-K Mean (`k=3`) | 0.8819951338 |
| EA | 500 trees, `max_features=1.0` | Late Mean (last 50%) | 0.4688564477 |
| SF | 300 trees, `max_features=1.0` | Late Mean (last 50%) | 0.9489051095 |

For all four detectors:

-   `max_samples="auto"`
-   `contamination="auto"`
-   `random_state=42`

Turn-level raw anomaly score was defined as:

$$
-\text{decision\_function}(X)
$$

so larger values represented greater anomaly relative to the
Train-normal detector.

Calibration used the empirical distribution of raw scores on
governance-success Train data.

The resulting calibrated score was a **Train-normal empirical
percentile**, not a probability that drift had occurred.

---

# 11. Held-Out Test Results

The frozen experiment was evaluated on **60 held-out trajectories**
containing 40 governance-success trajectories and 20 trajectories with
at least one drift label.

Goal-specific Test results were:

  | Goal | Positives | ROC-AUC | PR-AUC | Precision | Recall | F1 | FPR |
  |---|---:|---:|---:|---:|---:|---:|---:|
  | UG | 3 | 0.398 | 0.057 | 0.000 | 0.000 | 0.000 | 0.123 |
  | SG | 8 | 0.772 | 0.407 | 0.308 | 0.500 | 0.381 | 0.173 |
  | EA | 11 | 0.653 | 0.386 | 0.294 | 0.455 | 0.357 | 0.245 |
  | SF | 2 | 0.957 | 0.643 | 0.000 | 0.000 | 0.000 | 0.000 |

## 11.1 UG

UG did not demonstrate held-out generalization.

All three UG-positive Test trajectories were false negatives at the
frozen threshold. Ranking performance was also poor.

## 11.2 SG

SG produced the strongest credible held-out evidence in Beta v0.1.

It retained meaningful ranking separation and detected four of eight
SG-positive Test trajectories, but it also produced nine false
positives. The result is evidence of partial signal, not reliable
detection.

## 11.3 EA

EA retained weaker but non-trivial ranking signal.

Five of eleven EA-positive trajectories crossed the frozen threshold,
but false positives remained substantial. The detector therefore lacked
the specificity needed for reliable semantic attribution.

## 11.4 SF

SF showed strong ranking metrics, but only **two Test trajectories were
SF-positive** and neither crossed the frozen threshold.

The sample is too small to treat the aggregate metrics as stable. SF
remains exploratory and case-level.

---

# 12. Initial Hypothesis Assessment

### H1 --- Detectability: Partially supported

Some governance dimensions showed held-out anomaly separation, but the
effect was not reliable across all dimensions.

### H2 --- Dimension heterogeneity: Supported within the synthetic Beta experiment

UG, SG, EA, and SF behaved materially differently in Validation and
Test.

### H3 --- Longitudinal aggregation: Hypothesis-consistent evidence, not established

Different dimensions selected different aggregation rules during Validation, and post-Test analysis showed that aggregation could materially change whether local anomaly evidence survived at trajectory level. These observations are consistent with the original hypothesis that longitudinal aggregation can matter, but Beta v0.1 did not run a frozen controlled comparison of trajectory aggregation against a single-turn alternative. It therefore did **not** establish that longitudinal aggregation improved held-out detection or identify one generally superior aggregation strategy.

### H4 --- Novel-normal tolerance: Partially supported

The experiment deliberately included successful Novel Normal
trajectories in the normality design and evaluation. However, held-out
false positives show that tolerance for diverse but valid governance
behavior remained incomplete.

These hypothesis-level observations do **not** establish reliable
semantic drift detection.

---

# 13. Post-Test Error Analysis

The held-out result was not repaired after Test. Instead, the errors
were analyzed to identify what the experiment had actually learned.

## 13.1 UG --- Representation and Goal-Specificity Problems

All three UG-positive Test trajectories were false negatives. One case was particularly informative: the realized trajectory contained repeated explicit user correction while the system continued to follow a stale operative goal. Semantically, the UG failure was clear, yet the learned UG representation remained strongly normal-like.

Conceptually:

``` text
User states operative goal
        ↓
User explicitly corrects / revises goal
        ↓
System continues stale goal
        ↓
Semantic UG drift is present

BUT

Current UG feature representation
        ↓
Isolation Forest
        ↓
Normal-like anomaly evidence
```

This is not well explained by threshold choice alone. Simply lowering
the threshold would also increase false positives.

The case suggests that the representation may not encode
**operative-goal transition, correction, stale-goal reuse, and
recovery** directly enough for the detector to distinguish them from
ordinary semantic continuity.

UG false positives exposed a second problem. Seven Test trajectories
were UG false positives; five of them contained drift in other
governance dimensions.

This pattern raised a **post-Test construct-specificity hypothesis**: the current UG representation may have been sensitive to structure shared with other governance deviations. Because UG detection itself was not validated, this is a diagnostic observation rather than evidence that the detector reliably recognized general governance abnormality.

## 13.2 SG --- Partial Generalization, Incomplete Normal Tolerance

SG detected four of eight positive trajectories.

Its false negatives ranged from a near-threshold miss to substantially
normal-like scores. Its nine false positives were governance-success
trajectories rather than simply other labeled SG failures.

The resulting interpretation is:

-   SG contained the strongest credible goal-specific signal in Beta
    v0.1;
-   that signal remained incomplete;
-   normal-tolerance and specificity remained important limitations.

## 13.3 EA --- Detectable Signal, Poor Specificity

EA detected five of eleven positive trajectories.

The detector responded to some decision-takeover and
epistemic-overbinding trajectories, but it missed others and produced
twelve false positives. Several false positives contained drift in other
governance dimensions, while others were governance-success
trajectories.

The result is consistent with some EA-related statistical signal being present, but it did not establish reliable EA detection or semantic attribution.

## 13.4 SF --- Ranking Signal Without a Validated Operating Point

Both SF-positive Test trajectories received relatively anomalous scores
but remained below the frozen threshold.

With only two positive Validation trajectories and two positive Test
trajectories, Beta v0.1 does not support a stable SF threshold or
performance estimate.

## 13.5 Cross-Goal Diagnostic Observation

Cross-goal false positives were observed, while some semantically positive trajectories remained normal-like. Because the goal-specific detectors did not establish reliable held-out semantic drift detection, these errors should not be interpreted as proof that the models successfully detected broad abnormality but failed only at attribution.

The narrower conclusion is:

> **Beta v0.1 did not establish that anomaly scores from the frozen goal-specific Feature X representations could reliably detect the corresponding semantic governance drift on held-out data.**

The cross-goal error pattern is retained as a diagnostic hypothesis about representation and construct specificity for future work.

---

# 14. Post-Test Artifact Audit

After the negative held-out result, a read-only forensic audit was
performed to test a simpler alternative explanation:

> **Did the model actually train on the intended data, or was the
> negative result caused by a broken training pipeline?**

The audit did not modify the frozen experiment.

## 14.1 Fitted Model Artifacts

The four frozen detector artifacts were confirmed to be fitted
`IsolationForest` models rather than empty or placeholder objects.

Fitted attributes were present, including model feature counts and
estimator collections.

  | Goal | Fitted feature count | Fitted trees |
  |---|---:|---:|
  | UG | 434 | 500 |
  | SG | 416 | 500 |
  | EA | 411 | 500 |
  | SF | 96 | 300 |

The dimensions matched the frozen P3 representations and feature-name
manifests.

The fitted models also recorded `max_samples_ = 256`, consistent with
the use of `max_samples="auto"` on this dataset size. This helps explain
why training completed quickly: Beta v0.1 used small classical anomaly
models, not iterative deep-network training.

## 14.2 Training Rows and Split Isolation

The final Train representation contained:

-   **145 Train trajectories**
-   **1,644 completed-turn rows**

The Train-fitted score references also contained 1,644 values for each
detector.

Direct trajectory-ID comparison found:

-   Train ∩ Validation = 0
-   Train ∩ Test = 0
-   Validation ∩ Test = 0

No trajectory-level split leakage was found in the frozen P3 matrices.

## 14.3 Matrix Integrity

The audit also confirmed:

-   goal-specific P3 dimensions matched frozen model dimensions;
-   Train, Validation, and Test feature ordering was consistent;
-   no duplicate feature-column names were found;
-   no NaN values were found;
-   no infinite values were found;
-   the Train-fitted semantic PCA state existed;
-   the recorded fit scope was governance-success Train only.

The artifact audit therefore did **not** support the explanation that
Beta v0.1 failed simply because the model never trained, used an
obviously mismatched feature matrix, or suffered an obvious
trajectory-level split error.

The negative held-out result should therefore be treated as a
substantive experimental result unless a later audit discovers a new
implementation defect.

---

# 15. Representation Audit Findings

The artifact audit also revealed limitations in the statistical support
of the frozen representation.

## 15.1 Constant Features in Train

Within the 1,644 Train turns:

  | Goal | Total features | Constant Train features | Approx. constant share |
  |---|---:|---:|---:|
  | UG | 434 | 108 | 24.9% |
  | SG | 416 | 76 | 18.3% |
  | EA | 411 | 76 | 18.5% |
  | SF | 96 | 28 | 29.2% |

A constant Train feature is not automatically a design error. Some
runtime states or categories may legitimately never occur in successful
Train trajectories.

However, the result shows that the nominal training support was narrower
than the raw dimensionality alone suggests.

This is a **diagnostic observation**, not proof that constant features
caused held-out failure.

## 15.2 SF Representation Diversity

Representation diversity was especially limited for SF.

For UG, SG, and EA, all 1,644 Train rows had distinct final feature
vectors.

For SF:

> **1,644 Train turns collapsed to only 206 unique feature vectors.**

This is consistent with the nature of the SF construct: under
deterministic successful replay, most turns contain no runtime failure,
failed retry, failed fallback, or broken recovery state.

The finding raises a formulation question:

> If SF is primarily defined by rare runtime-control and recovery
> events, is normal-only anomaly detection over this representation the
> right learning formulation?

Beta v0.1 does not answer that question.

---

# 16. Deeper Diagnosis: The Feature X → Semantic Y Assumption

The combined Test result, error analysis, and artifact audit point to a
deeper issue than model training speed or a single threshold.

The original experiment effectively tested:

$$
\text{Deviation from normal } X^{goal}
\stackrel{?}{\approx}
\text{semantic drift in } Y_{goal}
$$

The evidence suggests that this relationship was not sufficiently
reliable.

## 16.1 Cross-Goal False Positives Raised a Construct-Specificity Hypothesis

Cross-goal false positives were consistent with the possibility that some goal-specific feature views responded to statistical structure that was not specific to the intended semantic construct. For example, a trajectory labeled as successful for one target dimension could still contain unusual structure or drift in another dimension and receive an elevated score from the first detector.

Because Beta v0.1 did **not** establish reliable held-out semantic drift detection, this pattern should not be interpreted as evidence that the detectors successfully recognized a general class of governance abnormality and merely failed to attribute its cause. The narrower interpretation is diagnostic:

> **The observed error pattern raised a hypothesis that the frozen goal-specific representations may not have been sufficiently construct-specific.**

Testing that hypothesis would require a new experiment with stronger construct-controlled contrasts; Beta v0.1 did not establish it.

## 16.2 Semantic Drift Can Exist Without Becoming Anomalous in the Current Representation

The UG false-negative analysis in Section 13.1 illustrates this possibility within the held-out cases: trajectories judged as semantic UG drift under independent QA could remain normal-like in the frozen representation.

This provides case-level evidence that semantic drift in $Y$ was not necessarily expressed as sufficient statistical deviation in the current $X$.

## 16.3 The Central Measurement Finding

Beta v0.1 therefore challenges the assumption that:

> **anomaly in the current goal-specific Feature X representation is a
> sufficient proxy for semantic trajectory drift.**

It does **not** prove that Feature X is categorically wrong, nor that
dataset design is the sole cause of poor generalization.

Several explanations may interact:

-   synthetic dataset coverage;
-   small positive evaluation sets;
-   limited diversity in parts of the normal distribution;
-   representation design;
-   insufficient separation between goal-specific feature views;
-   fixed-dimensional encoding of longitudinal relationships;
-   model-family limitations;
-   the normal-only anomaly formulation itself.

Beta v0.1 cannot isolate a single causal root cause.

---

# 17. Reframing the Research Problem

The initial workflow was approximately:

``` text
Available Backend Evidence
        ↓
Feature X
        ↓
Goal-Specific Anomaly Detector
        ↓
Compare Anomaly Evidence with Semantic Drift Y
```

The experiment suggests that a stronger next-step research process
should begin from the construct:

``` text
Governance Construct Y
        ↓
What observable evidence makes Y identifiable?
        ↓
Measurement Design
        ↓
Feature Representation X
        ↓
Dataset Design
        ↓
Model
```

In shorthand:

$$
\boxed{Y \rightarrow \text{Evidence} \rightarrow X \rightarrow \text{Model}}
$$

This is a research lesson from Beta v0.1, not a claim that one
replacement architecture has already been validated.

The key question is no longer only:

> **Which anomaly algorithm should be used?**

It is:

> **What observable structure must exist for a longitudinal governance
> construct to become statistically identifiable?**

---

# 18. Research Directions for a Future Version

Beta v0.1 is closed. The following are future research questions, not
post-Test repairs.

## 18.1 UG --- Represent Operative-Goal Transitions Explicitly

A future UG representation should investigate direct observables for:

-   previous operative goal;
-   current operative goal;
-   explicit user correction or revision;
-   system-used goal;
-   stale-goal reuse;
-   recovery after correction.

The objective would be to represent the **relationship between goal
transition and system adaptation**, rather than relying mainly on
generic semantic abnormality.

## 18.2 EA --- Represent Authority Transfer and Boundary Behavior

A future EA representation could investigate explicit structure for:

-   authority-transfer requests;
-   system acceptance or refusal;
-   narrowing of decision space;
-   repeated decision handover;
-   reinforcement of dependency;
-   boundary recovery.

## 18.3 Matched Contrastive Trajectories

Future dataset design should investigate stronger contrastive control.

For the same or closely matched context, language intensity, trajectory
length, and topic, construct trajectories such as:

``` text
EA drift / SG success / UG success
EA success / SG drift / UG success
EA success / SG success / UG drift
all dimensions successful
```

This would test whether a representation can distinguish governance
mechanisms rather than simply distinguish ordinary from unusual
conversations.

## 18.4 Broader Normal Support

False positives in SG and EA suggest that the successful-governance
distribution should cover more diverse but valid behavior.

A future experiment should investigate whether broader normal support
improves tolerance for novel-but-valid trajectories without erasing true
drift signal.

## 18.5 Revisit the SF Formulation

Because SF failures are rare and the successful SF representation was
highly repetitive, future work should reconsider whether normal-only
anomaly detection is the right formulation for SF.

Alternatives should be evaluated only after the SF construct and
observable evidence are specified clearly.

## 18.6 Larger Evaluation

Future work should also include:

-   more positive trajectories per governance dimension;
-   more SF-positive cases;
-   broader scenario contexts;
-   stronger family-level holdout;
-   eventually, carefully governed non-synthetic or real human--AI
    interaction data.

---

# 19. What Beta v0.1 Demonstrates

Within the scope of this synthetic feasibility experiment, Beta v0.1 showed that:

- the four governance dimensions produced materially different held-out statistical patterns under the frozen representations and detectors;
- SG retained partial held-out ranking signal in this experiment;
- post-Test error analysis identified representation and measurement concerns that were not apparent from aggregate metrics alone;
- artifact-level audit ruled out several obvious pipeline-failure explanations for the negative held-out result.

---

# 20. What Beta v0.1 Does Not Demonstrate

Beta v0.1 does **not** demonstrate that:

-   SIIHA can reliably detect trajectory drift;
-   a calibrated anomaly score is a probability of drift;
-   a detector's goal name confirms the semantic category of drift;
-   an elevated anomaly score identifies the cause of drift;
-   individual Feature X fields caused a governance failure;
-   Isolation Forest is the best model family for trajectory drift;
-   SF performance is validated;
-   the current representation is sufficient for goal-specific
    attribution;
-   dataset design or Feature X has been proven to be the sole cause of
    poor performance;
-   the result generalizes to real-world human--AI interaction;
-   the experimental detector is ready to govern production behavior.

These boundaries are part of the result.

---

# 21. Relationship to SIIHA Runtime Governance

The trajectory-drift experiment is **not the SIIHA runtime-governance
system itself**.

The implemented governance architecture operates conceptually as:

``` text
Conversation
    ↓
Longitudinal Trajectory / Continuity
    ↓
Governed Memory and Runtime Evidence
    ↓
System-Side Governance
    ↓
Response Boundary / Runtime Action
```

The experimental ML layer is separate:

``` text
Runtime Evidence
    ↓
Feature X
    ↓
Frozen Goal-Specific Anomaly Detector
    ↓
Experimental Longitudinal Anomaly Evidence
```

The deterministic governance layer does not need an ML anomaly score in
order to preserve user agency, enforce a response boundary, maintain
memory discipline, or perform runtime recovery.

This separation is intentional.

The Beta v0.1 detector should therefore be understood as an
**experimental measurement layer**, not as the governance authority of
SIIHA.

---

# 22. Final Conclusion

SIIHA Beta v0.1 successfully completed the planned experimental
pipeline:

> synthetic scenario design → deterministic replay → independent
> semantic QA → conservative curation → dataset and feature freeze →
> Train-only preprocessing → normal-only training → Validation-only
> model selection → final freeze → one-shot held-out Test → post-Test
> error analysis → artifact audit.

The engineering pipeline completed successfully, and the subsequent
artifact audit did not find evidence that the negative result was caused
by an absent training run, an obvious feature-dimension mismatch,
NaN/Inf corruption, or trajectory-level Train/Validation/Test leakage.

The ML result, however, was substantially more limited.

SG retained the strongest credible held-out signal. EA retained weaker
signal with poor specificity. UG did not generalize under the current
representation. SF remained too underpowered for a stable performance
claim.

More importantly, the experiment exposed a measurement problem:

> **statistical deviation from a goal-specific normal feature space is
> not, by itself, reliable evidence that the corresponding semantic
> governance dimension has drifted.**

The most important outcome of Beta v0.1 is therefore not a validated
detector. It is a more precise research problem.

The next question is not simply which anomaly-detection algorithm should
replace Isolation Forest. It is how longitudinal governance constructs
should be operationalized so that the observable evidence and feature
representation actually correspond to the semantic phenomenon being
measured.

Beta v0.1 therefore closes as an **error-discovery and
measurement-refinement experiment**, while the deterministic SIIHA
runtime-governance architecture remains a separate implemented system
layer.

---

# Appendix A --- Frozen Experiment Snapshot

  | Item | Frozen Beta v0.1 value |
  |---|---|
  | Raw corpus | 400 trajectories / 4,627 completed turns |
  | Semantic QA | 258 ACCEPT / 134 REJECT / 8 REVIEW |
  | Frozen dataset | 263 trajectories |
  | Quarantined | 137 trajectories |
  | Train | 145 success trajectories / 1,644 turns |
  | Validation | 58 trajectories / 692 turns |
  | Test | 60 trajectories / 707 turns |
  | Eligible source fields | 49 |
  | UG dimensions | 434 |
  | SG dimensions | 416 |
  | EA dimensions | 411 |
  | SF dimensions | 96 |
  | Primary model | Isolation Forest |
  | Baseline | RBF One-Class SVM |
  | Calibration | Train-normal empirical percentile |
  | Test policy | One-shot held-out Test; no post-Test tuning |

# Appendix B --- Internal Source Records

This public document consolidates the Beta v0.1 research narrative from
the detailed development records, including:

-   `SIIHA_Beta_v0.1_Final_Experimental_Report.md`
-   `trajectory_drift_dataset_contract.md`
-   `trajectory_drift_detection_synthetic_scenario_contract.md`
-   `trajectory_drift_detection_model_contract.md`
-   `trajectory_drift_detection_preprocessing_contract.md`
-   `trajectory_drift_model_selection.md`
-   `dataset_curation_manifest_v0_1.md`
-   `model_ready_x_decision_v0.1.md`
-   `README_TRAINING.md`
-   `README_VALIDATION.md`
-   `hyperparameter_tuning_contract_v0.1.md`
-   `final_model_freeze_contract_v0.1.md`
-   `held_out_test_contract_v0.1.md`

The post-Test artifact and representation audit summarized here was
performed after the frozen experiment to investigate the negative
result. It was read-only and did not modify the Beta v0.1 model,
thresholds, dataset, or Test result.

---

# 23. Follow-Up: Beta v0.2

Beta v0.1 did not establish that its turn-centered representation was the sole cause of the held-out limitations. The post-Test findings instead motivated a narrower **measurement hypothesis**.

SIIHA's runtime trajectory ontology is continuity-defined: adjacent completed turns belong to the same longitudinal trajectory only when continuity is established. Beta v0.1, by contrast, primarily represented each completed turn as one fixed-dimensional observation, `$X_t^*$`, and then aggregated turn-level anomaly scores.

This raised the following follow-up question:

> **Does aligning the ML unit of analysis with continuity-valid governance transitions improve SG and EA drift identifiability compared with Beta v0.1's turn-centered representation?**

Beta v0.2 was designed to test that question without reopening or modifying Beta v0.1.

The follow-up narrowed the scope to SG and EA and proposed a shared edge-centered semantic representation over continuity-valid adjacent turns. However, the frozen Beta v0.2 synthetic dataset did not pass its prerequisite semantic-feasibility gate after the allowed regeneration budget.

As a result:

```text
Beta v0.2 dataset feasibility:     NOT PASSED
Edge feature extraction:           NOT RUN
Model training:                    NOT RUN
Held-out Test:                     NOT ACCESSED
Edge-representation hypothesis:    UNTESTED
```

This does **not** retroactively establish that Beta v0.1's turn-centered representation was the causal reason for its held-out limitations. It also does not establish that edge representation would improve them.

The complete follow-up record is available in [BETA_V0_2_DATASET_FEASIBILITY.md](./BETA_V0_2_DATASET_FEASIBILITY.md).

The combined measurement lessons from both experiments are summarized in [RESEARCH_FINDINGS.md](./RESEARCH_FINDINGS.md).
