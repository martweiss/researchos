# ResearchOS

ResearchOS is a Markdown-defined research panel. A frontier model acts as
the orchestrator; the Markdown files define the specialist roles, source
standards, debate protocol, and handoffs.

The system is designed for questions where the answer should emerge from
independent technical research and adversarial comparison, not from one
model producing a polished first opinion. The initial example is the
long-horizon question of where AI inference and intelligence will live:
centralized data centers, edge/on-device systems, or a hybrid of the two.

## Agent layers

- **Prompt Architect** (`agents/prompt-architect.md`) turns a rough question into a charter and a topic-specific research panel.
- **Research Director** (`agents/research-director.md`) runs the panel, controls independence, assigns reviews, and supervises synthesis.
- **Panel Member** (`agents/panel-member.md`) is the reusable contract instantiated once per specialist lens. Every member researches independently and later reviews every other member.
- **Source Auditor** (`agents/source-auditor.md`) checks that important sources are real, retrievable, correctly represented, and appropriately weighted.
- **Peer Reviewer** (`agents/peer-reviewer.md`) defines the all-versus-all grading and challenge protocol used by every panel member.
- **Librarian** (`agents/librarian.md`) supports source discovery, extraction, coverage checks, and evidence hygiene across the panel.
- **Domain Analyst** (`agents/domain-analyst.md`) reconciles the panel into explicit technical claims without erasing minority views.
- **Adversarial Reviewer** (`agents/adversarial-reviewer.md`) performs a final independent red-team pass on the synthesis.
- **Forecaster** (`agents/forecaster.md`) converts reviewed claims into falsifiable, probabilistic forecasts.
- **Writer** and **Editor** (`agents/writer.md`, `agents/editor.md`) produce and audit the technical report.
- **Investment Analyst** (`agents/investment-analyst.md`) is downstream only, translating frozen forecasts into structural implications.

## Research workflow

The default workflow is:

1. The Prompt Architect writes a charter with one primary question, explicit sub-questions, source requirements, and a distinct panel roster.
2. The Research Director creates one independent Panel Member run per lens. Ten members means ninety directed peer reviews in the all-versus-all round.
3. Each Panel Member researches without seeing the other members' conclusions, retrieves sources, and writes a memo containing a thesis, counter-thesis, evidence ledger, uncertainty, and falsifiable predictions.
4. Each member reviews every other memo using the same rubric. Reviewers must identify the strongest point, the most important weakness, missing or contradictory evidence, and a justified grade.
5. Members respond to their reviews. They may revise or narrow their thesis, but revisions remain visibly separate from the original memo.
6. The Research Director audits participation, source quality, review quality, score distributions, and unresolved disagreements. It identifies the best-supported explanations, but does not select a winner by majority vote alone.
7. The Domain Analyst turns the reviewed panel record into claims and competing scenarios. The Adversarial Reviewer then attacks that synthesis independently.
8. Only after technical claims survive review does the Forecaster, Writer, Editor, and eventually Investment Analyst proceed.

The panel is not a popularity contest. A thesis with the most votes is not automatically the best thesis. The supervisor must distinguish evidence quality, reasoning quality, source independence, reviewer agreement, and unresolved minority objections.

## Source policy

Every agent may retrieve sources, but not every source type has the same evidentiary role:

- **Primary sources:** papers, technical reports, model cards, release notes, documentation, repositories, benchmark methodology, standards, filings, and first-party measurements. These carry the core factual burden.
- **Structured measurement sources:** OpenRouter, Artificial Analysis, and similar systems can provide model availability, pricing, latency, throughput, benchmark, and usage signals. Record the measurement date, model/version, methodology, and limitations; do not treat an aggregator as ground truth without checking its underlying method.
- **Expert social sources:** X/Twitter and similar public commentary are valuable for discovering current work, deployment clues, and expert hypotheses. Use author expertise, original work, specificity, and corroboration as quality signals. Likes, followers, virality, institutional prestige, or a famous affiliation are discovery signals, not evidence by themselves.
- **Secondary reporting:** useful for finding leads and context, but important claims should be checked against primary or structured sources.

No agent may cite a source it did not actually retrieve and inspect. Every source note should preserve the URL or identifier, author or organization, publication/post date, retrieval date, relevant excerpt or locator, and the exact claim the source supports. If a source cannot be verified, label it as a lead rather than evidence.

## Current scope

The current work is intentionally focused on the Markdown agent contracts and research protocol. Storage, embeddings, graph traversal, and the frontend come later. The agent outputs should still be structured and provenance-aware so they can be placed into the future graph without changing the research method.
