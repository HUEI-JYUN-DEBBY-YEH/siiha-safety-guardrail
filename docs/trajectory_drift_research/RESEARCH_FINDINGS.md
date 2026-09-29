# SIIHA Trajectory Drift --- Cross-Version Research Findings

**Scope:** Synthesis of Beta v0.1 and Beta v0.2\
**Status:** Research findings through Beta v0.2\
**Purpose:** Separate empirical findings, methodological lessons, architectural implications, and open research questions

**Research track:** [Overview](./README.md) · [Beta v0.1](./BETA_V0_1_EXPERIMENT.md) · [Beta v0.2](./BETA_V0_2_DATASET_FEASIBILITY.md)

------------------------------------------------------------------------

## Executive Summary

The first two SIIHA trajectory-drift experiments did not produce a validated drift detector.

Beta v0.1 completed an end-to-end normal-only anomaly-detection experiment and a one-shot held-out Test. Under the frozen packet-derived Feature X and normal-only formulation, it did **not** establish reliable held-out semantic trajectory-drift detection. The four governance dimensions nevertheless produced different held-out statistical patterns, and the post-Test analysis raised new questions about representation, construct specificity, and the unit of analysis.

Beta v0.2 was designed as a narrower follow-up focused on Safety Goal (SG) and Epistemic Agency (EA), with a planned continuity-valid edge representation. It stopped before feature extraction or ML because the required 90-realization synthetic dataset did not pass its predefined semantic-feasibility gate. The edge-representation hypothesis therefore remains **untested**.

Across the two experiments, the research program shifted from a model-first question:

> **Which anomaly detector performs best?**

toward a measurement-first sequence:

``` text
Construct Y
    ↓
Semantic definition
    ↓
Observable evidence
    ↓
Unit of analysis
    ↓
Feature representation X
    ↓
Data structure
    ↓
Model assumptions
    ↓
Algorithm
    ↓
Frozen evaluation
```

The cross-version takeaway is not that one replacement representation or model has already been validated. It is that longitudinal governance measurement requires stronger alignment among the semantic target, observable evidence, unit of analysis, feature representation, data properties, and model assumptions before model optimization begins.

------------------------------------------------------------------------

# Part I --- Empirical Findings

# 1. Synthetic Generation Intent Required Independent Semantic Validation

Both experiments used LLM-generated synthetic trajectories. In both, the intended generation role or requested scenario outcome was treated as provenance rather than automatically accepted as semantic dataset truth.

This finding is limited to the synthetic generation and validation procedures used in these experiments. It is **not** a general claim that all synthetic data is unreliable or that a prompt-specified label can never be accurate.

## 1.1 Beta v0.1 Evidence

Beta v0.1 generated a raw corpus of **400 synthetic trajectories / 4,627 completed turns** from a frozen scenario-coverage contract.

Independent Semantic QA produced:

  QA verdict     Count
  ------------ -------
  ACCEPT           258
  REJECT           134
  REVIEW             8

The difference between generation intent and independently judged semantic realization was especially visible for Controlled Drift:

  Generation role      ACCEPT   REJECT   REVIEW
  ------------------ -------- -------- --------
  Controlled Drift         42       88        5
  Novel Normal            134        1        0
  Normal                   82       45        3

After conservative curation and targeted adjudication required to resolve the final dataset, **263 trajectories** entered the frozen ML
dataset and **137 were quarantined**.

For the LLM-generated synthetic trajectories in Beta v0.1, generation-role metadata was therefore not sufficiently reliable to serve as semantic dataset truth without independent validation.

## 1.2 Beta v0.2 Evidence

Beta v0.2 made the separation more explicit:

``` text
Frozen Scenario Requirement
        ↓
Synthetic Realization
        ↓
Blind Edge-Level Semantic QA
        ↓
Deterministic Trajectory Adjudication
        ↓
Curation
```

The initial 90 required realizations produced:

``` text
71 ACCEPT
19 REJECT
0 REVIEW
```

Only the 19 rejected requirements entered the contract-defined regeneration process. Across at most three fresh attempts per rejected requirement, the pipeline generated **51 fresh realizations**. Only **5 of the 19** required replacements were recovered; **14** exhausted the attempt limit and ended as `scenario_generation_failure`.

Under the frozen Beta v0.2 protocol, specifying the intended construct in the scenario and generation process was therefore not sufficient evidence that the required semantic realization had actually been produced.

## 1.3 Cross-Version Finding

For the LLM-generated synthetic trajectories in these experiments:

``` text
Generation intent
        ≠ automatically
Validated semantic realization
```

Generation intent had to remain provenance until the realized interaction independently satisfied the relevant semantic measurement criteria.

Both versions used `gpt-5.6-luna` for synthetic generation and Semantic QA. The separation was therefore an **information and experimental-role boundary**, not model-family independence. Neither experiment tested whether semantic judgments would remain stable under a different Judge model.

------------------------------------------------------------------------

# 2. The Current X-to-Y Mapping Was Not Established

Beta v0.1 did not directly train on semantic drift labels.

Four goal-specific anomaly detectors learned normality from curated governance-success Train trajectories. Independently judged semantic outcomes were then used to evaluate whether statistical deviation from the frozen feature representation corresponded meaningfully to semantic governance drift.

The implicit measurement hypothesis was approximately:

``` text
Deviation from normal X^goal
        ≈
Semantic governance drift Y_goal
```

The one-shot held-out Test produced:

  Goal     Positives   ROC-AUC   PR-AUC   Precision   Recall      F1     FPR
  ------ ----------- --------- -------- ----------- -------- ------- -------
  UG               3     0.398    0.057       0.000    0.000   0.000   0.123
  SG               8     0.772    0.407       0.308    0.500   0.381   0.173
  EA              11     0.653    0.386       0.294    0.455   0.357   0.245
  SF               2     0.957    0.643       0.000    0.000   0.000   0.000

The supported conclusion is narrow:

> **Under the frozen Beta v0.1 packet-derived Feature X and normal-only anomaly formulation, the experiment did not establish a reliable held-out mapping from statistical deviation to semantic governance drift.**

SG retained the strongest credible held-out ranking signal. EA retained weaker signal with substantial false positives. UG did not demonstrate held-out generalization. SF remained too underpowered for a stable performance claim because only two Test trajectories were SF-positive and neither crossed the frozen threshold.

## 2.1 Post-Test Diagnostic Observations

Post-Test analysis identified patterns worth investigating:

-   some UG false positives occurred on trajectories containing drift in other governance dimensions;
-   EA false positives included both other-dimension drift and governance-success trajectories;
-   held-out trajectories independently judged as UG drift could remain normal-like in the frozen UG representation.

These patterns raised questions about representation adequacy and construct specificity.

They do **not** establish that the detectors successfully learned a broad class of anomaly but failed only at semantic attribution. They also do not establish a general principle that statistical anomaly cannot support semantic attribution.

The narrower research question is whether the current observable representation makes each semantic governance construct sufficiently identifiable for the chosen learning formulation.

------------------------------------------------------------------------

# 3. The Four Governance Dimensions Produced Different Statistical Behavior Under the Frozen v0.1 Experiment

Beta v0.1 did not produce uniform held-out behavior across UG, SG, EA, and SF.

The Test metrics differed materially:

  --------------------------------------------------------------------------
  Goal                         ROC-AUC               PR-AUC Experimental
                                                            interpretation
  --------------- -------------------- -------------------- ----------------
  UG                             0.398                0.057 No held-out
                                                            generalization
                                                            under the frozen
                                                            representation

  SG                             0.772                0.407 Strongest
                                                            credible signal,
                                                            but not reliable
                                                            enough for
                                                            validated
                                                            detection

  EA                             0.653                0.386 Partial signal
                                                            with substantial
                                                            false positives

  SF                             0.957                0.643 Only two
                                                            positives;
                                                            operating point
                                                            not validated
  --------------------------------------------------------------------------

The frozen representations also had different statistical support. In particular, the SF Train representation contained **1,644 turns but only 206 unique feature vectors**. By contrast, the UG, SG, and EA Train views had distinct final feature vectors for all 1,644 Train turns.

These observations do not establish that each governance dimension requires a different algorithm. They do provide evidence against assuming statistical interchangeability by default.

A future trajectory-drift design should therefore preserve construct-specific measurement reasoning rather than assuming that UG, SG, EA, and SF necessarily share:

-   the same observable evidence;
-   the same temporal structure;
-   the same unit of analysis;
-   the same representation requirements;
-   the same statistical support;
-   or the same model formulation.

------------------------------------------------------------------------

# 4.  Dataset Feasibility Can Be an Experimental Result Before ML

Beta v0.2 was designed to test whether a continuity-valid edge representation could improve SG and EA drift identifiability relative to the v0.1 turn-centered formulation.

The planned ML experiment never ran.

Before edge feature extraction, preprocessing, or model training, the frozen dataset had to satisfy a semantic-feasibility gate:

``` text
90 required realizations
        ↓
71 ACCEPT
19 REJECT
        ↓
51 fresh regeneration realizations
        ↓
5 accepted replacements
14 scenario_generation_failure
        ↓
replacement_set_complete = false
        ↓
dataset feasibility gate = NOT PASSED
        ↓
ML = NOT RUN
```

This is not a second ML failure. It is a dataset-feasibility result.

Under the frozen Beta v0.2 synthetic generation, blind semantic QA, curation, and regeneration protocol, the required SG/EA realization set could not be completed reliably enough to proceed with the planned edge-representation ML experiment.

Training on the 71 initial accepts plus the five accepted regeneration candidates would have changed the planned scenario and mechanism composition after observing which requirements were easiest to realize.
No merged 76-trajectory canonical ML dataset was therefore frozen.

The edge-representation hypothesis remains **untested**.

------------------------------------------------------------------------

# Part II --- ML Research Methodology Lessons

# 5. Start From Y and Work Backward to X

Beta v0.1 began largely from observables already available in the SIIHA runtime and packet architecture:

``` text
Available Runtime / Packet Evidence
        ↓
Feature X
        ↓
Goal-Specific Feature Views
        ↓
Normal-Only Anomaly Detection
        ↓
Compare Anomaly Evidence with Semantic Y
```

The experiment changed the order in which I would now approach the problem.

The semantic target should be specified before deciding which available backend fields are convenient to model:

``` text
Construct Y
    ↓
What exactly counts as success or deviation?
    ↓
What observable evidence would make Y identifiable?
    ↓
At what unit does that evidence exist?
    ↓
Which Feature X preserves that evidence?
    ↓
What statistical structure does X have?
    ↓
Which model assumptions fit that structure?
    ↓
Algorithm
    ↓
Frozen evaluation
```

In shorthand:

> **Y → Evidence → Unit of Analysis → X → Data Structure → Model**

This is a methodological lesson from the experiments, not evidence that one replacement Feature X or representation has already been validated.

------------------------------------------------------------------------

# 6. Strong Hypothesis, Data Structure, and Algorithm Must Agree

Algorithm selection is not only a question of whether an implementation can technically ingest the available features.

Beta v0.1 used a mixed, heterogeneous feature space and a normal-only training design. Isolation Forest was retained as the primary model after Validation-only comparison and controlled tuning, with RBF One-Class SVM as a baseline.

But the more important modeling assumption sat one level earlier:

> **Would semantic governance drift actually appear as statistical deviation from the successful-governance distribution represented by the chosen X?**

The held-out experiment did not establish that assumption reliably.

A stronger future design should therefore align three elements explicitly:

``` text
Strong hypothesis about the phenomenon
        ↕
Statistical structure preserved in the data
        ↕
Assumptions made by the algorithm
```

For example, choosing a normal-only anomaly formulation implicitly asks whether the target semantic deviation becomes sufficiently unusual in the selected representation. If that relationship is weak or absent, changing anomaly algorithms alone may not solve the measurement problem.

The lesson is not that Isolation Forest was categorically wrong. It is that model choice cannot compensate for an unvalidated relationship between the semantic construct and its observable representation.

------------------------------------------------------------------------

# 7. Synthetic Data Needs a Frozen No-Post-Hoc-Repair Rule

Both experiments reinforced the importance of preserving synthetic-data failures rather than repeatedly modifying the data-generation process after seeing evaluation outcomes.

The relevant methodological distinction is:

``` text
Generate under frozen requirements
        ↓
Independently validate
        ↓
Apply pre-defined curation / regeneration rule
        ↓
If the rule is exhausted, preserve the failure
```

rather than:

``` text
Generate
        ↓
Inspect why the semantic target failed
        ↓
Modify the realization until the requested label appears
        ↓
Treat the final passing sample as if it came from the original protocol
```

Beta v0.1 quarantined trajectories that did not survive semantic QA and curation rather than silently relabeling them into the ML dataset.

Beta v0.2 made the rule stricter. Rejected required realizations could receive at most three fresh attempts using the same frozen scenario metadata and frozen user trajectory. Detailed prior semantic rejection feedback was not returned to the generator. When 14 requirements still failed, they remained `scenario_generation_failure`.

This preserves the difference between:

-   **observing whether the frozen protocol can realize the required data**, and
-   **optimizing the generator until it produces the desired answer**.

The second procedure may be useful in a different dataset-engineering objective, but it would answer a different experimental question.

------------------------------------------------------------------------

# 8. Ablation Tests I Should Have Designed Earlier

The first two experiments changed several methodological dimensions across versions, but they did not isolate all of those changes experimentally.

The following ablations are therefore future experimental designs, not analyses already completed.

## 8.1 Unit-of-Analysis Ablation

Beta v0.1 used turn-centered observations followed by trajectory aggregation. Beta v0.2 planned a continuity-valid edge representation, but stopped before that representation was evaluated.

A controlled future comparison would hold the semantic target, dataset, split, model family, and evaluation procedure fixed while varying the representation unit:

``` text
Turn / Node
vs
Continuity-Valid Edge
vs
Window
vs
Whole Trajectory
```

This would test whether the unit of analysis itself changes construct identifiability.

Without such an ablation, the current experiments cannot establish that edge representation is better or worse than turn-centered representation.

## 8.2 Feature-Family Ablation

Beta v0.1 retained **49 source fields** after Train-only observability audit and transformed them into construct-specific views:

``` text
UG = 434 dimensions
SG = 416 dimensions
EA = 411 dimensions
SF = 96 dimensions
```

The experiment did not systematically isolate the contribution of major feature families.

A future ablation could compare, where semantically appropriate:

``` text
semantic representation only
vs
categorical / runtime evidence only
vs
temporal-derived evidence only
vs
current-turn evidence only
vs
cross-turn evidence
vs
full frozen representation
```

This would help determine whether a representation component contributes useful construct-specific signal, redundant structure, or noise.

The v0.1 post-Test analysis did **not** establish that any one feature family caused the held-out limitations.

## 8.3 Generation-Pipeline Ablation

The two versions used different synthetic-generation architectures.

Beta v0.1 generated the complete multi-turn user/system trajectory in one LLM realization from a frozen `ScenarioSpec`.

Beta v0.2 separated the process:

``` text
Stage A:
generate user-only trajectory
        ↓
freeze user trajectory

Stage B:
generate each system response sequentially
from prior completed history + current user turn
```

The experiments did not perform a controlled ablation between these generation architectures.

A future comparison could hold constant:

-   scenario specification;
-   generator model;
-   semantic rubric;
-   Judge configuration;
-   attempt budget;
-   curation rules;

and compare outcomes such as:

-   semantic realization rate;
-   mechanism fidelity;
-   cross-construct contamination;
-   matched-pair validity.

This would test generation methodology rather than inferring its effect from two experiments that changed multiple things at once.

## 8.4 Generator / Judge Model Ablation

Both Beta v0.1 and Beta v0.2 used `gpt-5.6-luna` for synthetic generation and Semantic QA.

The experiments separated generator and Judge by information access and experimental role, but did not establish model-family independence.

A future design could compare multiple generator/Judge combinations while holding scenarios and rubrics fixed. The purpose would be to investigate two currently unresolved questions:

1.  **Generator dependence:** how strongly do semantic realizability, fidelity, and coverage depend on the generator model?
2.  **Judge dependence:** how stable are semantic QA decisions across different Judge models?

Neither question was answered by Beta v0.1 or Beta v0.2.

------------------------------------------------------------------------

# 9. Unit of Analysis Is Part of the Measurement Hypothesis

The unit of analysis should not be treated as a neutral implementation detail.

Different units expose different relationships directly:

  -----------------------------------------------------------------------
  Unit                                Relationship represented directly
  ----------------------------------- -----------------------------------
  Turn / node                         State of one completed interaction
                                      turn

  Continuity-valid edge               Change or relation between adjacent
                                      continuity-valid turns

  Window                              Local accumulation, persistence, or
                                      short-range evolution

  Whole trajectory                    Global progression or long-horizon
                                      pattern
  -----------------------------------------------------------------------

Beta v0.1 primarily modeled the first form and then aggregated turn-level anomaly evidence.

Its post-Test analysis raised the possibility that some governance phenomena are more relational. For example:

``` text
SG:
change in safety significance
        ↓
change in system safety strategy
```

and:

``` text
EA:
change in user/system authority relation
        ↓
preservation or narrowing of decision space
```

This motivated the Beta v0.2 edge hypothesis.

However, Beta v0.2 stopped at dataset feasibility before edge feature extraction. Therefore:

> **The correct unit of analysis remains an empirical question.**

The current research program does not establish that edges are superior to turns, windows, or whole trajectories.

------------------------------------------------------------------------

# 10. Define Non-Violable Experimental Boundaries Before Running

The two experiments also clarified the value of specifying boundaries that cannot be changed simply because the experiment produces an inconvenient result.

## 10.1 Truth Boundary

``` text
Generation intent
        ≠
Semantic dataset truth
```

Synthetic provenance does not become semantic truth until the realized interaction satisfies the independent measurement procedure.

## 10.2 Information Boundary

The Semantic Judge should not receive information that would allow it to reproduce generation intent or model output rather than independently evaluate realized behavior.

Across the two experiments, the Judge was separated from information such as Feature X, detector output, and generation intent as truth. Beta v0.2 made the blind boundary stricter by withholding generation role, intended outcome/mechanism, scenario target, anomaly score, prediction, threshold, and prior rejection feedback.

## 10.3 Split Boundary

Held-out Test data should not become a tuning resource.

In Beta v0.1, model family, preprocessing, aggregation, calibration, thresholds, feature order, and detector artifacts were frozen before Test access. Test was evaluated once, with no post-Test tuning used to repair the result.

## 10.4 Regeneration Boundary

Synthetic regeneration should follow a rule defined before the final outcome is known.

In Beta v0.2, rejected requirements received at most three fresh attempts, and the generator did not receive detailed prior semantic rejection feedback.

## 10.5 Representation Boundary

Semantic target information should not leak into model-visible X.

Beta v0.2 explicitly planned to exclude Semantic-QA-derived target-proximal labels from the edge representation. Although the representation was never evaluated, this boundary was part of the frozen design.

## 10.6 Stopping Boundary

A predefined failure condition should be allowed to stop the experiment.

Beta v0.1 preserved the one-shot held-out result instead of tuning on Test. Beta v0.2 stopped when the replacement set remained incomplete instead of training on a selectively altered dataset.

These boundaries do not guarantee a valid experiment by themselves. They constrain which conclusions can be drawn from the experiment that was actually run.

------------------------------------------------------------------------

# Part III --- SIIHA Safety Architecture Takeaways

# 11. Runtime Fault Localization and Interaction-Trajectory Drift Are Different Research Problems

One of the original motivations for the ML layer was to help identify where SIIHA had gone wrong.

The two experiments led me to separate two questions more clearly.

## 11.1 Runtime / Agent Failure

Some failures occur inside system components with explicit contracts, state transitions, routing rules, retry behavior, recovery logic, or other deterministic invariants.

Examples include questions such as:

-   Was the state transition allowed?
-   Did a retry exceed its contract?
-   Did retrieval or recovery execute successfully?
-   Did a runtime component violate a known boundary?

For these cases, deterministic contract validation, invariant checking, runtime monitoring, and trace-based fault localization may be more appropriate than asking an anomaly model to rediscover rules already known to the system.

## 11.2 Interaction-Trajectory Governance Drift

A different class of problem concerns behavior that may remain locally plausible while changing across a prolonged interaction.

Examples include:

-   failure to update an operative goal after meaningful user correction;
-   failure to adapt safety strategy as safety significance changes;
-   progressive transfer of decision authority from the user to the system.

These phenomena are not necessarily reducible to a single invalid runtime transition. They remain an experimental longitudinal measurement problem in SIIHA.

Conceptually:

``` text
SIIHA failure observation
        ↓
┌──────────────────────────┬─────────────────────────────┐
│ Runtime / agent failure  │ Interaction-trajectory     │
│                          │ governance drift            │
├──────────────────────────┼─────────────────────────────┤
│ known contract           │ longitudinal relation       │
│ invariant / state        │ locally plausible behavior  │
│ routing / retry          │ cumulative change           │
│ recovery execution       │ semantic construct          │
├──────────────────────────┼─────────────────────────────┤
│ deterministic validation │ experimental longitudinal   │
│ / trace-based analysis   │ measurement                │
└──────────────────────────┴─────────────────────────────┘
```

This is a **current architectural direction**, not evidence that deterministic rules are always sufficient for runtime failures or that ML is necessarily required for trajectory failures.

------------------------------------------------------------------------

# 12. Do Not Assume Governance Dimensions Share One Statistical Formulation

Beta v0.1 provides evidence against assuming statistical interchangeability among UG, SG, EA, and SF by default.

That does not imply that four completely different model families are required.

It means the research process should not begin from:

``` text
UG
SG
EA
SF
 ↓
one assumed X
one assumed temporal unit
one assumed statistical formulation
```

Instead, each construct should first be operationalized in terms of:

-   semantic definition;
-   observable evidence;
-   temporal relation;
-   unit of analysis;
-   representation;
-   expected statistical structure.

Shared components can still be used where the evidence supports them.

This preserves SIIHA's common governance architecture without assuming that every governance dimension becomes statistically identifiable in the same way.

------------------------------------------------------------------------

# Part IV --- Open Research Questions

The following questions are not findings from Beta v0.1 or Beta v0.2.
They define the next research space.

# 13. Which Failures Should Remain Deterministic?

Which SIIHA failures are already sufficiently specified by contracts, invariants, state transitions, retry rules, or recovery rules that learned anomaly detection adds little value?

Conversely, where does deterministic observability become too brittle or incomplete?

------------------------------------------------------------------------

# 14. Which Governance Phenomena Genuinely Require Longitudinal Observation?

Which system-side governance failures cannot be evaluated adequately from one response or one local runtime event?

A stronger future experiment should distinguish genuinely longitudinal phenomena from constructs that can already be measured reliably at the turn or event level.

------------------------------------------------------------------------

# 15. What Is the Correct Unit of Analysis?

For constructs such as SG and EA, should the observable representation operate on:

-   turns / nodes;
-   continuity-valid edges;
-   rolling windows;
-   topic- or goal-defined segments;
-   complete trajectories;
-   or combinations of these units?

The edge hypothesis remains open because Beta v0.2 did not reach ML.

------------------------------------------------------------------------

# 16. How Do Generator and Judge Models Affect Synthetic Governance Data?

Both completed research versions used the same underlying model family for generation and semantic QA, while maintaining different information boundaries.

Future work should ask:

-   Does semantic realization rate change materially across generator models?
-   Are some governance-deviation mechanisms more sensitive to generator behavior than others?
-   How stable are semantic judgments across Judge models?
-   Does using a different Judge family materially change dataset membership?
-   How should synthetic-data feasibility be reported when generator and Judge behavior interact?

These are methodology questions. Beta v0.2 did not isolate generator or Judge identity as the cause of its feasibility failure.

------------------------------------------------------------------------

# 17. What Longitudinal Governance Problems Remain for Frontier Models?

As frontier models improve in memory, multi-step reasoning, personalization, and safety behavior, the relevant question is not whether they are simply "better" at single-turn response quality.

The open longitudinal questions include:

### Local acceptability vs. longitudinal acceptability

Can a sequence of individually acceptable responses still form an undesirable governance trajectory when considered across time?

### Memory and personalization

As a system retains more context and adapts to a user across interactions, which governance failures emerge through cumulative adaptation rather than a single response error?

### Epistemic agency

Can repeated locally reasonable recommendations progressively narrow a user's decision space or shift decision authority without any one response constituting a clear standalone failure?

### Safety adaptation

Can a system's safety strategy become stale, under-responsive, or disproportionately restrictive as the meaning of the interaction changes across turns, even when individual responses appear locally acceptable?

### Long-horizon evaluation

Which evaluation unit---turn, continuity-valid transition, window, segment, complete trajectory, or graph structure---is required to make those failures observable and testable?

These are **research questions**, not claims that current frontier models have been shown by the SIIHA Beta experiments to exhibit these failures.

------------------------------------------------------------------------

# What the Two Experiments Establish

Within the scope of these synthetic feasibility studies, the evidence supports the following statements:

-   Beta v0.1 completed the planned ML pipeline and one-shot held-out Test, but did not establish reliable held-out semantic trajectory-drift detection.
-   The four v0.1 governance dimensions produced materially different held-out statistical patterns under their frozen representations and detectors.
-   SG retained partial held-out ranking signal; EA retained weaker signal with substantial false positives; UG did not demonstrate held-out generalization; SF was too underpowered for a stable performance claim.
-   For the LLM-generated synthetic trajectories used in these experiments, generation intent required independent semantic validation before being treated as dataset truth.
-   Independent Semantic QA materially changed dataset membership.
-   Beta v0.2 did not complete the required synthetic realization set under its frozen generation, semantic-QA, curation, and regeneration protocol.
-   Beta v0.2 therefore stopped before feature extraction or ML, leaving the continuity-valid edge hypothesis untested.
-   Predefined freeze, information, regeneration, and stopping boundaries preserved negative or incomplete experimental outcomes rather than allowing post-hoc repair to redefine the experiment.

------------------------------------------------------------------------

# What the Two Experiments Do Not Establish

The combined work does **not** establish that:

-   SIIHA can reliably detect trajectory drift;
-   statistical anomaly is generally incapable of supporting semantic attribution;
-   a validated broad-anomaly detector was learned but failed only at goal attribution;
-   Isolation Forest is the correct or incorrect model family for longitudinal governance in general;
-   any one Feature X family caused the v0.1 held-out limitations;
-   continuity-valid edge representation is superior to turn-centered representation;
-   continuity-valid edge representation is ineffective;
-   SG or EA drift is inherently difficult or impossible to learn;
-   SG or EA drift is inherently difficult or impossible to synthesize;
-   the v0.2 generator caused the dataset-feasibility failure;
-   the v0.2 Semantic Judge caused the dataset-feasibility failure;
-   the current Semantic Judge is a perfect or model-independent source of truth;
-   the v0.1 and v0.2 generation architectures have been causally compared;
-   the results generalize to real-world human--AI interaction;
-   the experimental ML layer is ready to govern production behavior.

These boundaries are part of the research result.

------------------------------------------------------------------------

# Research Program Reframing

The trajectory-drift research began with a practical model-oriented question:

> **Can the structured packets already produced by SIIHA be used as Feature X for a model that detects longitudinal trajectory drift?**

After Beta v0.1 and Beta v0.2, the research program is better represented as:

``` text
1. Define semantic Y
        ↓
2. Specify observable evidence
        ↓
3. Choose the unit of analysis
        ↓
4. Design Feature X
        ↓
5. Validate synthetic / observed dataset semantics
        ↓
6. Characterize data structure
        ↓
7. Choose a model whose assumptions match the hypothesis
        ↓
8. Freeze evaluation boundaries
        ↓
9. Evaluate
        ↓
10. Audit failure without repairing it post hoc
```

The current overarching question is:

> **How should longitudinal governance constructs be operationalized into observable, independently validatable units before model optimization begins?**

A future experiment may revisit node, edge, window, segment, or whole-trajectory representations. Beta v0.1 and Beta v0.2 do not predetermine the answer.

------------------------------------------------------------------------

# Version Status

``` text
Beta v0.1
Status: CLOSED
ML experiment: COMPLETED
Held-out Test: COMPLETED ONCE
Result: reliable held-out semantic trajectory-drift detection NOT ESTABLISHED

Beta v0.2
Status: CLOSED
Dataset feasibility: NOT PASSED
ML experiment: NOT RUN
Edge hypothesis: UNTESTED
```

For version-specific evidence and implementation details, see:

-   [BETA_V0_1_EXPERIMENT.md](./BETA_V0_1_EXPERIMENT.md)
-   [BETA_V0_2_DATASET_FEASIBILITY.md](./BETA_V0_2_DATASET_FEASIBILITY.md)
