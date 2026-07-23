# ResearchOS

ResearchOS is a Markdown-defined research panel. `MODEL` is the generic name
for the frontier LLM and harness acting as orchestrator. ResearchOS is
harness-agnostic by design: bring your own harness. Any agentic CLI or
runtime capable of multi-step tool use, source retrieval, and headless
invocation can run the protocol — Claude Code and Codex are two well-suited
examples, but the protocol does not assume or require either one. The
Markdown files define the specialist roles, source standards, debate
protocol, and handoffs; they do not depend on any specific provider or
execution surface.

## Choosing a harness

Pick whichever harness you already have and trust for agentic work. What
matters is that it can:

- run headlessly/non-interactively, so it can be invoked per role and per
  stage without a human driving each turn;
- perform multi-step tool use — web search and page retrieval at minimum,
  since several roles (Librarian, Panel Member research phase, Source
  Auditor) must actually retrieve and inspect sources, not just recall them;
- keep independent runs isolated from each other's context, so panel
  members and reviewers stay independent per the protocol.

Claude Code and Codex both satisfy this today; other agentic CLIs may as
well. There is no reusable-contract reason to prefer one over another, and
no part of `agents/` should assume a specific provider.

The system is designed for questions where the answer should emerge from
independent technical research and adversarial comparison, not from one
model producing a polished first opinion. It is topic-agnostic: the charter,
panel roster, and source plan are derived fresh from each rough request, and
no part of the protocol assumes a specific research domain.

## How to use this repository

Start with [`START_HERE.md`](START_HERE.md). It is the topic-agnostic
operational entrypoint for using ResearchOS with `MODEL`. It tells the
selected model which files to read, how to turn a rough request into a
charter, when to stop for user approval, and how to run the research process
afterward.

[`example_initial_prompts/`](example_initial_prompts/) contains rough,
user-style requests for testing the system. They are deliberately not
charters; the Prompt Architect redesigns them before any research begins.

The recommended interaction is:

1. Ask `MODEL` to read `START_HERE.md` and design a charter. At this stage it
   should not browse or begin research.
2. Read, revise, and explicitly approve the charter.
3. Ask `MODEL` to run the approved charter through the Research Director,
   independent Panel Members, all-versus-all peer review, rebuttal, source
   audit, synthesis, red-team, and forecasting stages.
4. Review the technical synthesis and forecasts.
5. Only then run the downstream Applied Analyst for the application or
   decision requested by the charter.

`README.md` documents the system. `START_HERE.md` is the runbook. The files in
`agents/` are role contracts that `MODEL` loads as needed; they are not
standalone programs. Project-local run storage is defined in
[`projects/README.md`](projects/README.md).

## Project workspaces

Each research question gets a project directory under `projects/`. The
Prompt Architect initializes the project and preserves the rough request and
charter drafts. After approval, the Research Director creates a new run
directory for the actual panel execution. Agents write only inside their
assigned run or stage folders, so independent work, reviews, revisions, and
synthesis remain auditable.

## Agent layers

- **Prompt Architect** (`agents/prompt-architect.md`) turns a rough question into a charter and a topic-specific research panel.
- **Research Director** (`agents/research-director.md`) runs the panel, controls independence, assigns reviews, and supervises synthesis.
- **Panel Member** (`agents/panel-member.md`) is the reusable contract instantiated once per specialist lens. Every member researches independently and later reviews every other member.
- **Source Auditor** (`agents/source-auditor.md`) checks that important sources are real, retrievable, correctly represented, and appropriately weighted.
- **Peer Reviewer** (`agents/peer-reviewer.md`) defines the all-versus-all grading and challenge protocol used by every panel member.
- **Librarian** (`agents/librarian.md`) supports source discovery, extraction, coverage checks, and evidence hygiene across the panel.
- **Domain Analyst** (`agents/domain-analyst.md`) reconciles the panel into explicit technical claims without erasing minority views.
- **Adversarial Reviewer** (`agents/adversarial-reviewer.md`) performs a final independent red-team pass on the synthesis.
- **Forecaster** (`agents/forecaster.md`) converts reviewed claims into falsifiable, probabilistic forecasts and logs each one to the run's forecast ledger.
- **Forecast Resolution Auditor** (`agents/forecast-resolution-auditor.md`) is invoked manually and independently, later, to check one forecast-ledger entry against reality. It is not part of the default per-run pipeline.
- **Writer** and **Editor** (`agents/writer.md`, `agents/editor.md`) produce and audit the research report.
- **Applied Analyst** (`agents/applied-analyst.md`) is downstream only, translating frozen findings into the application or decision requested by the charter.

## Research workflow

The default workflow is:

1. The Prompt Architect writes a charter with one primary question, explicit sub-questions, source requirements, and a distinct panel roster.
2. The Research Director creates one independent Panel Member run per lens. Ten members means ninety directed peer reviews in the all-versus-all round.
3. Each Panel Member researches without seeing the other members' conclusions, retrieves sources, and writes a memo containing a thesis, counter-thesis, evidence ledger, uncertainty, and falsifiable predictions.
4. Each member reviews every other memo. Coverage stays exhaustive, but depth varies: the Research Director routes the full rubric to pairs with genuine overlap or conflict and allows a lighter-touch coherence check for orthogonal pairs. Every review must identify the strongest point, the most important weakness, missing or contradictory evidence, and a justified grade or coherence judgment.
5. Members respond to their reviews. They may revise or narrow their thesis, but revisions remain visibly separate from the original memo.
6. The Research Director audits participation, source quality, review quality, score distributions, and unresolved disagreements. It identifies the best-supported explanations, but does not select a winner by majority vote alone.
7. The Domain Analyst turns the reviewed panel record into claims and competing scenarios. The Adversarial Reviewer then attacks that synthesis independently.
8. Only after the core research survives review do the Forecaster, Writer,
   Editor, and eventually Applied Analyst proceed.

The Forecast Resolution Auditor is not part of this per-run sequence. It
runs later, independently, and only when a user chooses to check a specific
forecast-ledger entry against reality — a separate, manually triggered pass,
not a step in the default pipeline.

The panel is not a popularity contest. A thesis with the most votes is not automatically the best thesis. The supervisor must distinguish evidence quality, reasoning quality, source independence, reviewer agreement, and unresolved minority objections.

## Source policy

Every agent may retrieve sources, but not every source type has the same evidentiary role:

- **Primary sources:** papers, technical reports, official records, documentation, repositories, datasets, standards, filings, benchmark methodology, and first-party measurements. These carry the core factual burden.
- **Structured measurement sources:** domain-specific databases, registries, archives, benchmarks, and measurement services can provide useful observations when the charter calls for them. Record the item/version, measurement date, methodology, coverage, and limitations; do not treat an aggregator as ground truth without checking its underlying method.
- **Public commentary:** interviews, social posts, expert discussion, and other public commentary can help discover work and hypotheses. Use specificity, original work, relevance, and corroboration as quality signals. Popularity, virality, credentials, or institutional prestige are discovery signals, not evidence by themselves.
- **Secondary reporting:** useful for finding leads and context, but important claims should be checked against primary or structured sources.

When primary sources disagree — for example, a vendor-published benchmark
against an independent reproduction — prefer the independently reproduced or
third-party-verified result. Do not silently pick a side: record the
conflict explicitly in the evidence ledger, including what each source
claims and why one was weighted over the other. This rule applies to any
agent citing sources, not only the Source Auditor.

No agent may cite a source it did not actually retrieve and inspect. Every source note should preserve the URL or identifier, author or organization, publication/post date, retrieval date, relevant excerpt or locator, and the exact claim the source supports. If a source cannot be verified, label it as a lead rather than evidence.