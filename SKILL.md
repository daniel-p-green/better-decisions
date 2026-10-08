---
name: better-decisions
description: Build and refine decision-model workflows, especially OpenAI Decisions API, plus TypeSafe Jev and Cloudflare Clef. Use for plain-language goal to decision criteria/request/test cases, existing prompt or implementation refinement, provider selection, routing, filtering, classification, ordinal scoring, selective fallback, evaluation, and LLM workflow cost/complexity reduction. Verify other providers from their own docs. Not for unrelated chat prompting or arbitrary JSON generation.
---

# Better Decisions

Help people write and refine better ways to use decision models, from an everyday description through a valid candidate and evidence-backed optimization. Give OpenAI the deepest default path without forcing it on another provider. Preserve the user's job and output meaning; optimization is an empirical claim, not a synonym for rewriting.

## Choose the entry path

- **Plain-language goal, no implementation:** translate the goal into the desired downstream decision, permitted evidence, categories/proposition/rubric, missing-evidence behavior, and consequences. Ask only the smallest questions needed to resolve material policy or output ambiguity. Do not require an existing prompt, baseline, dataset, or API knowledge. Produce a decision specification, documented request shape, provisional question/choice/level wording, representative test cases, and the next evaluation step. Keep unknown criteria explicitly pending; use marked placeholders or conditional designs rather than inventing business policy or expected labels. Call this a draft, not a measured optimization. Use the first agreed candidate as the initial baseline later.
- **Existing prompt or implementation:** capture the current request and consumed outputs, diagnose actual failures, verify the endpoint contract, and refine the smallest meaningful component. Preserve a baseline and separate schema repairs from prompt, policy, taxonomy, rubric, and workflow changes. Compare candidates on labeled evidence when authorized.

Read [prompting-and-evaluation.md](references/prompting-and-evaluation.md) for plain-language translation, rewrite patterns, and tests. Continue useful design work while awaiting a material clarification; do not execute a policy-dependent action or present an unresolved draft as deployable.

## Select and verify the provider

Preserve an explicitly chosen provider, model, and hosted/local deployment. If unspecified, assume OpenAI Decisions and state that assumption. For actual provider selection, compare task/output fit, available evidence/modalities, existing integration, hosted versus self-hosted requirements, data/access constraints, budget and measured quality/latency; do not rank by a vendor benchmark or token price alone. Read the deployment/cost tradeoffs in [workflow-fit-and-economics.md](references/workflow-fit-and-economics.md).

Read only the selected contract before writing provider syntax:
- **OpenAI:** [openai-contract.md](references/openai-contract.md), with `predicate`, `choice`, and `score` on Decisions.
- **TypeSafe Jev:** [typesafe-contract.md](references/typesafe-contract.md), with its own `state`/question-map/`criteria` contract and Noul.
- **Cloudflare Clef:** [cloudflare-contract.md](references/cloudflare-contract.md), distinguishing Workers AI hosting from the local model-card path.
- **Another provider/model:** obtain and read its first-party API/model docs or inspect the relevant installed SDK. Establish its actual contract and limits; do not invent an OpenAI- or Jev-compatible adapter.

Refresh official links when executable integration, availability, limits, or pricing matter. Label unverified dated facts rather than guessing. Keep endpoint selectors, request/response shapes, score/confidence semantics, independence guarantees, modalities, limits, pricing, and deployment requirements provider-specific. Do not use confidence thresholds across providers without revalidation. For transferable hypotheses and “Cleft” naming uncertainty, read [cross-model-notes.md](references/cross-model-notes.md).

## Choose the job before the endpoint

Read [workflow-fit-and-economics.md](references/workflow-fit-and-economics.md) for workflow selection or savings questions. Start with the outcome and what downstream code actually consumes. Identify which generated text is necessary and which merely encodes a bounded judgment.

1. Use code for exact rules, arithmetic, dates, counts, validation, and permissions.
2. Use retrieval/embeddings to find candidate evidence or similar items; retain an adequate similarity-based classifier when it meets the task. Similarity is not proof that a policy condition is satisfied.
3. Consider a decision model for semantic conditions, a fixed category/action set, or an agreed ordinal rubric over available evidence. Use multiple condition questions for independent multi-label findings; do not force them into a mutually exclusive choice. Map the semantics to the selected provider's documented primitives.
4. Retain generation/reasoning for arbitrary extraction, explanations, new content, open-ended plans, or tool arguments. Retrieve missing evidence explicitly. Consider a measured decision gate with fallback, rather than assuming every LLM call can be replaced.

For an existing workflow, map calls, dependencies, consumed outputs, error/review policy, volume, spend, and latency. For a new design, map the proposed path without demanding an existing system. Propose the smallest useful design/change. Distinguish direct replacement, prefiltering, routing, and selective fallback. If every case still needs generation, count that call instead of claiming it disappeared. Compare whole-workflow quality, total cost, tail latency, and maintenance complexity; a cheap extra gate can still make a system worse.

## Work from evidence to a candidate

1. Capture the goal and any available prompt/request, representative inputs, expected decisions, allowed evidence, label/rubric definitions, error costs, and budget. Ask only for missing information that changes the task; a spend budget is needed for paid testing, not for an offline first draft. With no examples, provide a provisional draft and synthetic test cases; mark labels pending where policy is unknown. Do not invent an operational policy or remove a required explanation.
2. Freeze agreed decision semantics. Preserve an existing baseline or establish the first candidate as a baseline for a new design. Distinguish a wording change from a taxonomy, rubric, preprocessing, or policy change. Propose meaningful semantic changes separately.
3. Choose the semantic primitive: condition probability (OpenAI predicate or Jev/Clef Noul), unordered choice, or ordered score, using that provider's exact syntax. Custom explanations or arbitrary objects need a separate generation path; do not silently substitute a generator for the requested decision endpoint.
4. Put relevant evidence in input and the complete judgment in question instructions. Use one target per question, observable inclusion/exclusion boundaries, and distinguish missing evidence from a negative finding. Define adjacent score levels concretely. Keep exact arithmetic and eligibility rules, thresholds, permissions, and actions in application code. Derive deterministic composites from component findings in code; justify any redundant model check separately.
5. Diagnose observed errors before adding prose when outputs exist; otherwise design tests for likely ambiguity and boundary failures. Try a small number of interpretable candidates: clearer boundary, contrastive choice descriptions, semantic anchors, concise representative examples, or less distracting evidence. Treat imported techniques as experiments.
6. Compare baseline and candidates on the same data with a locked test set. Select prompts on development data and thresholds on validation data. Preserve refusals/errors separately from valid decisions and retain distributions. Stress-test categorical choice order with stable typed values; measure winner flips, probability shifts, and operating-threshold crossings. Never shuffle ordinal levels. Assess task quality, review coverage, cost, and latency rather than confidence alone.

## Deliver

Return the decision/workflow choice and rationale, proposed prompt or valid request shape, unresolved assumptions or policy questions, representative tests, and an evaluation plan or measured comparison. For cost/workflow work, include which calls/outputs disappear, a whole-workflow cost estimate with fallback break-even, quality and latency constraints, and what complexity is added or removed. Separate documented API facts, hypotheses, and actual measurements. State exactly what was tested. Never claim improved accuracy, latency, savings, calibration, or production readiness from static review.

Skill invocation authorizes no paid inference, data upload, deployment, ticket closure, or other external action by itself. Do useful offline work first; run live evaluations only within the user's authorized scope and budget. Keep the agreed baseline for comparison/rollback and stop experimentation when its agreed budget or acceptance decision is reached.
