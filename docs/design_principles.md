# Design Principles

This document explains
> What assumptions shaped this system in this way?

## Principles

### Principle 1: Runtime Governance over Model Modification

Reason:

```text
Current frontier models already incorporate substantial safety mechanisms and are increasingly capable of handling many single-turn safety risks. Model capability is the advantage to take while the observability, traceability, and explainability need to be reinforced. 
```

Decision:

```text
The goal is not to change model capability.

The goal is to regulate interaction behavior.
```

### Principle 2: Observe Before Constrain

Reason:

```text
In social systems, human beliefs and behaviors are often shaped through repeated interactions over time. AI systems may similarly influence human perceptions, emotional regulation, and decision-making through prolonged engagement. A single output might be safe, while long term interaction might drift to undesirable directions. 
```

Decision:

```text
A single turn may not indicate a harmful interaction pattern.

The system should observe interaction trajectories before applying constraints.
```

### Principle 3: Constraints Should Be Reversible

Reason:

```text
Human emotional states and beliefs are dynamic rather than static. A temporary high-risk signal should not automatically be treated as a permanent characteristic of the individual.
```

Decision:

```text
Runtime controls should not persist indefinitely.

The system should release constraints when recovery signals appear.
```

### Principle 4: Human Agency Preservation

Reason:

```text
As AI becomes increasingly accessible across domains such as coding, information retrieval, and emotional support, individuals may begin relying on AI for a growing number of decisions and daily activities.

Without sufficient critical thinking and reflection skills, excessive reliance on AI may weaken independent judgment and reduce opportunities for real-world engagement.
```

Decision:

```text
AI should not become the sole source of emotional regulation or support. AI should be an "useful tool" for task assistance and help maintaining human agency in real-world relationship building and decision making.
```

### Principle 5: Explicit Governance

Reason:

```text
AI safety is a multi-layer challenge involving policy, systems, and models.

Policy frameworks define desired behavior. Model developers build safety mechanisms into foundation models. System designers determine how models are deployed and governed in real-world applications.

While foundation models increasingly incorporate safety capabilities, many governance decisions remain difficult to observe from the outside. System-level governance provides an additional layer of observability and accountability during deployment.
```

Decision:

```text
Governance decisions should be observable and explainable.

A system-level governance layer helps make runtime decisions more transparent during human-AI interactions.
```
