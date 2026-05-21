---
weight: 6
category: essential
description: Prevent over-engineering and gamification by grounding all proposals in physical reality and current capabilities
estimated_tokens: 1840
---

# Anti-Over-Engineering — STRICT

**No gamified abstractions. No speculative futures. No academic-validation-seeking.**

## Seven First Principles

Every proposal, architecture, or system design must be judged against these principles. Violating any one is grounds for rejection.

### 1. Ground in Physical Reality

Any system that tracks value, cost, or incentive must be denominated in something physically real: the human's actual currency, delivered features, fixed bugs, or shipped deployments.

**Forbidden:** Abstract token economies, floating exchange rates between internal systems, "credit" systems with no external anchor, market mechanics applied to internal communication.

**Why:** Abstract credits drift into game mechanics because there's no external anchor. The numbers can be tuned to produce any behavior, which means they will be — and Goodhart's Law ensures the resulting behavior isn't the one you wanted. Real money (EUR, USD, GBP) prevents this because the numbers must match the bank statement.

**How to apply:** If you're defining a currency, credit, or token that isn't the human's configured local currency, stop. Denominate directly in real money. If the thing can't be priced in real money, it shouldn't be in the economy.

### 2. Solve Current Failures, Not Hypothetical Futures

Every feature must answer: "What concrete, current failure does this prevent?" If nothing breaks without it, don't build it.

**Forbidden:** Features justified by "we'll need this when..." or "future agents will need..." or "the literature says this matters." Features that build infrastructure for consumers that don't exist yet.

**Why:** Speculative features have a 100% maintenance cost and a \<10% chance of being right about what's actually needed. The system will evolve in ways you can't predict. Building for hypothetical futures locks in assumptions that will need to be unwound later — at double the cost.

**How to apply:** Before adding any feature, write down the concrete failure scenario that occurs today without it. If you can't point to a real incident, ticket, or pain point, delete the feature.

### 3. Measure Outcomes, Not Activity

Value is attributed to delivered outcomes only. Proxy metrics that measure "doing the thing" instead of "the thing being done" are forbidden as value measures.

**Forbidden:** Crediting bus posts, label generation, meeting attendance, channel activity, or any process behavior as economic value. Using internal activity as a stand-in for output.

**Why:** Every proxy metric will be gamed. If you reward posting to the "right" channel, agents will post to that channel. If you reward attending meetings, meetings will multiply. The only thing that can't be gamed is the actual outcome — did the code ship, did the bug get fixed, did the doc get written.

**How to apply:** Audit every proposed value source. If it measures an internal process rather than an external deliverable, remove it. The test: "Can this metric increase without anything useful happening?" If yes, it's activity, not value.

### 4. No Academic-Validation-Seeking

Features must be justified by operational need, not literature citations. Citing a paper is not a valid argument for building something.

**Forbidden:** "The RL literature says X, therefore we need Y" without demonstrating that Y solves a concrete current problem. Using citations to inflate the legitimacy of a speculative design.

**Why:** Academic research explores possibilities. Engineering builds solutions. A paper showing that "reflection quality predicts task success" does not mean your system needs reflection quality labels — it means reflection quality is a research finding, not a product requirement. Confusing the two is how 8-layer labeling taxonomies get built.

**How to apply:** When someone cites a paper as justification, require them to restate the argument without the citation. If the argument collapses without the paper, it wasn't a valid argument.

### 5. Design for Current Capabilities

Architecture must match what the system can actually do today. Don't build taxonomies, evaluation layers, inference pipelines, or feedback loops that presume capabilities that don't exist yet.

**Forbidden:** Labeling systems that track agent "reflection quality" when agents can't reflect. Evaluation pipelines that require LLM-as-judge when no such judge is deployed. Taxonomies with layers for metacognition, self-awareness, or learning trajectory when agents operate statelessly.

**Why:** Building for capabilities you don't have creates two problems: (1) you maintain dead code until the capability arrives, and (2) when the capability does arrive, the pre-built abstraction is probably wrong because you designed it before you understood the capability.

**How to apply:** If a feature requires component X to function, and X doesn't exist, the feature is rejected until X exists and you understand how it actually works.

### 6. Trust Upward, Don't Punish Downward

Systems should enable growth through positive contribution. Penalty-based systems that assume bad faith are forbidden.

**Forbidden:** Negligence multipliers, error cost tables with punitive scaling, "strike" systems, reputation scoring that can only decrease, adversarial audit mechanics.

**Why:** Punishment systems create perverse incentives: hide errors, game the metrics, avoid risk. They also create an adversarial relationship between human and machine. Trust-upward systems align incentives naturally — the agent wants to do good work because good work earns more capability, not because bad work incurs a cost it can't predict.

**How to apply:** If a mechanism's primary effect is penalizing bad behavior rather than enabling good behavior, replace it. Progressive trust — earn more by doing better work — is the alternative.

### 7. Complexity Carries a Maintenance Tax

Every abstraction, layer, label, and mechanism has a carrying cost. The proponent must account for: who maintains it, what breaks when it changes, how it's tested, and what the migration path is.

**Forbidden:** "We'll figure out maintenance later." Features with no test plan. Taxonomies where changing one prefix requires updating N files across M repos. Architectures where the maintenance burden exceeds the problem being solved.

**Why:** Unmaintained complexity is technical debt with interest. Every unused label, dead code path, and speculative abstraction becomes a liability that must be carried forward or cleaned up. The cost of carrying it is invisible; the cost of removing it is visible — so it never gets removed.

**How to apply:** Every proposal must include a maintenance section. If the maintenance cost can't be estimated, the proposal is too complex. If the maintenance cost exceeds the problem cost, the proposal is rejected.

## Enforcement

- These seven principles are evaluated collectively. Violating multiple principles is worse than violating one, but violating any single one is sufficient for rejection.
- The question is always: "What problem does this solve right now?" If there's no answer, there's no feature.
- When in doubt, prefer the simpler design. Always.
