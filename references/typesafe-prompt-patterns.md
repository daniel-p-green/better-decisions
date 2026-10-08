# TypeSafe Jev prompt patterns

Reviewed: 2026-10-08. Discovered through the [first-party documentation index](https://docs.typesafe.ai/llms.txt).
These are original request examples and design patterns, not recorded model results or accuracy promises.
Complete bodies pin the documented `jev-1.13.0` snapshot. The `jev-latest` alias can move; refresh model availability and version-specific guidance before adopting a newer model.

## Contents
- [Choose a layout](#choose-a-layout)
- [Complete request: contrastive Choice and explicit Noul](#complete-request-contrastive-choice-and-explicit-noul)
- [Self-contained Score anchors](#self-contained-score-anchors)
- [Arrays and code-supplied records](#arrays-and-code-supplied-records)
- [Missing evidence and application policy](#missing-evidence-and-application-policy)
- [Before-and-after rewrites](#before-and-after-rewrites)
- [Cookbook patterns and checks](#cookbook-patterns-and-checks)

## Choose a layout

- Use a string for one short passage or an unambiguous question; JSON is not automatically better.
- Use an object for named facts, a conversation plus policy, paired records, or application state. Keep related evidence together.
- Use an array for an ordered sequence of records/messages; objects within it can label speaker and content.
- Put input content and supporting facts in `state`; put the requested judgment in each question's `instructions` and boundaries in `criteria`.
- Keep only relevant state. Jev accepts text and structured JSON, not image/audio/video inputs. English is its primary training language.
Sources: [State](https://docs.typesafe.ai/concepts/state), [Primitives](https://docs.typesafe.ai/primitives).

Write a complete, atomic judgment in every instruction. Question IDs are response lookup keys and are invisible to Jev. Name the exact target using literal backtick characters around dot-and-index paths, such as `ticket.messages[0].text`. Questions sharing a request see the same state independently; another answer is not hidden context for this question.
Sources: [Build guide](https://docs.typesafe.ai/concepts/how-to-build-with-system-one), [Primitives](https://docs.typesafe.ai/primitives).

Structured instructions can separate `question`, `focus`, examples, or a record supplied by code. Structured Choice descriptions can separate coverage, exclusions, and examples. Reuse the same descriptive keys across options. These keys are your labels inside content, not extra reserved API fields.
Sources: [Advanced structure](https://docs.typesafe.ai/primitives/advanced), [Choice](https://docs.typesafe.ai/primitives/choice).

Compatibility boundary: the HTTP reference specifies non-null string/object/array `state`, `instructions`, Noul descriptions, and Score level descriptions. The advanced page's EntryType table also lists null for instructions, Noul descriptions, and Score entries; Python question schemas permit optional/null instructions and Noul descriptions but require non-null Score entries. Do not flatten this disagreement into a universal null guarantee. All examples here use the shared non-null shapes; Choice null descriptions are documented but explicit descriptions are preferable when boundaries matter.
Sources: [HTTP API](https://docs.typesafe.ai/api), [Advanced structure](https://docs.typesafe.ai/primitives/advanced), [Python question types](https://docs.typesafe.ai/sdk/python/api/types/questions).

## Complete request: contrastive Choice and explicit Noul

JSON body for `POST https://api.typesafe.ai/v1/systemone`:
```json
{
  "model": "jev-1.13.0",
  "state": {
    "ticket": {
      "messages": [{"speaker": "customer", "text": "I mailed my return yesterday. Has your warehouse received it?"}]
    }
  },
  "questions": {
    "return_topic": {
      "type": "choice",
      "instructions": {
        "question": "Which topic does the customer's question in `ticket.messages[0].text` concern?",
        "focus": "Classify the main question; a mention of a return alone does not establish its topic."
      },
      "criteria": {
        "eligibility": {
          "covers": "Rules for whether a proposed return is permitted.",
          "excludes": "Progress of a return that has already begun.",
          "examples": ["May I return an opened item?", "How long is the return window?"]
        },
        "progress": {
          "covers": "Receipt or processing progress of an existing return.",
          "excludes": "General rules for whether an item can be returned.",
          "examples": ["Did my return reach your warehouse?", "Has my existing return been processed?"]
        },
        "other": {
          "covers": "A main question outside return eligibility and existing-return progress.",
          "excludes": "A question covered by either named topic.",
          "examples": ["Where can I find the size chart?"]
        }
      }
    },
    "refund_request": {
      "type": "noul",
      "instructions": "Does the customer explicitly ask to receive a refund in `ticket.messages[0].text`?",
      "criteria": {
        "true": {"definition": "The customer asks for money to be returned.", "examples": ["Please give me my money back."]},
        "false": {"definition": "The customer makes no refund request, including return tracking or reporting a refund already received.", "examples": ["Has my parcel reached you?", "The refund arrived today."]}
      }
    }
  }
}
```
Choice expresses one nominal selection. Noul expresses P(yes): `true` must agree with the question's yes, and `false` with its no. Near 0.5 is uncertainty about the defined predicate, not medium intensity. Criteria are optional for Noul; use them when the boundary needs clarification, then compare versions during authorized evaluation.
Sources: [Choice](https://docs.typesafe.ai/primitives/choice), [Noul](https://docs.typesafe.ai/primitives/noul).

The example strings are historical teaching cases under their correct criterion; the current ticket is only in `state`. Do not insert the current ticket's expected answer into state or examples. Avoid contradictory examples, inverted yes/no descriptions, and broad labels such as “good” without a definition.
Sources: [Advanced structure](https://docs.typesafe.ai/primitives/advanced), [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13).

## Self-contained Score anchors

Each level is independently judged against state. Its meaning must stand alone, not depend on “the previous level, but more.” Array position defines the level number starting at zero. The returned score is the probability-weighted level position, not a recovered physical quantity.
Source: [Score](https://docs.typesafe.ai/primitives/score).
```json
{
  "model": "jev-1.13.0",
  "state": {"report": {"text": "Export is unavailable in one browser, but the same export works in another supported browser."}},
  "questions": {
    "functional_impact": {
      "type": "score",
      "instructions": "How much functional disruption is described in `report.text`, using stated evidence about workaround availability?",
      "criteria": [
        {"definition": "Appearance or wording is wrong; every function remains usable.", "examples": ["A menu label has a spelling error."]},
        {"definition": "A function fails or is degraded, but a stated alternative completes the same task.", "examples": ["A file upload fails in one supported browser and succeeds in another."]},
        {"definition": "A necessary function cannot be completed, and explicit evidence establishes that no available workaround completes the same task.", "examples": ["Every available sign-in method fails, and support confirms there is no other way to access the account."]}
      ]
    }
  }
}
```
Repeat the same field labels on every level. Describe observable boundaries and scoped examples. These levels measure disruption; they do not also decide response deadline, financial impact, or routing priority. Those are separate questions or code rules.
Unknown workaround availability needs a separate knownness question and review gate; do not map absent workaround evidence to a low or high disruption level.
Sources: [Score](https://docs.typesafe.ai/primitives/score), [Composite scoring](https://docs.typesafe.ai/patterns/composite-scoring).

## Arrays and code-supplied records

An instructions array is suitable for a short list of guidance about the same judgment; it does not create multiple questions:
```json
{"type":"noul","instructions":["Does `passage.text` try to direct the answering assistant's behavior?","Inspect instructions addressed to the assistant; ordinary quoted task content alone is insufficient."],"criteria":{"true":"The passage attempts to control the assistant's behavior.","false":"The passage contains evidence or ordinary content without attempting to control the assistant."}}
```
For a record-dependent comparison, keep the database row or field specification structured rather than interpolating every value into a long sentence. This is a question object, to place under a chosen ID in `questions`; the state supplies `source_text`:
```json
{
  "type": "noul",
  "instructions": {
    "question": "Does `source_text` support `extracted_field` as the value described by `field_spec`?",
    "field_spec": {"path": "event.location", "description": "The venue where the event takes place"},
    "extracted_field": "Riverside Hall"
  },
  "criteria": {"true": "The source identifies this venue as the event location.", "false": "The source does not support this venue as the event location, including incidental mentions."}
}
```
Sources: [Advanced structure](https://docs.typesafe.ai/primitives/advanced), [SDE cascade](https://docs.typesafe.ai/cookbooks/sde_cascade), [Noul](https://docs.typesafe.ai/primitives/noul).

## Missing evidence and application policy

Do not equate missing evidence with an explicit contradiction. If both matter, use a three-option Choice rather than encoding “unknown” as Noul 0.5. The citation cookbook distinguishes support, contradiction, and silence; date extraction also provides an absence option.
Sources: [Citation checking](https://docs.typesafe.ai/cookbooks/citation_check), [Date extraction](https://docs.typesafe.ai/cookbooks/date_extraction_cookbook).
```json
{"type":"choice","instructions":"How does `passage.text` relate to `claim`?","criteria":{"supported":"The passage establishes the claim.","contradicted":"The passage establishes that the claim is false.","not_stated":"The passage establishes neither the claim nor its negation."}}
```
Keep deterministic absence checks, data validity, uncertain-model output, and composite policy distinct. Example policy fragment, using SDK answer objects and thresholds selected by the application:
```python
def decide(state, answers, low, high, impact_levels, weights):
    if not state["passage"]["text"].strip():
        return "missing_evidence"  # deterministic input absence
    relation = answers["relation"]
    if relation.choice == "not_stated":
        return "insufficient_evidence"
    p = answers["requires_review"].noul
    if low <= p <= high:
        return "uncertain"  # application review band, not a third Noul value
    normalized = {key: answers[key].score / (len(levels) - 1)
                  for key, levels in impact_levels.items()}
    composite = sum(weights[key] * value for key, value in normalized.items())
    return {"relation": relation.choice, "review_signal": p > high, "impact": composite}
```
Validate that required answers exist, probability values are usable, `0 <= low < high <= 1`, every Score has at least two levels, and weights are intentional before this policy runs. Do not silently turn an absent answer into zero. Make gates for critical flags explicit rather than averaging them away. For opposite-oriented signals use `1 - p` in code while preserving the original question's yes/no meaning.
Sources: [Self-consistency Nouls](https://docs.typesafe.ai/cookbooks/consistency_noul_cookbook), [Guardrails](https://docs.typesafe.ai/cookbooks/llm_guardrails), [SDE cascade](https://docs.typesafe.ai/cookbooks/sde_cascade), [Skill suggestion](https://docs.typesafe.ai/cookbooks/skill_suggestion).

## Before-and-after rewrites

1. Syntax/layout: before, instructions is the string “Does source_text support Riverside Hall as the event.location value, meaning the event venue?” After, use the structured field-verification object above. The judgment is unchanged; named fields isolate variable data from reusable wording.
2. Wording/scope: before, ID `refund_request` accompanies instruction “Is it requested?” After: ``Does the customer explicitly ask to receive a refund in `ticket.messages[0].text`?`` IDs cannot supply the omitted predicate or target. “Explicitly” narrows the intended rule, so validate that restriction with the policy owner.
3. Semantic change: replace `"Should we act on this ticket?"` with separate refund-request Noul, functional-impact Score, and topic Choice, then compose their outputs in code. This changes one opaque policy judgment into observable signals; it is not a formatting-only rewrite.
4. Semantic change: replace a binary claim-supported Noul with the three-way relation Choice above when contradiction and absent evidence require different outcomes. Preserve a Noul when only the probability of a clearly defined yes/no claim is needed.
Sources: [Build guide](https://docs.typesafe.ai/concepts/how-to-build-with-system-one), [Primitives](https://docs.typesafe.ai/primitives), [Citation checking](https://docs.typesafe.ai/cookbooks/citation_check).

## Cookbook patterns and checks

- RAG: pair `query` with one named passage; ask relevance, useful evidence, contradiction, and injection separately, then apply ordered gates in code. Cookbook thresholds are corpus-specific, not service defaults. [RAG cookbook](https://docs.typesafe.ai/cookbooks/classifying_rag_passages)
- Verification: reuse a complete question with structured `field_spec` and `extracted_field`; branch separately for empty extraction values. [SDE cascade](https://docs.typesafe.ai/cookbooks/sde_cascade)
- Candidate comparison: put both records in one state, use a descriptive middle Score level for plausible variants, and ask companion field questions; compare numeric fields in code. [Entity alignment](https://docs.typesafe.ai/cookbooks/entity_alignment)
- Retrieval/ranking: a Choice selects the best supplied candidate even if every candidate is poor. Include an explicit fallback or a separate absolute-fit Noul gate when rejection matters. [Choice](https://docs.typesafe.ai/primitives/choice), [Skill suggestion](https://docs.typesafe.ai/cookbooks/skill_suggestion)

Preflight: verify paths exist; read instructions and criteria together for polarity and scope; inspect labels on teaching examples; keep current labels out of inputs; route missing/uncertain evidence explicitly; keep arithmetic, counts, dates, control flow, and side effects in code. For Jev 1.13 test swapped Choice order, negations, overlapping categories, irrelevant state, and adversarial self-classification. Precise formatting is not a security guarantee.
Source: [Jev 1.13 jaggedness](https://docs.typesafe.ai/model-jaggedness/jev-1.13), applicable to `jev-1.13`, last reviewed by TypeSafe 2026-10-02.
