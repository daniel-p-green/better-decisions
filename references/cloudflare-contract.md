# Cloudflare Clef provider contract

Checked **2026-10-08 UTC** against first-party public documentation and release code. This is a documentation-verified contract, not a live API test. No inference request, account operation, installation, or weight download was performed. Refresh the contract before executable use.

Contents: [Transport](#identity-and-transport) · [Hosted input](#hosted-input) · [Answers](#hosted-answers-and-confidence) · [Modalities](#hosted-modalities-and-bounds) · [Local release](#local-release-is-a-separate-contract) · [Deployment and cost](#deployment-budget-and-reproducibility) · [Evaluation](#prompt-bundle-and-evidence-caveats) · [Refresh](#refresh-before-executable-use)

## Identity and transport

**Clef** and **Clef-flash** are verified names. A provider named **Cleft** was not established by this research; do not silently substitute Clef for a requested Cleft integration.

- Hosted Clef: `@cf/cloudflare/clef`; body selector: `"model": "clef"`.
- Hosted Clef-flash: `@cf/cloudflare/clef-flash`; body selector: `"model": "clef-flash"`.
- REST: `POST https://api.cloudflare.com/client/v4/accounts/{account_id}/ai/run/{hosted_model_name}`. Send the model-specific body directly. Clef's required body `model` is its short selector; it is not an outer wrapper. [Model documentation](https://developers.cloudflare.com/workers-ai/models/clef/), [REST run reference](https://developers.cloudflare.com/api/resources/ai/methods/run/).
- Workers: `env.AI.run(hosted_model_name, body)`, with an authorized Workers AI binding. This uses Cloudflare-hosted inference. [Binding documentation](https://developers.cloudflare.com/workers-ai/configuration/bindings/).
- Do not replace this contract with `messages`, a text-generation `prompt`, or an assumed `/v1/chat/completions`/hosted `/v1/systemone` endpoint. Those routes were not verified for hosted Clef.
- REST requires an authorized account ID and bearer token. The general REST guide shows a Cloudflare `result`/`success`/`errors`/`messages` envelope; the Clef binding example returns the model object directly. Verify the actual transport response before parsing `answers`. [REST getting started](https://developers.cloudflare.com/workers-ai/get-started/rest-api/).

## Hosted input

The two published catalog schemas have the same decision input contract:

- Required top-level fields: `model`, `state`, `questions`.
- `state`: documented string, object, or array. Long text may be truncated.
- `questions`: 1–64 entries keyed by question ID. Documented IDs use letters, digits, `_`, `.`, `-`, with at most 100 characters. Prefer conservative ASCII IDs.
- Every hosted question requires `type` and `instructions`. Instructions may be a nonempty string or structured object/array containing the question and referenced data.
- `noul`: binary yes/no proposition; optional `criteria` object with `true` and `false` descriptions.
- `choice`: required `criteria` object, 2–255 named options. Each nonempty option ID maps to a string, object, array, or `null` description.
- `score`: required `criteria` array, 2–10 ordered descriptions, lowest first. Levels are zero-indexed; descriptions may be strings, objects, or arrays.

Some descriptive restrictions, including choice-option counts and ID rules, are not fully encoded as JSON Schema keywords. Apply explicit validation in addition to schema validation. [Official Clef-flash catalog JSON](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef-flash.json).

### Minimal original request

For `@cf/cloudflare/clef`, the following original synthetic body matches the documented shape. It has not been sent to Cloudflare; no result is implied.

```json
{
  "model": "clef",
  "state": {
    "ticket": "The export button returns an error for one customer. Other functions work."
  },
  "questions": {
    "outage": {
      "type": "noul",
      "instructions": "Does the stated evidence establish a service-wide outage?"
    },
    "owner": {
      "type": "choice",
      "instructions": "Which team should inspect this ticket first?",
      "criteria": {
        "engineering": "Product defects and technical errors",
        "billing": "Invoices and payment questions"
      }
    },
    "impact": {
      "type": "score",
      "instructions": "Rate only the customer impact explicitly stated in the ticket.",
      "criteria": ["No blocked task", "One task blocked", "All core tasks blocked"]
    }
  }
}
```

For Flash, change both the hosted route/binding selector and short body selector. Do not rely on the schema's acceptance of either short name to resolve a mismatched endpoint.

## Hosted answers and confidence

Model output requires `model`, `answers`, and `usage`. `answers` is keyed by the same question IDs, and each answer is an object:

- `noul`: `{ "type": "noul", "noul": p }`, where `p` is the probability of yes, in `[0,1]`. It is not a bare number or boolean.
- `choice`: `type`, `choice`, `probabilities`, `confidence`. `choice` is the highest-probability option ID.
- `score`: `type`, `score`, `legend`, `probabilities`, `confidence`. `score` is a probability-weighted level and can be fractional. `legend` maps stringified level indices to descriptions.
- `probabilities` maps options/levels to values in `[0,1]`; documented to sum to 1 per question.
- Hosted `confidence` is in `[0,1]` and described as probability-derived. The hosted schema does not specify its formula.
- `usage` contains integer `input_tokens` and `output_tokens`.

These fields are verified in the [official Clef catalog JSON](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef.json). Do not transfer a confidence formula from another provider or the local implementation into the hosted contract without verification. Do not present classifier confidence as demonstrated real-world correctness, expected utility, or authorization to act.

## Hosted modalities and bounds

Both hosted pages list **65,536-token context**. The explicit hosted image contract accepts up to four PNG/JPEG/WebP images, each at most 4 MiB and 16 megapixels; total decoded image bytes at most 8 MiB; whole body at most 13 MiB. Images precede the state. Remote image URLs are refused. [Clef](https://developers.cloudflare.com/workers-ai/models/clef/), [Clef-flash](https://developers.cloudflare.com/workers-ai/models/clef-flash/).

Image entries are base64 data URLs, or objects with `content_type` and `base64`. The model descriptions mention video, but the published hosted schema contains no `videos` field. Treat hosted video and local `media_kwargs` as **unverified**, not enabled. [Catalog schema](https://raw.githubusercontent.com/cloudflare/cloudflare-docs/production/src/content/workers-ai-models/clef.json).

## Local release is a separate contract

Cloudflare's model card documents a custom `joint_schema_model.py` path: `load_release_model`, `encode_record`, `collate_records`, and `systemone`. The decision head scores allowed options jointly; the caller applies softmax per question. Generic Hugging Face “Use this model” chat-generation snippets do not establish that decision-head contract. [Cloudflare model card](https://huggingface.co/Cloudflare/clef).

Local card differences: instructions are optional with the question ID as fallback; state can be any JSON value; images are PIL objects; videos are frame arrays; processor options use `media_kwargs`; encoding defaults to **16,384 tokens**, with `max_state_tokens` available. The card reports testing on a single H200 with Torch 2.11 and Transformers 5.10.2, and Pillow for media. These are reported test conditions, not minimum hardware requirements. Both releases are Apache-2.0. Hugging Face's provider panel says neither model is deployed by an HF Inference Provider, independently of verified Workers AI hosting. [Clef card](https://huggingface.co/Cloudflare/clef), [Flash card](https://huggingface.co/Cloudflare/clef-flash).

Inspected local release code `systemone_answer` uses winning-option probability for choice confidence, maximum level probability for score confidence, `sum(index * p)` for score, and four-decimal rounding. `systemone` reports zero output tokens. Encoding preserves question iteration order, sorts choice options, and trims the end of the state to the remaining token budget; schema overflow raises an error. These implementation facts are **local only**. File revision observed: `a20e258b170e7f7a9fae1fb58d7bb6e089f2ce29`. [Release code](https://huggingface.co/Cloudflare/clef/blob/main/joint_schema_model.py), [revision](https://huggingface.co/Cloudflare/clef/commit/a20e258b170e7f7a9fae1fb58d7bb6e089f2ce29).

## Deployment, budget, and reproducibility

- Current published input prices: **USD 0.24/million tokens** for Clef; **USD 0.09/million** for Flash. Published equivalent usage: 21,818 and 8,182 neurons/million input tokens. Free allocation is 10,000 neurons/day, resets 00:00 UTC; exceeding it requires a paid plan. Paid overage is USD 0.011/1,000 neurons. Refresh billing before costing a run. [Pricing, updated 2026-10-01](https://developers.cloudflare.com/workers-ai/platform/pricing/).
- General Text Generation limit is 300 requests/minute except paid-required models. Clef pages currently classify them under Text Generation and show no paid-required marker. This establishes the published default, not account-specific availability or a guaranteed service level. Local Wrangler inference also counts toward hosted limits. [Limits, updated 2026-09-17](https://developers.cloudflare.com/workers-ai/platform/limits/).
- No immutable hosted model-revision selector or `seed` was found in the reviewed Clef schemas/pages. Record hosted model name, body selector, date, schema snapshot/hash, complete request/bundle, and transport metadata; do not call this weight-pinned reproducibility. A local code commit is not evidence that hosted weights use that revision.

## Prompt, bundle, and evidence caveats

Cloudflare describes joint cross-field attention and training with permuted field orders, prompts, and schema structures, plus calibration-oriented losses. The benchmark figures are **vendor-reported internal evaluations**. They do not establish performance, calibration, order invariance, latency, or safe automation for this application. [Announcement, published 2026-10-01](https://blog.cloudflare.com/clef-decision-models/).

Provider-specific evaluation requirements: version the whole question bundle and rubric; test bundle composition, paraphrases, IDs, option order, question order, and truncation. No reviewed source promises invariance. Do not assume independent outputs or a joint probability table. Declare a rounding tolerance when checking normalization; local four-decimal rounding can matter. The main skill governs calibration, abstention, permissions, and action gating.

## Refresh before executable use

1. Reopen both model pages, their raw catalog JSON, current pricing/limits, and the relevant model card/code. Record the check date and exact source revisions where available.
2. With separately authorized account access, the documented schema-discovery route is `GET /accounts/{account_id}/ai/models/schema?model=@cf/cloudflare/clef` (or Flash). It returns `result.input` and `result.output`. [Schema API](https://developers.cloudflare.com/api/resources/ai/subresources/models/subresources/schema/methods/get/). This research did not call it.
3. Confirm account/model access, transport envelope, modalities, token/body limits, billed units, budget and retry cap. Resolve documentation conflicts before inference; do not create access, install, deploy, or spend just to refresh this document.
4. Only if inference is separately authorized, run a small synthetic compatibility test. Save actual response metadata; check confidence behavior rather than assume local/hosted equivalence. Then run application evaluation before any automatic action.

All linked sources were inspected on **2026-10-08 UTC**. Mutable `production`/`main` links are refresh targets, not pins. Missing documented capabilities remain unverified.
