# Better Decisions

An instruction-only agent skill for designing and refining decision-model workflows. It supports OpenAI Decisions, TypeSafe Jev, and Cloudflare Clef while keeping each provider’s API contract separate.

This is an independent project, not an official OpenAI, TypeSafe, or Cloudflare product.

## What it helps with

- Turn a plain-language goal into decision criteria, a request draft, representative cases, and an evaluation plan
- Refine an existing classification, routing, filtering, or ordinal-scoring workflow while preserving its baseline
- Choose between deterministic code, retrieval, bounded decisions, and generation
- Compare whole-workflow cost, latency, quality, review coverage, and selective fallback

## Install in local Codex

Clone the complete skill folder into Codex’s user skill directory:

```sh
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/daniel-p-green/better-decisions.git "$HOME/.agents/skills/better-decisions"
```

The destination must be new; keep the existing folder if the skill is already installed. Codex discovers user skills under `$HOME/.agents/skills`. If it does not appear, restart Codex. See the [official skill documentation](https://learn.chatgpt.com/docs/build-skills) for other supported locations and installation options.

Invoke it with `$better-decisions`, for example:

> Use $better-decisions to turn my customer-routing goal into a decision-model design with test cases.

For another compatible agent, install the complete directory using that host’s supported skill mechanism. `SKILL.md` is the entry point; `agents/openai.yaml` provides optional host metadata.

## Package

- `SKILL.md`: workflow and provider selection
- `references/`: provider contracts, prompt experiments, evaluation, and economics
- `agents/openai.yaml`: display and invocation metadata
- `assets/icon.svg`: skill icon

## Limits and maintenance

Provider references are dated documentation snapshots checked on October 8, 2026. Recheck official availability, schemas, limits, and prices before executable integration. The examples are synthetic drafts; no live inference or performance improvement is implied. This skill does not include a runtime, SDK, API credentials, or a deployed service.

Using the skill does not authorize paid inference, data uploads, deployment, or downstream actions. Live evaluation needs an authorized workload and budget.

## Sources and license

Provider-specific statements link to [OpenAI documentation](https://developers.openai.com/api/docs/guides/decisions), [TypeSafe documentation](https://docs.typesafe.ai/), and [Cloudflare documentation](https://developers.cloudflare.com/workers-ai/models/clef/). References distinguish documentation, engineering hypotheses, and measured evidence. Andrew Mayne’s [choice-order anecdote](https://x.com/AndrewMayne/status/2107610144247025929) is briefly paraphrased and treated as a test hypothesis.

The repository includes the [Apache License 2.0](LICENSE). Third-party documentation and linked materials retain their own terms.
