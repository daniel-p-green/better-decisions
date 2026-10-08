# Cloudflare Clef prompt patterns

Checked **2026-10-08 UTC**. Read this for Clef/Clef-flash wording, evidence formatting, missing-data handling, or image questions; read [cloudflare-contract.md](cloudflare-contract.md) for the full transport/output contract.
Contents: [Facts](#documented-boundaries) · [State](#structured-evidence-and-chronology) · [Noul](#noul-one-bounded-proposition) · [Choice](#choice-contrastive-categories) · [Score](#score-ordered-self-contained-anchors) · [Unknown](#missing-is-not-false) · [Images](#image-specific-wording-and-mapping) · [Rewrite](#before-and-after-semantic-change-flags) · [Encoding/tests](#local-encoding-and-bundle-tests) · [Sources](#sources).

**Evidence labels:** “Documented” means first-party contract/code; “candidate pattern” means an original, untested design hypothesis. Examples are synthetic, not measured improvements or operational policy. No inference was performed.

## Documented boundaries

- Hosted bodies use `model`, `state`, `questions`; match `clef` to `@cf/cloudflare/clef`, or `clef-flash` to `@cf/cloudflare/clef-flash`. Do not substitute chat messages. [Hosted pages](https://developers.cloudflare.com/workers-ai/models/clef/).
- Hosted `instructions` are required for every question. A nonempty string is sufficient; an object/array may hold the question plus referenced material. Put the actual judgment there, even with descriptive IDs. [Clef catalog](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef.json).
- `state` is documented as text/object/array. Question-map limits are 1–64; choice criteria are an option-description map with 2–255 options; score criteria are 2–10 levels in an array, lowest first. IDs are response keys. [Flash catalog](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef-flash.json).
- Noul is probability of yes, not severity. Choice selects one category; score is a probability-weighted ordinal level, potentially fractional. These primitives do not return an explanation or extracted text. [Clef catalog](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef.json).
- The local model card permits omitted instructions and falls back to the question ID; that does not make omission valid on Workers AI. [Model card](https://huggingface.co/Cloudflare/clef).

## Structured evidence and chronology

**Candidate pattern:** use named evidence fields, attribution, observation times, and a chronological array. Separate policy from quoted reports. Preserve relevant contradictions rather than replacing reports with an unsupported conclusion.
The following complete, original hosted-shaped request is syntactically valid JSON. Its synthetic policy is specified solely for illustration; the request has not been executed.

```json
{
  "model": "clef",
  "state": {
    "evaluated_at": "2026-10-08T12:10:00Z",
    "policy": {
      "current_report": "Use the latest monitoring report; customer messages describe symptoms, not verified scope.",
      "owner": "Platform handles checkout software failures; billing handles account charge disputes."
    },
    "evidence": [
      {"at": "2026-10-08T12:00:00Z", "source": "customer", "text": "Checkout failed for me."},
      {"at": "2026-10-08T12:05:00Z", "source": "monitoring", "text": "Checkout blocked in one region."},
      {"at": "2026-10-08T12:09:00Z", "source": "monitoring", "text": "Checkout works in every region; responses remain slow."}
    ]
  },
  "questions": {
    "checkout_blocked": {
      "type": "noul",
      "instructions": "Does the current monitoring report establish that checkout cannot complete in at least one region? Use state.policy.current_report and state.evidence.",
      "criteria": {
        "true": "The current monitoring report states that checkout cannot complete in one or more regions.",
        "false": "The current monitoring report states that checkout can complete in every region; slowness alone does not establish blocking."
      }
    },
    "initial_owner": {
      "type": "choice",
      "instructions": "Choose the initial owner for the reported checkout problem using state.policy.owner. Judge the problem described, not the customer's emotional tone. If both categories are reported and the owner policy supplies no precedence, choose unknown.",
      "criteria": {
        "platform": "Checkout software failure or degradation; excludes an account charge dispute without a technical symptom.",
        "billing": "Account charge dispute; excludes a checkout software failure or degradation.",
        "unknown": "The evidence does not identify either problem category, or both categories are reported and the owner policy supplies no precedence."
      }
    },
    "current_impact": {
      "type": "score",
      "instructions": "Rate current checkout impact using only the current monitoring report selected by state.policy.current_report. Do not rate historical peak impact or customer frustration.",
      "criteria": [
        "Checkout completes normally in every region; no degradation or blocking is reported.",
        "Checkout completes in every region, but degradation such as slowness is reported; no region is blocked.",
        "Checkout cannot complete in exactly one region; checkout completes in the other regions.",
        "Checkout cannot complete in two or more regions."
      ]
    }
  }
}
```

The fields `policy`, `source`, `at`, and `evidence` are application-defined data, not special Clef control fields. Referencing their paths is a wording hypothesis, not a documented query language.
Keep source text explicitly quoted as evidence; it must not redefine the question or categories. Enforce the application/schema boundary in code and test adversarial quoted instructions.
Do not fabricate monitoring evidence or source authority. If timestamps/order can be resolved exactly, select/sort reports in code and retain attribution; label that preprocessing change.
Illustrative, untested labels under this synthetic policy: Noul **false**, choice **platform**, score level **1**. No probabilities or model results are implied.

## Noul: one bounded proposition

**Candidate pattern:** name the subject, observable condition, evidence scope, and time window. Keep one proposition per question.
- Weak: “Is this serious?” Stronger candidate: “Does the current monitoring report establish that checkout cannot complete in at least one region?”
- Use `criteria.true`/`criteria.false` to state the positive boundary and nearby exclusion. Do not quietly redefine “false” as both disproved and unobserved.
- Distinguish “the report says checkout is blocked” from “checkout really is blocked”; the former tests reported evidence, the latter may require verification.
- Avoid bundling “blocked, widespread, and urgent” unless that exact conjunction is the agreed output meaning. Independent conditions can be separate questions, with composites in code.
- A near-0.5 Noul probability is uncertainty about the binary proposition; it is not a third category or an “unknown” sentinel. Preserve the agreed review policy separately.
- Exact counts, dates, arithmetic, and permissions belong in code when the input provides reliable structured values. Do not ask a model to reproduce a deterministic comparison unnecessarily.

## Choice: contrastive categories

**Candidate pattern:** make each option a usable inclusion/exclusion definition, and explain ambiguous overlaps in instructions.
- Avoid merely “technical,” “billing,” “other”; “account charge dispute without a technical failure” differentiates a neighboring class.
- State whether the task asks for primary intent, initial owner, or all applicable findings. A choice selects one; a multi-label job needs separate condition questions or another output design.
- Add `unknown`/`other` only when the task permits that outcome. These are ordinary options, not built-in abstention controls.
- Preserve stable option IDs consumed by code. Renaming IDs is an interface change; changing category boundaries is a taxonomy/policy change.
- A tie-break such as “technical failures take precedence over charge disputes” is policy. Obtain it or mark it pending, rather than inventing it to make the prompt tidy.
- Structured descriptions are allowed by the hosted catalog, but short explicit prose is a sensible first candidate. Extra structure/examples must earn their place in evaluation.

## Score: ordered, self-contained anchors

**Candidate pattern:** describe each level in the same dimension, with observable adjacent boundaries. Use lowest-to-highest order and keep it fixed.
- Weak anchors: “low,” “medium,” “high.” Better candidates explain completion, degradation, one-region blocking, and multi-region blocking, as in the request above.
- Each anchor should stand on its own; avoid “worse than above,” unexplained numeric labels, or a highest level mixing user anger with service impact.
- Self-contained anchors are a transferable design hypothesis for Clef. [Jev's independently judged level descriptions](https://docs.typesafe.ai/primitives/score) are a provider-specific implementation claim, not a Clef guarantee.
- Specify current versus peak impact and who/what is rated. Do not silently collapse scope, urgency, and frustration into one severity score.
- Missing impact evidence is not the lowest impact level. Gate use of the score on evidence sufficiency, or explicitly redesign the output if an unknown result is needed.
- A fractional expected score is not a newly defined rubric category. Preserve the per-level distribution; thresholds/action policy require validation outside the prompt.
- Splitting or reordering anchors, adding levels, or changing from expected score to winning level alters output semantics. Treat it separately from wording cleanup.

## Missing is not false

**Candidate pattern:** if the downstream job needs supported/refuted/unknown, use a choice with those three explicit meanings rather than hoping a binary probability encodes evidence status.
This question fragment can be placed in a hosted request's `questions` map:

```json
{
  "checkout_evidence_status": {
    "type": "choice",
    "instructions": "What does the current monitoring evidence establish about whether checkout cannot complete in at least one region? Contradictory unresolved reports or no current report count as unknown.",
    "criteria": {
      "supported": "A current monitoring report establishes checkout cannot complete in at least one region.",
      "refuted": "A current monitoring report establishes checkout completes in every region.",
      "unknown": "No current monitoring report establishes either outcome, or current reports conflict without a resolved precedence rule."
    }
  }
}
```

Replacing Noul with this choice is a **primitive/interface and meaning change**, not a meaning-preserving rewrite. Alternatively retain Noul and add a sufficiency question, then gate its use in code; test the expanded bundle.
Distinguish absent keys, explicit nulls, unreadable evidence, contradiction, and negative findings in examples. A model's `unknown` choice is not guaranteed reliable abstention.

## Image-specific wording and mapping

Documented hosted images are PNG/JPEG/WebP, embedded as base64 objects or data URLs, before state; remote URLs are not accepted. Maximum: four images; 4 MiB and 16 megapixels each, 8 MiB decoded total, 13 MiB request body. Context is 65,536 tokens. [Hosted documentation](https://developers.cloudflare.com/workers-ai/models/clef/).
**Candidate pattern:** explain which image depicts which evidence and what visible feature answers the question. The following fragments use valid bytes for a meaningless one-pixel gray PNG, not a receipt or an evaluated example:

```json
{"images": [{"content_type": "image/png", "base64": "iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAAAAAA6fptVAAAACklEQVR4nGNoAAAAggCBd81ytgAAAABJRU5ErkJggg=="}]}
```

Equivalent data-URL form for those bytes:

```json
{"images": ["data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAAAAAA6fptVAAAACklEQVR4nGNoAAAAggCBd81ytgAAAABJRU5ErkJggg=="]}
```

For an actual receipt, an application-defined mapping could be `state.media = [{"image_index": 0, "role": "receipt front", "source": "uploaded receipt"}]`. Zero-based indexing is this application's convention, not a reserved hosted selector; verify multi-image alignment.
Original wording candidates:
- “In the image identified as receipt front in state.media, is the printed total visually legible? Do not substitute an amount from state.transcription.”
- “Choose the evidence status of a printed total in that image: readable total, visibly absent total on a complete page, or unknown because blurred/cropped/occluded.”
- For two images: “Compare the visible item identifier on the package photo with the identifier on the label photo described in state.media; missing or unreadable identifiers are unknown.”
Separate image observations, OCR/transcriptions, and claimed metadata; mark whether transcription is authoritative, advisory, or excluded. Redacting/cropping/OCR replacement can remove evidence and is a preprocessing change, not mere formatting.
Do not ask Clef to emit arbitrary OCR text through a decision question. Use a separately authorized extraction path if the workflow requires exact text; keep a visual decision when that is the actual job.
The local card documents PIL images and video frame arrays. Hosted descriptions mention video, but reviewed hosted schemas expose no `videos` field; no audio path was established. Do not infer hosted audio/video support or copy local media kwargs into hosted bodies. [Model card](https://huggingface.co/Cloudflare/clef), [catalog](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef.json).

## Before and after: semantic-change flags

Given the synthetic policy and evidence above, an ambiguous baseline fragment is:

```json
{"current_impact": {"type": "score", "instructions": "How bad is checkout?", "criteria": ["None", "Slow", "One region blocked", "Many regions blocked"]}}
```

The full request's `current_impact` fragment is a candidate after version. Classify changes before calling it an optimization:
- **Wording clarification, conditional:** defining current monitoring evidence preserves meaning only if that was already the intended baseline interpretation.
- **Rubric clarification, conditional:** “many” → “two or more” preserves meaning only if that numeric boundary was already agreed; otherwise it is a rubric/policy change.
- **Potential scope change:** ignoring historical peak impact or excluding customer frustration changes the job if the old workflow considered them.
- **No change in this example:** same score primitive, four-level order, question ID, and expected-score output. This alone does not prove semantic equivalence.
- **Explicitly meaningful alternatives:** adding `unknown`, changing Noul to tri-state choice, or replacing photos with OCR changes taxonomy/interface/evidence. Keep separate candidates and obtain needed policy decisions.
Do not promise improved accuracy from this static rewrite. Keep the baseline and compare labeled cases, especially recovery, missing report, conflicting reports, and adjacent score boundaries.

## Local encoding and bundle tests

Documented **local release only**: structured values serialize with sorted JSON keys; question iteration order is preserved; choice IDs are sorted; score-array order is preserved. Missing/empty instructions fall back to the question ID, which is encoded. Media precedes state; schema is budgeted before state tail-truncation; default encoding limit is 16,384 tokens. [Release code](https://huggingface.co/Cloudflare/clef/blob/main/joint_schema_model.py).
Do not manually insert the local encoder's control tokens or schema delimiters into hosted state. Use the public request fields. Hosted serialization/sorting/truncation details are not established by local code.
**Candidate tests:** preserve timestamps in an array, explicitly select permitted evidence, and check that crucial late evidence survives preprocessing/truncation. Moving evidence may change the retained input, so record it as a preprocessing candidate.
Cloudflare documents joint cross-field attention and permutation-based training. That motivates testing the whole schema bundle; it proves neither independence nor invariance. [Announcement](https://blog.cloudflare.com/clef-decision-models/).
Jev explicitly documents independent same-call questions and invisible question IDs. Those guarantees must not be imported into Clef. [Jev primitives](https://docs.typesafe.ai/primitives).
- Version state representation, instructions, option IDs/descriptions/order, score anchors, and complete question bundle together.
- Compare a target question alone versus in the production bundle; add/remove/reorder unrelated questions and measure answer/probability/threshold changes.
- For hosted choice order tests, preserve ID-to-description meaning. Local insertion-order shuffles may be erased by sorting; renaming IDs to change sort order also changes model-visible text and is a separate test.
- Paraphrase instructions, test ID changes, contradictions, quoted prompt injections, missing fields, and image-to-state alignment. Never shuffle ordinal score levels.
- A later question cannot consume an earlier emitted answer in the same request. For a real dependency requiring fetched evidence, build the second state in code; joint scoring does not establish a causal workflow.

## Sources

All sources below were inspected on **2026-10-08 UTC**; `main`/`production` are mutable refresh links, not hosted revision pins.
- [Clef hosted page](https://developers.cloudflare.com/workers-ai/models/clef/) and [Clef-flash hosted page](https://developers.cloudflare.com/workers-ai/models/clef-flash/).
- [Clef catalog schema](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef.json) and [Flash catalog schema](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef-flash.json).
- [Cloudflare model card](https://huggingface.co/Cloudflare/clef) and [local release code](https://huggingface.co/Cloudflare/clef/blob/main/joint_schema_model.py).
- [Cloudflare announcement](https://blog.cloudflare.com/clef-decision-models/), [TypeSafe primitive guarantees](https://docs.typesafe.ai/primitives), and [Jev score encoding](https://docs.typesafe.ai/primitives/score).
