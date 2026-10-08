# Jev and likely referent Clef

First-party research checked 2026-10-08. “Cleft” was not verified as an exact model name; Cloudflare **Clef** is a plausible, unconfirmed referent. Do not silently rename the user's source or imply equivalence.

## Verified provider facts

**TypeSafe Jev**
- [Primitives](https://docs.typesafe.ai/primitives): question IDs are not visible to the model; instructions must contain the actual question. Same-call questions are documented as independent; dependent judgments require separate calls.
- [Score](https://docs.typesafe.ai/primitives/score): levels are judged without their numeric index or adjacent descriptions, motivating self-contained semantic anchors. This is a Jev-specific implementation fact.
- [Noul](https://docs.typesafe.ai/primitives/noul): probability of a yes/no proposition, not severity.
- [Confidence](https://docs.typesafe.ai/confidence): distribution-derived formulas differ by primitive; confidence is not a guarantee of correctness. Do not transfer these formulas to OpenAI.
- [Jev 1.13 weaknesses](https://docs.typesafe.ai/model-jaggedness/jev-1.13): documented concerns include negation, literal interpretation, arithmetic/date/counting errors, irrelevant context, adversarial input, and option-order bias. These motivate tests, not blanket claims about all decision models.
- [Building with TypeSafe](https://docs.typesafe.ai/concepts/how-to-build-with-system-one): keep deterministic rules and workflow control in code; provide relevant state and separate judgments.

**Cloudflare Clef**
- [Hosted documentation](https://developers.cloudflare.com/workers-ai/models/clef/) supplies its own model selectors, input schema, and limits. They do not define the OpenAI endpoint.
- [Local model card](https://huggingface.co/Cloudflare/clef) describes fallback to question ID when instructions are absent, unlike Jev. Hosted Clef requires instructions in its own schema. Keep those deployment contracts separate; explicit complete instructions avoid relying on either fallback behavior.
- [Announcement](https://blog.cloudflare.com/clef-decision-models/) describes joint cross-field attention. Do not import Jev's independence property into Clef or OpenAI. Training with permutations and calibration-oriented losses does not prove invariance or calibration on a user's task.

## Transfer as hypotheses

Worth testing on OpenAI: one semantic target per question; compact relevant evidence; contrastive categories; self-contained ordered anchors; deterministic arithmetic outside the model; a validation-derived review band; category-order, bundle, paraphrase, missing-evidence, and injection tests.

Do not transfer: `state`/Noul request fields, confidence formulas, exact context or category limits, score-level caps, question independence guarantees, determinism, modality support, pricing, or vendor benchmark rankings. Verify OpenAI's own contract and measure the user's workload.

Favor the smallest change tied to an observed failure. Clearer instructions can help without architecture claims; extra verbosity, examples, or more rubric levels are experiments, not universally beneficial tricks.
