# ResearchOS

An agent-driven research system. You describe a research question, an
agent turns it into a rigorous research charter, and a team of
specialist agent roles carries out the research and writes the report.

## Structure

- `agents/` — role-definition files Claude Code reads and follows when
  acting as that role.

## Agents

- **Prompt Architect** (`prompt-architect.md`) — turns a rough request
  into a rigorous charter; designs the specialist research organization
  for the topic.
- **Research Director** (`research-director.md`) — coordinates every
  other agent, decides what happens next, gates each stage.
- **Librarian** (`librarian.md`) — finds real sources and extracts
  evidence for an assigned scope.
- **Domain Analyst** (`domain-analyst.md`) — reasons from evidence to
  claims and hypotheses, counter-evidence included.
- **Adversarial Reviewer** (`adversarial-reviewer.md`) — red team;
  attacks claims independently, never rewrites them.
- **Forecaster** (`forecaster.md`) — turns reviewed claims into
  falsifiable, probabilistic forecasts.
- **Writer** (`writer.md`) — queries the graph and drafts the report.
- **Editor** (`editor.md`) — independent review of the draft against the
  graph; explicit pass/revise verdict.
- **Investment Analyst** (`investment-analyst.md`) — downstream only;
  translates frozen forecasts into a layer-then-company investment view.

## How this works

Research and reasoning agents (Librarian → Domain Analyst → Adversarial
Reviewer → Forecaster) *build* the graph — a set of linked entries for
sources, evidence, claims, challenges, and forecasts. Report agents
(Writer → Editor) *query* that graph to produce the report; they never
create research content themselves. The Investment Analyst is a further
downstream query stage, reading only forecasts, never raw evidence or
claims directly. The Research Director sits above all of it, deciding
what gets worked on next and when a stage is actually ready to hand off
to the next one.
