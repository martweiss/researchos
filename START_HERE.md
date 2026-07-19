# Start Here

ResearchOS is a collection of Markdown-defined research roles and
procedures. It is not yet an executable agent framework. `MODEL` is the
orchestrator placeholder: the frontier LLM selected and invoked through the
CLI. `MODEL` may be Codex, Claude Code, AWS Bedrock (bring your own model),
or a model invoked using usage credits. It reads these files, adopts the
appropriate role, and—when the execution layer is available—runs the research
process described by them.

This file is intentionally topic-agnostic. It applies to any domain that
benefits from structured research and adversarial review.

## Fast path

Open this repository in the CLI environment for your chosen `MODEL` and send
the kickoff prompt below with a rough user request. The first response should
be a research charter only. Do not start browsing or launch the research
panel until the charter has been reviewed and approved.

```text
Read START_HERE.md first. Then read README.md and
agents/prompt-architect.md.

Act as the Prompt Architect for the rough research request below.

Rewrite it into a rigorous research charter suitable for ResearchOS. Design
the appropriate research organization for this topic rather than assuming a
fixed roster. The charter should define the primary question, scope,
sub-questions, source-acquisition plan, specialist lenses, independent
research outputs, peer-review protocol, rebuttal process, supervisory
synthesis, adversarial review, forecasting where appropriate, and any
downstream translation the request calls for.

Keep the primary research independent from downstream recommendations or
applications. Specify what evidence would support, weaken, or falsify the
major conclusions. Match the source classes and review standards to the
domain.

Do not perform the research yet. Do not browse yet. Return only the draft
charter for my review and approval.

ROUGH USER REQUEST:

[Paste the rough request here.]
```

The directory [`example_initial_prompts/`](example_initial_prompts/) contains
rough requests for testing this workflow. Those files are intentionally not
charters; the Prompt Architect is expected to redesign them before research
begins.

## Approval and execution

After reviewing the charter, reply:

```text
APPROVED — run the charter.
```

The Research Director should then:

1. freeze the approved charter;
2. create the topic-specific research organization;
3. instantiate independent research runs for the approved specialist
   lenses;
4. wait until independent research outputs are complete;
5. assign peer reviews according to the charter, with all-versus-all review
   when that protocol is appropriate;
6. collect reviews before exposing aggregate scores or other reviews;
7. route reviews to authors for rebuttal or explicit revision;
8. run source auditing and coverage checks;
9. supervise synthesis while preserving unresolved minority views;
10. run adversarial review and forecasting where appropriate;
11. stop before downstream recommendations if a material research challenge
    remains unresolved.

Before any applied output—investment, policy, product, strategic, or other
recommendation—review the technical or factual synthesis and its remaining
uncertainty.

## Source expectations

The charter should define the appropriate source classes for its domain.
Agents may use primary research, official records, technical documentation,
structured measurements, expert commentary, datasets, historical sources,
and secondary reporting as appropriate, but each class must be weighted for
what it can actually establish.

Every source treated as evidence must have been retrieved and inspected.
Record the source locator, author or organization, publication date,
retrieval date, relevant excerpt or section, methodology or version where
applicable, limitations, and the exact proposition supported. Unverified
material remains a lead, not evidence.

Popularity, credentials, institutional affiliation, or rhetorical confidence
may help discover sources, but do not establish that a claim is true.

## If execution is not available

The Markdown files define the research protocol, but they do not by
themselves create parallel agent runs or domain-specific web/API access. If
the selected `MODEL` runtime cannot launch the required research organization
or retrieve the requested sources, state that limitation clearly. Do not
claim that research, reviews, or source verification happened when they did
not.
