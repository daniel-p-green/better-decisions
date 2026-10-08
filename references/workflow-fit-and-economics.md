# Workflow fit and economics

Checked 2026-10-08. The selection rules, formulas, and patterns below are engineering synthesis, not guaranteed API savings. Ground implementation in the selected provider contract linked from SKILL.md; OpenAI is the default assumption, not a forced migration.

## Select a provider and deployment deliberately

Start from the user's chosen provider/deployment if supplied. Otherwise state an OpenAI assumption, or compare providers when the job genuinely requires that choice. Use the same labeled workload and required outcome to assess:

- Output semantics, category/rubric capacity, modalities and input limits verified for the exact endpoint
- Existing infrastructure/account access, data-processing/location requirements, and authorized integration scope
- Hosted operational burden versus local/open-weight deployment, including hardware availability, throughput, model/license requirements, maintenance and total serving cost
- Current provider-specific billing plus retries, fallback/review, conditional workload cost and end-to-end latency
- Application quality, calibration/coverage, versioning and change risk; vendor benchmarks are evidence about those runs, not the user's task

OpenAI, TypeSafe and Cloudflare have different endpoint selectors, schemas and availability. An advertised compatible request format does not imply identical confidence formulas, question-independence behavior, modality support or operating thresholds. Use the provider-specific references before drafting each adapter. For an unfamiliar decision model, ask for its official source when needed and verify it instead of treating these three as exhaustive. Self-hosting is an option to evaluate when explicitly wanted and supported, not permission to download weights, install software, configure credentials or deploy.

## Start with the job

Ask: what must the workflow accomplish, which outputs are consumed, and what happens when it is wrong? Inventory each call's inputs, output contract, dependencies, volume, cost, latency, retries, human review, and consequential action. Preserve required outputs and policy. Treat unnecessary explanations as removable only when the user confirms they are not needed. “Bounded” describes the answer space, not a promise that every judgment is easy or reliable.

Use this routing guide:

| Job | First candidate | Boundary |
| --- | --- | --- |
| Exact lookup, status/amount/date rule, calculation, permission check | Deterministic code or database query | Do not ask a model to guess facts already available in structured data. |
| Find similar items, cluster, search a large corpus | Embeddings/retrieval | Similarity finds candidates; it does not establish truth, policy compliance, or authorization. |
| Judge one semantic condition over supplied evidence | Condition primitive: OpenAI predicate, Jev/Clef Noul | The probability concerns truth, not severity; define the proposition and check modality support. |
| Assign one category or supported route | The provider's choice primitive | Define mutually distinguishable categories and an out-of-scope/insufficient-evidence path where needed. |
| Detect several independent properties | Several condition questions sharing input | A single choice is exclusive; multi-label findings need separate judgments. |
| Prioritize on a defined ordered dimension | The provider's documented score primitive | For these providers, an expected ordinal index is not an exact measurement or custom numeric value. Verify another provider's semantics. |
| Extract arbitrary fields, write explanations/content, develop plans, or request tool calls | Generation/reasoning with the appropriate contract | OpenAI Decisions, Jev and Clef's bounded primitives do not generate those deliverables; retain the needed path. |

The [OpenAI Decisions guide](https://developers.openai.com/api/docs/guides/decisions) defines its three primitives and distinguishes them from Structured Outputs and function calling; Jev and Clef's own contracts are linked from SKILL.md. The [embeddings guide](https://developers.openai.com/api/docs/guides/embeddings) documents search, clustering and similarity-based classification. Retain an adequate embedding classifier; compare it rather than automatically adding a decision stage. Selection among these methods requires application tests; their output contracts do not establish which will be most accurate or cheapest for a particular workload.

## Useful bounded-decision patterns

- **Direct replacement:** replace a call whose only consumed output is a class, boolean judgment, or anchored ordinal rating. Return the typed decision and validate it in code. Keep generation if a required explanation or extraction remains.
- **Prefilter:** decide whether evidence is relevant or an item belongs in a processing queue before expensive generation/review. Measure missed-relevant-item cost and recall; discarded inputs will otherwise hide failures.
- **Router:** choose a known handler, department, or model path from the request plus current state. Include a general reasoning/review path. Benchmark mistaken cheap-path routing, not just correctly routed easy examples.
- **Selective fallback:** automate only the validated subset; send ambiguous, missing-evidence, out-of-domain, or failed cases to the authorized fallback. Handle refusals under the safety/review policy; never use another model path to bypass a safeguard. Compare risk and automated coverage together. High confidence alone is not a reliable routing policy.
- **Prioritizer:** score an explicit severity/relevance rubric, preserve the distribution, and let code combine dimensions or apply approved weights. Validate ranking quality and harmful under-prioritization. Retrieve a shortlist before judging a huge candidate corpus; measure recall of that shortlist too.
- **Shared-evidence bundle:** ask independent judgments in one request instead of resending the same evidence. Budget question tokens and test bundle composition. If a later question requires a new answer, retrieved evidence, or dynamically selected options, implement that dependency explicitly. Do not assume bundling leaves probabilities unchanged.

These are applications of bounded-decision contracts, not promises that every pattern saves money. [Connect voice to Decisions](https://developers.openai.com/api/docs/guides/decisions-voice) provides an OpenAI router example: choose an available slide action or route analysis to a reasoning model, validate current state before acting, and pass original evidence to the reasoning path. The pattern generalizes as a hypothesis; voice, transcription, execution, cancellation, and authorization are separate workflow responsibilities.

## Price the whole workflow

Compare equivalent outcomes over the same workload. Report API spend separately from any estimated labor, error loss, and engineering cost; use agreed assumptions or leave them as constraints. Count:

- All calls and tokens, including preprocessing/extraction, retrieval, question descriptions, retries, refusals, and conditional fallback
- Human review rate and minutes, correction/rework, and false-positive/false-negative consequences
- End-to-end p50/p95, queueing, sequential dependencies, and fallback-path latency
- Deployment/maintenance: schemas, adapters, taxonomies/rubrics, threshold validation, logs, monitoring, and rollback

The [cost optimization guide](https://developers.openai.com/api/docs/guides/cost-optimization) recommends measuring token usage and comparing smaller models on evaluations; the [latency guide](https://developers.openai.com/api/docs/guides/latency-optimization) recommends fewer serial requests and doing deterministic work outside the LLM. These tradeoffs also apply to an added decision gate. Do not apply another endpoint's caching, batching, or processing-tier discounts without verifying Decisions support.

For a gate applied to every case, let:
- `C_G`: baseline generation/API cost per case
- `C_D`: decision gate/API cost per case, including gate retries
- `q`: fraction invoking fallback, including out-of-scope, uncertain, refused, and error cases
- `C_F`: mean fallback/API cost conditional on those cases, not automatically the overall baseline average
- `Delta`: candidate minus baseline non-common costs, such as added retrieval, review, errors, and amortized maintenance, in the same cost units

Then `C_new = C_D + q * C_F + common costs + candidate other costs`. Relative to the baseline, require `C_D + q * C_F + Delta < C_G`. If `C_F > 0`, the break-even condition is `q < (C_G - C_D - Delta) / C_F`. If the numerator is nonpositive, the modeled gate cannot save money. A break-even above 1 means all feasible fallback rates meet the cost condition under those assumptions; quality and latency still constrain adoption.

Only in the simplified case `C_F = C_G` and `Delta = 0` does this become `q < 1 - C_D / C_G`. If earlier rules resolve part of the workload, weight costs by the fractions reaching each branch. If every case still needs unchanged generation (`q = 1`), an added gate normally adds API cost; count any measured reduction in that generation separately.

Illustration using invented prices, not a benchmark: `C_G = $0.004`, `C_D = $0.0002`, `C_F = $0.006`, and `Delta = 0` imply `q < 63.3%`. At 30% fallback, API cost is `$0.002` per case, half that baseline before other costs; at 80%, it is `$0.005`, 25% more. Harder fallback cases need not cost the same as easy baseline cases. Label all assumed inputs and never report this illustration as observed savings.

For a sequential gate, mean model-path latency is approximately `E[L_D] + q * E[L_F | fallback]`, before other stages. The fallback path pays for both stages. Do not add or scale p95s as if quantiles were means; measure the end-to-end distribution. OpenAI's advertised roughly 10x endpoint speedup is not a workflow-specific guarantee or a cross-provider ranking.

## Simplify deliberately

Bounded typed answers can remove prose-to-label parsing or generation used only to encode a decision. Compare against an existing Structured Outputs baseline fairly: it may already provide valid schema outputs. Keep answer validation, refusal/error handling, permissions, and action policy in code. An extra router, fallback service, provider adapter, or moving threshold can add complexity; count the components actually removed and added, then choose the smallest system meeting quality and risk requirements.

Avoid proliferating questions, levels, examples, and providers without a demonstrated need. When a composite decision follows completely from component findings and an exact policy, derive it in code. Do not automatically ask the model for both the components and the same composite; treat redundant consistency checks as an extra experiment whose benefit and cost need justification. For factual explanation templates, obtain only the findings the templates need, and preserve downstream machine values during assembly. Preserve stable machine values and keep richer semantics in descriptions. Version the decision contract separately from action policy so a clearer prompt does not silently change business rules.

## Decide using measurements

Evaluate the full baseline and candidate paths on the same representative, labeled cases. Include rare classes, missing evidence, retrieval misses, out-of-domain cases, and gate failures. Sweep policy thresholds on validation data and compare total cost versus accepted-action quality, coverage, and latency. Make a final locked test comparison and retain per-slice regressions. Do not optimize only the gate's accuracy or choose a threshold from a confidence field's appearance.

Deliver: workflow map; proposed replacement/gate and preserved outputs; explicit cost assumptions and fallback break-even; quality/latency/review constraints; complexity tradeoff; valid request/prompt; and adopt/reject/inconclusive with evidence. Without authorized inference, deliver the draft, calculation, and test plan, and state that no live quality or performance benefit has been measured.
