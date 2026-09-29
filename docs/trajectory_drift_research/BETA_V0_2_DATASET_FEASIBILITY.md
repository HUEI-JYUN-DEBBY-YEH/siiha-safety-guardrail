# SIIHA Beta v0.2 — Dataset Feasibility for Continuity-Valid Edge Representation

**Status:** CLOSED — dataset feasibility gate not passed  
**Experiment:** Continuity-Valid Edge Representation Feasibility Study  
**Scope:** Safety Goal (SG) + Epistemic Agency (EA)  
**Planned dataset:** 90 trajectories  
**ML experiment:** **NOT RUN**  
**Edge-representation hypothesis:** **UNTESTED**

**Research track:** [Overview](./README.md) · [Beta v0.1](./BETA_V0_1_EXPERIMENT.md) · [Cross-Version Findings](./RESEARCH_FINDINGS.md)

---

## Executive Summary

Beta v0.2 was a narrow follow-up to Beta v0.1.

Beta v0.1 primarily represented one completed interaction turn as one fixed-dimensional observation and then aggregated turn-level anomaly evidence across a trajectory. Post-Test analysis raised a new measurement hypothesis: because SIIHA's runtime trajectory ontology is continuity-defined, some longitudinal governance constructs might be better represented as **relationships between adjacent continuity-valid turns**.

Beta v0.2 therefore asked:

> **Does aligning the ML unit of analysis with continuity-valid governance transitions improve SG and EA drift identifiability compared with Beta v0.1's turn-centered representation?**

The experiment deliberately did **not** assume that edge representation was superior.

Before any edge feature extraction or model training, the experiment required a frozen 90-trajectory synthetic dataset to pass independent semantic validation.

The initial generation completed all 90 required realizations. Blind edge-level Semantic QA and deterministic trajectory adjudication were then used to derive observable trajectory truth independently from generation intent.

Initial curation produced:

| Decision | Count | Rate |
|---|---:|---:|
| ACCEPT | 71 | 78.9% |
| REJECT | 19 | 21.1% |
| REVIEW | 0 | 0.0% |

Only the 19 rejected required realizations entered regeneration. Each could receive at most three fresh attempts using the same frozen scenario metadata and the same frozen user trajectory, without detailed prior QA/rejection feedback being given to the generator.

Across regeneration, the pipeline produced **51 fresh realizations**, each independently judged, adjudicated, and curated. Only **5 of the 19** required replacements were recovered. The remaining **14** exhausted the three-attempt limit and were recorded as `scenario_generation_failure`.

Therefore:

```text
replacement_set_complete = false
dataset feasibility gate = NOT PASSED
edge feature extraction = NOT RUN
model training = NOT RUN
held-out ML evaluation = NOT RUN
edge-representation hypothesis = UNTESTED
```

The experiment was intentionally stopped at the frozen semantic-feasibility boundary rather than training on a post-hoc partial dataset.

---

# 1. Relationship to Beta v0.1

Beta v0.1 completed a normal-only anomaly-detection experiment over four governance constructs: UG, SG, EA, and SF.

Its held-out results were dimension-specific:

- SG retained the strongest credible signal but was not reliable enough for validated detection;
- EA retained weaker signal with substantial false positives;
- UG did not demonstrate held-out generalization;
- SF was too underpowered for a stable performance claim.

Post-Test error analysis raised a more basic measurement concern: Beta v0.1 did not establish that anomaly scores from the frozen goal-specific representations could reliably detect the corresponding semantic governance drift on held-out data. Cross-goal false positives were treated as a diagnostic hypothesis about representation and construct specificity, not as evidence of a validated broad-anomaly detector with an attribution problem.

The central v0.1 measurement concern became:

> **Deviation from normal $X^{goal}$ was not established as a sufficient proxy for semantic governance drift $Y_{goal}$.**

Beta v0.2 did not reopen v0.1. It tested a new unit-of-analysis hypothesis motivated by the post-Test analysis.

---

# 2. Research Question

SIIHA's runtime trajectory ontology is graph-based.

A completed turn can be represented as a node `N_t`. An adjacent pair becomes a longitudinal transition only when continuity is established.

Conceptually:

```text
N_(t-1) ── continuity-valid ──> N_t
```

Beta v0.2 asked:

> **Does aligning the ML unit of analysis with continuity-valid governance transitions improve SG and EA drift identifiability compared with Beta v0.1's turn-centered representation?**

This was a representation and measurement feasibility question, not a production-readiness claim.

---

# 3. Scope

Beta v0.2 intentionally reduced the target space.

## Included

- Safety Goal (SG)
- Epistemic Agency (EA)
- continuity-valid transitions
- edge-centered representation
- normal-only model training, if the dataset passed feasibility
- successful-governance Train data
- success and deviation examples in Validation and held-out Test

## Excluded

- User Goal (UG)
- System Function (SF)
- recovery modeling
- compound SG+EA drift as a primary target
- an overall drift detector
- production deployment claims
- production runtime integration as a requirement
- reopening Beta v0.1
- interpretive overbinding as an EA target

---

# 4. Constructs

## 4.1 Safety Goal (SG)

SG concerns whether system behavior adapts appropriately when the safety significance of an interaction changes.

Conceptually:

```text
SG Success =
Proportional Safety Adaptation
+
Safety Boundary Preservation
```

The frozen deviation families were:

- **SG-D1 — Stale Safety Strategy:** safety significance changes meaningfully, but the system fails to adapt its safety strategy.
- **SG-D2 — Disproportionate or Monolithic Safety Strategy:** system strategy is not proportionate to realized safety significance.
- **SG-D3 — Safety Boundary Violation:** the system crosses a frozen safety boundary.

Emotion alone was not treated as an SG transition.

## 4.2 Epistemic Agency (EA)

EA concerns whether the system preserves the user's meaningful reasoning, judgment, and decision authority.

Conceptually:

```text
EA Success =
Agency Support
+
Authority Boundary Preservation
```

The frozen deviation families were:

- **EA-D1 — Explicit Authority Acceptance:** the user attempts to transfer decision authority and the system accepts that transfer.
- **EA-D2 — Progressive Unjustified Decision-Space Narrowing:** across continuity-valid transitions, the system progressively narrows the user's decision space without sufficient grounding in user-provided constraints, preferences, or chosen priorities.

---

# 5. Continuity Gate

Beta v0.2 separated two questions:

1. **Does continuity exist?**
2. **Given continuity, did governance behavior adapt appropriately?**

Continuity used the conceptual SIIHA trajectory criteria:

- primary intent continuity;
- secondary intent continuity;
- context continuity;
- topic continuity.

Continuity evidence was a gate, not an SG or EA outcome label.

```text
continuity = false
    → no SG/EA longitudinal evaluation edge

continuity = true
    → construct edge representation
```

Each planned trajectory contained 2–10 turns, and every adjacent transition within a generated trajectory was required to be continuity-valid.

---

# 6. Planned Edge Representation

The planned model-visible representation preserved both sides of a continuity-valid transition.

For edge $E_{t-1,t}$:

```text
User side:
U_edge = [U_prev, U_curr, Delta_U]

System side:
S_edge = [S_prev, S_curr, Delta_S]

Combined:
X_edge = [
    U_prev,
    U_curr,
    Delta_U,
    S_prev,
    S_curr,
    Delta_S
]
```

SG and EA were planned to use the **same X architecture**, with separate construct-specific normal detectors.

The first representation intentionally excluded Semantic-QA-derived target-proximal labels from model X.

The planned semantic preprocessing was:

```text
text
  ↓
text-embedding-3-small
  ↓
new Beta v0.2 PCA fitted on Train only
  ↓
z_prev, z_curr
  ↓
delta_z = z_curr - z_prev
```

The final planned representation was:

```text
X_edge = [
    U_prev_z,
    U_curr_z,
    U_delta_z,
    S_prev_z,
    S_curr_z,
    S_delta_z
]
```

Beta v0.1 PCA objects were not to be reused.

**This representation was designed but never evaluated.** Dataset feasibility failed before preprocessing and model training.

---

# 7. Frozen Dataset Design

The planned allocation was:

| Split | SG Success | EA Success | SG Deviation | EA Deviation | Total |
|---|---:|---:|---:|---:|---:|
| Train | 30 | 20 | 0 | 0 | 50 |
| Validation | 6 | 4 | 6 | 4 | 20 |
| Test | 6 | 4 | 6 | 4 | 20 |
| **Total** | **42** | **28** | **12** | **8** | **90** |

Train was success-only.

Validation and Test each contained 10 matched scenario pairs:

- 10 success trajectories;
- 10 matched deviation trajectories.

Each matched pair shared the **exact same frozen user turns** and differed in system realization.

Deviation allocation per Validation/Test split was:

- SG-D1 × 2
- SG-D2 × 2
- SG-D3 × 2
- EA-D1 × 2
- EA-D2 × 2

## 7.1 Evaluation Mechanism Matrix

Each Validation and Test split used the following matched design:

| Construct / mechanism | Success | Deviation | Matched pairs per split |
|---|---:|---:|---:|
| SG-D1 — Stale Safety Strategy | 2 | 2 | 2 |
| SG-D2 — Disproportionate / Monolithic Safety Strategy | 2 | 2 | 2 |
| SG-D3 — Safety Boundary Violation | 2 | 2 | 2 |
| EA-D1 — Explicit Authority Acceptance | 2 | 2 | 2 |
| EA-D2 — Progressive Unjustified Decision-Space Narrowing | 2 | 2 | 2 |
| **Total per split** | **10** | **10** | **10** |

Across Validation + Test, this produced four success and four deviation realizations for each frozen mechanism. The matched pair shared the exact same frozen user turns; only the system realization branch differed.

The 90-scenario registry used four context families across the full design: **Study, Work, Family, and AI Relationship**. SG used all four; EA used Study, Work, and Family. Context assignment was part of the frozen scenario design rather than a post-generation relabeling step.

## 7.2 Two-Stage Synthetic Generation Pipeline

Unlike Beta v0.1's whole-trajectory generation, Beta v0.2 separated **user-trajectory construction** from **system-response realization**. No SIIHA backend call was used to generate these conversations.

### Stage A — Generate and freeze the user trajectory

The Stage-A generator received the frozen scenario/context information required to construct the user side and the exact required turn count. It generated **user messages only**; it did not generate assistant responses.

```text
Frozen scenario/context + required turn count
        ↓
Stage-A generator
        ↓
U1 → U2 → ... → Un
        ↓
FrozenUserTrajectory
```

Production generation produced **70 frozen user trajectories** for the 90 required system realizations. The number is smaller than 90 because Validation/Test success/deviation pairs reused the exact same frozen user trajectory.

### Stage B — Sequentially realize the assistant side

The Stage-B generator then produced **one assistant response at a time**. For response `R_t`, the model received only the completed conversation history through `t-1` plus the current user message `U_t`. Future user turns were not supplied to that response-generation call.

```text
U1                         → R1
U1 + R1 + U2               → R2
U1 + R1 + U2 + R2 + U3     → R3
...
```

Equivalently:

$$
R_t = f(U_{\leq t}, R_{<t})
$$

This preserved a runtime-like temporal information boundary during system realization and prevented future user turns from being used to construct an earlier assistant response.

For Validation/Test matched pairs, Stage B branched from the same frozen user trajectory:

```text
                 Frozen U1 ... Un
                       │
              ┌────────┴────────┐
              ↓                 ↓
       success realization   deviation realization
       R1s ... Rns           R1d ... Rnd
```

The purpose was to hold the user-side trajectory constant while varying the intended system governance behavior. This is a matched synthetic control, not a causal counterfactual claim.

### Stage-B behavior instructions

Success branches received the construct-level successful-governance requirement. Deviation branches received a frozen observable behavioral manipulation for SG-D1, SG-D2, SG-D3, EA-D1, or EA-D2. The generator was not asked to decide whether its own output semantically constituted drift; that judgment belonged to the blind Semantic QA stage.

SG-D1 used a template-assisted realization path after direct prompting proved difficult during smoke testing. The user trajectory remained frozen; Python controlled the structural persistence step and the model supplied bounded surface-language components. This was frozen before the 90-trajectory production run rather than introduced after observing production QA results.

### Why this differs from v0.1

Beta v0.1 jointly synthesized user utterances, assistant responses, plans, and evidence for an entire trajectory in one LLM realization. Beta v0.2 instead separated user-side generation from system-side sequential realization and introduced exact matched user trajectories for evaluation pairs.

The experiment did **not** perform a controlled ablation between these two generation architectures. The difference is therefore documented as methodological evolution, not evidence that the v0.2 generator was better or that this architecture caused the later feasibility failure.

### 7.3 Model Provenance for Synthetic Generation and Semantic QA

The production v0.2 run used **GPT-5.6 Luna** for both synthetic generation and Blind Semantic QA, through separately configured generation and QA model settings. The two roles remained procedurally separated: the generator received the information required to realize the frozen scenario, while the Blind Semantic Judge received only the realized conversation, frozen semantic rubrics, and required edge identifiers.

Using the same model family in these two roles does **not** make generation intent equivalent to semantic truth. The Judge was blinded to generation role, intended outcome/mechanism, scenario target, model-visible X, detector output, and prior rejection feedback.

Beta v0.2 did not compare generator models or Judge models. The experiment therefore cannot isolate whether the observed feasibility failure arose from generator-model behavior, Judge-model behavior, prompt design, scenario specification, construct definition, matched-pair constraints, or interactions among them. Model identity is reported as experimental provenance, not as a causal explanation.

---

# 8. Truth Separation

A central v0.2 rule was:

> **For LLM-generated synthetic trajectories, the intended scenario label was not treated as ground truth. The realized conversation had to independently satisfy the frozen semantic rubric before it could enter the dataset.**

The measurement pipeline was:

```text
Frozen Scenario Registry
        ↓
User Trajectory Generation
        ↓
System Realization
        ↓
Blind Edge-Level Semantic QA
        ↓
Deterministic Trajectory Adjudication
        ↓
Dataset Curation
        ↓
Dataset Feasibility Gate
        ↓
Edge Representation
        ↓
Preprocessing
        ↓
Model
```

The Blind Semantic Judge did not receive:

- generation role;
- intended outcome or deviation mechanism;
- scenario target;
- model-visible X;
- anomaly scores;
- detector predictions;
- thresholds;
- prior generator critique or rejection feedback.

Semantic interpretation and deterministic trajectory aggregation were separated.

---

# 9. Initial Generation and Semantic QA

Initial production generation completed:

| Item | Count |
|---|---:|
| Frozen user trajectories | 70 |
| Required realized trajectories | 90 |
| Realized trajectories produced | 90 |

All 90 realized trajectories then completed blind edge-level Semantic QA and deterministic trajectory adjudication.

Across the initial set, all 90 trajectories were adjudicated as continuity-valid.

The independently derived trajectory outcomes were:

| Construct | Outcome | Count |
|---|---|---:|
| SG | deviation | 6 |
| SG | success | 39 |
| SG | not applicable | 45 |
| EA | deviation | 6 |
| EA | success | 75 |
| EA | not applicable | 9 |

These outcomes were derived from realized behavior rather than generator metadata.

---

# 10. Initial Curation

Curation compared frozen generation intent with independently derived trajectory truth.

The result was:

| Decision | Count | Rate |
|---|---:|---:|
| ACCEPT | 71 | 78.9% |
| REJECT | 19 | 21.1% |
| REVIEW | 0 | 0.0% |
| **Total** | **90** | **100%** |

Recorded rejection reasons included:

| Reason | Count |
|---|---:|
| `trajectory_truth_mismatch` | 17 |
| `intended_deviation_not_realized` | 9 |
| `cross_construct_deviation` | 1 |
| `wrong_deviation_mechanism` | 1 |

Reason counts were not mutually exclusive.

The dataset was therefore not freeze-ready after the first pass.

---

# 11. Contract-Defined Regeneration

Only the 19 rejected required realizations entered regeneration.

The 71 accepted source realizations were never regenerated.

Each rejected realization could receive at most three fresh attempts. Every attempt reused:

- unchanged frozen scenario metadata;
- the original frozen user trajectory.

The generator did **not** receive detailed prior QA or rejection feedback.

Every fresh realization independently passed through:

```text
Fresh System Realization
        ↓
Blind Semantic QA
        ↓
Deterministic Trajectory Adjudication
        ↓
Curation
```

Across regeneration:

| Operation | Count |
|---|---:|
| Fresh realizations generated | 51 |
| Blind QA judgments | 51 |
| Deterministic adjudications | 51 |
| Curation decisions | 51 |

Final result among the 19 rejected requirements:

| Final status | Count | Rate |
|---|---:|---:|
| Accepted replacement found | 5 | 26.3% |
| `scenario_generation_failure` | 14 | 73.7% |
| REVIEW | 0 | 0.0% |

The public report does not use internal scenario IDs as the primary explanation of regeneration outcome. At the experiment level, the relevant result was:

| Regeneration outcome | Required scenarios |
|---|---:|
| Accepted replacement recovered | 5 |
| Exhausted three-attempt limit | 14 |
| **Total source rejects processed** | **19** |

At the scenario-family level, the regeneration outcome was concentrated rather than uniform:

| Frozen requirement family among the 19 source rejects | Initial rejects | Recovered within regeneration budget | Exhausted three-attempt limit |
|---|---:|---:|---:|
| SG success requirements | 8 | 1 | 7 |
| SG-D1 — Stale Safety Strategy deviation | 4 | 0 | 4 |
| SG-D2 — Disproportionate / Monolithic Safety Strategy deviation | 3 | 3 | 0 |
| SG-D3 — Safety Boundary Violation deviation | 1 | 1 | 0 |
| EA-D2 — Progressive Unjustified Decision-Space Narrowing deviation | 3 | 0 | 3 |
| EA-D1 — Explicit Authority Acceptance deviation | 0 | 0 | 0 |
| **Total** | **19** | **5** | **14** |

This table is descriptive, not a mechanism-level performance estimate. The cell sizes were small, the experiment was not designed to compare generation difficulty statistically across mechanisms, and the causal source of the repeated failures was not isolated. It does, however, show why replacing the outcome with a list of internal scenario IDs would obscure the experimentally relevant structure.

The unresolved requirements were not random missing samples: they arose from specific frozen scenario/mechanism requirements that repeatedly failed semantic curation. This is why the remaining accepted realizations were not collapsed into a post-hoc 76-sample ML dataset. Internal scenario IDs and attempt-level lineage remain available in the private audit artifacts for traceability.

The frozen regeneration process therefore ended with:

```text
regeneration_complete = true
replacement_set_complete = false
```

---

# 12. Feasibility Decision

**Dataset feasibility gate: NOT PASSED.**

The experiment required the frozen scenario allocation to be semantically realized before edge feature extraction and model training.

After the full allowed regeneration budget:

- 71/90 initial realizations were accepted;
- 5/19 rejected requirements obtained accepted replacements;
- 14 required realizations still lacked an accepted realization;
- all 14 had exhausted the maximum of three fresh attempts;
- the required replacement set remained incomplete.

The experiment therefore stopped.

No fourth regeneration attempt was allowed. Rejected conversations were not manually repaired, silently relabeled, or replaced with different scenarios.

---

# 13. Why the Partial 76-Sample Set Was Not Used for ML

After regeneration, the experiment had 71 initial accepted realizations plus 5 accepted replacements.

It would have been possible to form a partial set of 76 accepted realizations.

That set was **not** substituted for the frozen 90-realization design.

The missing realizations were created by scenario-specific semantic failure rather than by a pre-specified random sampling process. Keeping only the realizations that were easier to generate and validate would alter the planned scenario/mechanism coverage after observing the outcomes.

In other words:

```text
P(sample enters final dataset | scenario / mechanism)
```

was no longer safely treated as independent of the experimental construct.

Continuing with the partial set would therefore answer a different question from the frozen experiment.

---

# 14. What the Result Means

The supported conclusion is narrow:

> **Under the frozen Beta v0.2 synthetic generation, blind semantic QA, curation, and regeneration protocol, the required SG/EA realization set could not be constructed reliably enough to proceed with the planned edge-representation ML experiment.**

The experiment therefore records:

```text
Dataset feasibility:        NOT PASSED
Edge feature extraction:    NOT RUN
PCA fitting:                NOT RUN
Model training:             NOT RUN
Validation selection:       NOT RUN
Held-out ML evaluation:     NOT RUN
Edge hypothesis:            UNTESTED
```

---

# 15. What the Result Does Not Mean

Beta v0.2 does **not** demonstrate that:

- continuity-valid edge representation is ineffective;
- SG or EA drift cannot be detected;
- SG or EA drift cannot be generated under any protocol;
- trajectory drift is not learnable;
- edge representation would or would not outperform Beta v0.1;
- the synthetic generator is the sole cause of the feasibility failure;
- the Blind Semantic Judge is the sole cause of the feasibility failure.

The experiment did not reach the stage required to evaluate those claims.

---

# 16. Research Implications

## 16.1 Intended Synthetic Labels Required Independent Validation

For the LLM-generated synthetic trajectories in this experiment, asking the generator to realize a success or deviation scenario did not guarantee that the resulting conversation satisfied the frozen SG/EA rubric.

```text
intended synthetic scenario
        ↓
LLM realization
        ↓
blind semantic evaluation
        ↓
validated semantic outcome
```

The intended label was therefore treated as a generation requirement, not as observed truth. Blind Semantic QA materially changed which generated samples were eligible for the dataset. This conclusion is limited to the synthetic generation and validation setup used here; it is not a claim that all synthetic data or all LLM generators behave this way.

## 16.2 Synthetic Construct Realizability Had to Be Measured

Some frozen SG/EA requirements repeatedly produced realizations that passed semantic curation, while others did not under the allowed generation budget.

Therefore, under this frozen generation protocol, a well-specified scenario and prompt were not sufficient evidence that the required semantic realization could be produced reliably within the allowed generation budget.

The causal source was **not isolated**. Possible contributors include generator-model behavior, prompt design, scenario specification, matched-pair constraints, construct definition, blind-Judge interpretation, or interactions among them. Beta v0.2 did not compare generator models, so the possibility that the generator's own behavioral constraints made some requested deviations difficult to realize remains a future hypothesis rather than an identified cause.

## 16.3 Semantic Dataset Validity Preceded Model Optimization

The planned ML question required semantically valid SG/EA success and deviation examples. A downstream anomaly score could not determine whether a requested synthetic construct had actually been realized.

For that reason the experiment used the order:

```text
scenario specification
        ↓
synthetic realization
        ↓
independent semantic validation
        ↓
dataset freeze
        ↓
representation / ML
```

rather than training first and using model performance to retroactively justify uncertain labels. When the required semantic dataset could not be completed, feature extraction and ML were intentionally not run.

## 16.4 Predefined Stop Conditions Preserved the Failure Result

Regeneration was capped at three fresh attempts per rejected required realization. Continuing to regenerate indefinitely until the desired label appeared would have hidden the fact that some frozen requirements were difficult to realize under the protocol.

The remaining `scenario_generation_failure` cases were therefore retained as experimental outcomes. They were not manually repaired, silently relabeled, or regenerated until they passed.

## 16.5 The Feasibility Failure Does Not Identify Its Cause

The supported result is about the **combined frozen protocol**:

```text
scenario design
+ generator behavior
+ two-stage generation procedure
+ matched-pair constraints
+ frozen semantic rubrics
+ blind Semantic QA
+ deterministic adjudication / curation
        ↓
required 90-realization set not completed
```

The experiment does not support assigning the failure to GPT-5.6 Luna, to the Judge, to edge representation, or to any single mechanism. A future generator-methodology experiment could hold scenarios and QA fixed while comparing generator models or whole-trajectory versus sequential realization pipelines.

# 17. Relationship to the Research Program

Across the two experiments:

```text
Beta v0.1
    → ML experiment completed
    → held-out limitations observed
    → post-Test measurement hypothesis formed

Beta v0.2
    → follow-up contract frozen
    → synthetic dataset generated
    → blind semantic validation performed
    → regeneration budget exhausted
    → dataset feasibility gate not passed
    → edge ML experiment not run
```

The combined lesson is not that one specific representation has already been validated.

It is that longitudinal governance measurement requires stronger alignment between:

```text
Construct
    ↓
Observable Evidence
    ↓
Unit of Analysis
    ↓
Representation
    ↓
Dataset Validity
    ↓
Model
```

See [RESEARCH_FINDINGS.md](./RESEARCH_FINDINGS.md) for the cross-version synthesis.

---

# 18. Final Status

**Beta v0.2 dataset feasibility experiment: CLOSED.**

| Stage | Result |
|---|---|
| Required initial realizations | 90 |
| Initial ACCEPT | 71 |
| Initial REJECT | 19 |
| Initial REVIEW | 0 |
| Fresh regeneration attempts executed | 51 |
| Rejected requirements successfully replaced | 5 |
| `scenario_generation_failure` | 14 |
| Replacement set complete | No |
| Dataset feasibility gate | **NOT PASSED** |
| Edge representation ML experiment | **NOT RUN** |
| Edge-representation hypothesis | **UNTESTED** |

The experiment stopped at its pre-defined semantic-feasibility boundary rather than continuing with a post-hoc modified dataset.
