# TypeSafe Jev provider contract

Verified against first-party documentation on **2026-10-08**. This is a dated integration reference, not a promise that aliases, limits, prices, schemas, or behavior will remain unchanged. No inference request was executed during verification.

## Refresh before executable use

1. Reopen the [documentation index](https://docs.typesafe.ai/llms.txt), then the current [HTTP API](https://docs.typesafe.ai/api), [models](https://docs.typesafe.ai/models), primitive pages, [confidence](https://docs.typesafe.ai/confidence), and the selected SDK's request/response schemas.
2. Confirm endpoint, accepted shapes, resolved model, context/rate limits, error handling, and any SDK version differences. Resolve the discrepancies below before relying on their wider shapes. Read pages themselves; do not treat snippets or this snapshot as current evidence.
3. With separately authorized account access, model discovery is `GET https://api.typesafe.ai/v1/models`; discovery currently lists aliases, while explicit version IDs can still be accepted. Do not infer version availability solely from absence in that list. [Model listing](https://docs.typesafe.ai/models#listing-models)
4. Pin and log the actual model, evaluate representative labeled cases, and retune thresholds after model, options, levels, or rubric changes. Documentation refresh does not authorize credentials, data transmission, live calls, installation, or charges.

## Documented transport and conservative request

`POST https://api.typesafe.ai/v1/systemone`, with `Authorization: Bearer <API_KEY>` and `Content-Type: application/json`. The HTTP reference requires `state`, `model`, and `questions`; `questions` is an ID-to-question map. Its reference requires each question's `type` and `instructions`; Choice and Score also require `criteria`. Responses contain `model`, `answers` under the same question IDs, and `usage` with input/output token counts. [HTTP request and response](https://docs.typesafe.ai/api)

Original minimal request body, valid against the documented conservative shape; no response or quality claim is implied:

```json
{
  "model": "jev-1.13.0",
  "state": "Please cancel my appointment tomorrow.",
  "questions": {
    "requests_cancellation": {
      "type": "noul",
      "instructions": "Does the message explicitly request cancellation of an appointment?"
    }
  }
}
```

The Noul answer shape is `{"type":"noul","noul":0.0}` where the example number illustrates structure only. [Noul response](https://docs.typesafe.ai/primitives/noul#response-structure)

## Model and input snapshot

- Current stable version: `jev-1.13.0`. Both `jev-latest` and `jev-preview` currently resolve to it; there is no separate preview build. Aliases move, and the response reports the version that answered. Pin the version after tuning thresholds; upgrade on your own validation schedule.
- Verified published context budgets: **64k tokens for state plus all questions**, and **32k for state plus the longest single question**. Both constraints apply.
- Published rate limits: **100K tokens/second and 80 requests/second**. The provider explicitly says limits can change without notice and higher plan limits exist; these are a dated snapshot, not an account entitlement or permanent guarantee.
- Jev accepts text, including JSON objects/arrays interpreted as textual structured data. It does not accept image, audio, video, or binary input directly. English is its strongest language; validate non-English workloads separately. [Models](https://docs.typesafe.ai/models)

The conservative top-level state shape is a non-null **string, object, or array**. Related conversation, record, and policy fields can coexist in one state. Official object examples contain nested numbers as well as text; “text only” does not mean every nested value must be a string. Prefer named fields; put content/facts in state and judgments in questions. [State](https://docs.typesafe.ai/concepts/state)

## Typed questions and outputs

### Choice

`type: "choice"`; `criteria` maps option-name strings to descriptions. Descriptions can be strings, objects, arrays, or `null` when labels need no elaboration. Maximum: **255 options**. Option names and descriptions enter inference; the question ID does not. Use exhaustive, distinguishable options and an explicit `other`/`none_of_the_above` when coverage is incomplete.

Answer: `type`, `choice` (highest-probability option), `probabilities` (option-to-number map, collectively summing to 1), and `confidence` in `[0,1]`. A Choice selects one closed-set outcome; it is not native multi-label output. Ask separate Nouls when several labels may apply independently. [Choice](https://docs.typesafe.ai/primitives/choice)

### Score

`type: "score"`; `criteria` is an ordered array of level descriptions, preferably low to high. The docs recommend **at least two levels** and accept **up to ten**; do not describe two as a verified hard API minimum. Levels are indexed `0..n-1`. Each description is evaluated individually; the model does not see its level number or neighboring descriptions. Describe each level fully, without “more than the previous level” or numeric-only anchors.

Answer: `type`, `score`, `legend`, `probabilities`, `confidence`. On the HTTP wire, level keys are strings such as `"0"`; Python SDK keys are integers. `score = sum(i * p_i)` lies in `[0,n-1]`, may be fractional, and is neither the winning level nor a measurement in physical/business units. Different distributions can share the same mean. Inspect the probabilities too. [Score](https://docs.typesafe.ai/primitives/score)

### Noul

`type: "noul"`; ask a yes/no question or proposition. Optional `criteria` describes `true` and `false`; descriptions can be structured. Align a high value with yes/true and make the boundary explicit. Ask one proposition per Noul and combine conditions in code.

Answer: `type` and `noul` in `[0,1]`, interpreted as probability of yes/true. Near `0.5` is uncertainty, **not medium intensity**. There is no separate returned confidence field. Thresholds depend on false-positive/false-negative costs; uncertain cases can go to review. [Noul](https://docs.typesafe.ai/primitives/noul)

### Structured descriptions and documented discrepancies

Instructions and rubric descriptions can use objects/arrays for labeled context, definitions, exclusions, and examples. The [advanced structure page](https://docs.typesafe.ai/primitives/advanced) additionally advertises `null` for instructions and every description type, but sources differ:

- HTTP says instructions are required/non-null; [Python question schemas](https://docs.typesafe.ai/sdk/python/api/types/questions) and [JavaScript Noul](https://docs.typesafe.ai/sdk/javascript/api/interfaces/NoulQuestion) allow omitted/null instructions.
- HTTP/state guidance excludes top-level null; [JavaScript request payload](https://docs.typesafe.ai/sdk/javascript/api/interfaces/SystemOneRequestPayload) includes it; Python expressly forbids `state=None` while allowing nested null values.
- Advanced allows null Score levels, but Python's Score item schema accepts string/object/array, excluding null.
- HTTP lists `legend` values as strings, while the Score page demonstrates objects and [Python response schemas](https://docs.typesafe.ai/sdk/python/api/types/responses) support string/object/array descriptions. SDK usage counters also permit absent/null counts; do not turn unknown usage into zero.

**Integration advice:** use meaningful non-null instructions, non-null state, and non-null Score descriptions as the safe shared subset. Preserve structured legend values instead of assuming all are strings. Verify your chosen SDK/version and current service behavior before widening acceptance; these differences are unresolved documentation boundaries, not tested server capabilities.

## Independence, batching, and dependencies

One request evaluates one state. All questions see that shared state and run independently; an answer is not hidden context for another question. Question IDs are bookkeeping only, so put the full judgment in instructions. Explicit backticked paths such as `ticket.messages[0].text` identify relevant state fields.

Batch independent questions using that state, including speculative questions your code may ignore. Genuine dependencies need a second request only when the first answer is required to fetch/build new state, construct new entities, or choose later options. Branching on an answer alone does not require serial calls when both questions could already have been asked. These are TypeSafe's documented independence/parallelism properties; they do not establish independence for another provider. [Primitives and dependencies](https://docs.typesafe.ai/primitives)

## Probability versus confidence

TypeSafe confidence is a **derived distribution statistic**, not a second prediction or a verified probability of correctness. The current documented formulas are:

- Choice: `(max(p) - 1/n) / (1 - 1/n)`.
- Score: `max(0, 1 - sum(p_i * abs(i - m)) / D)`, where `m` is a most-probable level and `D = sum(abs(i - (n-1)/2)) / n`.
- Noul returns no confidence; the docs offer `abs(2*p - 1)` as an optional application-derived measure.

Uniform distributions yield zero and concentrated ones yield one; Score penalizes mass far from its modal level more than nearby mass. Choice confidence depends on the number of options, so do not reuse thresholds blindly after changing that set. Confidence 1 is not a correctness guarantee. Calibrate action/review thresholds on held-out task data and consequences, rather than maximizing confidence alone. [Confidence formulas and threshold guidance](https://docs.typesafe.ai/confidence)

## Errors, unknowns, and nonportable boundaries

HTTP documents `401` (authentication), `422` (validation; body identifies the field), `429` (rate limit), and `529` (overload), with JSON error bodies. Back off exponentially for `429`/`529`. [HTTP errors](https://docs.typesafe.ai/api#errors) SDKs provide broader HTTP, connection, and timeout exceptions and expose request IDs; their error body may also be text or empty. Honor retry headers and configure a bounded retry/time budget instead of assuming every failure is retryable. [Exceptions](https://docs.typesafe.ai/sdk/python/api/exceptions), [retry controls](https://docs.typesafe.ai/sdk/python/api/retries)

Unknown from reviewed sources: maximum question count apart from token budgets, choice minimum cardinality, tie-breaking, exhaustive error-body schema, a dedicated refusal envelope, streaming semantics, and universal accuracy/calibration guarantees. Do not invent defaults or fields. An `unknown`/`other` outcome must be designed into the available options or handled by review logic; it is not a documented implicit refusal.

Jev is a decision API, not a chat/completion engine; it does not generate prose, execute tools, browse, or write code. No OpenAI chat/Responses endpoint compatibility, message-role mapping, JSON-Schema conversion, or shared confidence meaning is established by this contract. Build explicit provider adapters and preserve raw responses. [Coding-agent boundaries](https://docs.typesafe.ai/introduction/coding-agents)

Provider-specific cautions for `jev-1.13` (official page reviewed **2026-10-02**): literal wording, indirection, arithmetic/counting/date comparisons, irrelevant long context, adversarial state, contradictory criteria, and option-order bias can cause errors. Keep arithmetic in code; filter context; state exact conditions; align instructions with rubrics; reorder options during evaluation; test hostile/boundary inputs. Do not interpolate Score outputs into exact numeric magnitudes or treat a confidence gate as a security boundary. [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13)
