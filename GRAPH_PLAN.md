# ResearchOS Knowledge Graph and Retrieval Plan

**Status:** premium architecture proposal; implementation follows the dry run  
**Decision:** use a hybrid knowledge system, not a graph-versus-embeddings choice  
**Canonical project scope:** `projects/<project-slug>/`  

This document defines how ResearchOS should retain research so that future
agents can retrieve evidence, test competing theses, and produce a citation-
complete dissertation-style LaTeX report. It is intentionally more ambitious
than the first Markdown-only sketch. Markdown remains the human-facing work
surface; durable machine retrieval and provenance require a database, object
storage, and an embedding index.

## 1. Architecture decision

The premium default is:

1. **Markdown and Git** for agent assignments, memos, reviews, rebuttals,
   synthesis, forecasts, and reports. These are the reviewable research
   artifacts and the audit trail of how a run reasoned.
2. **Managed PostgreSQL** as the canonical structured store. It holds source
   metadata, evidence cards, claims, forecasts, reviews, citations, run
   metadata, and a typed `edges` table. PostgreSQL is the authority for
   identity, status, versioning, and relationships.
3. **S3-compatible object storage** for immutable source snapshots, downloaded
   PDFs, HTML captures, extracted text, rendered pages, figures, and generated
   report assets. Store only material that ResearchOS is permitted to retain;
   otherwise store metadata, a locator, a hash, and permitted excerpts.
4. **`pgvector` inside PostgreSQL** for semantic retrieval. This keeps vector
   search, metadata filters, graph joins, and citation provenance in one
   transactional system. A separate vector service is an optimization option,
   not a second source of truth.
5. **OpenAI `text-embedding-3-large`** as the premium canonical embedding model,
   using its full vector size initially. The embedding model name, dimensions,
   preprocessing version, and creation date are stored with every vector so
   the corpus can be re-embedded without ambiguity.
6. **Hybrid retrieval**: PostgreSQL full-text/BM25-style lexical search plus
   vector similarity plus graph expansion. Embeddings find semantically related
   passages; lexical search protects exact names, numbers, equations, model
   versions, and citations; graph traversal explains why a passage matters.
7. **Optional hosted retrieval mirror** for rapid experiments. OpenAI File
   Search can provide managed semantic and keyword search over vector stores,
   with metadata filtering, but it is not the canonical ResearchOS graph. It
   cannot replace the explicit evidence lineage and multi-agent review records.

This is a premium design because it optimizes for evidence quality,
reproducibility, and long-lived research rather than minimum infrastructure.
The first implementation can still run locally with PostgreSQL and a local
object-store-compatible directory, then move the same schema to managed
services.

### Why not choose only one representation?

| Representation | Best at | Cannot safely provide alone |
|---|---|---|
| Markdown/Git | Human review, prompts, reasoning, diffs, reproducible run artifacts | Fast corpus-wide retrieval and reliable referential integrity |
| Relational database | Identity, versioning, filters, joins, status, auditability | Meaning-based discovery without search indexes |
| Typed graph edges | Support, contradiction, derivation, challenge, and citation lineage | Similarity search over unfamiliar wording |
| Embeddings | Semantic recall and discovery across differently worded passages | Exact provenance, truth, chronology, or logical entailment |
| Lexical search | Exact terms, identifiers, numbers, names, and quoted language | Synonyms, paraphrase, and conceptual similarity |

The graph answers **“what supports, challenges, or derives this?”**. The
embedding index answers **“what else is semantically relevant?”**. The writer
needs both, plus exact lexical search and source metadata.

## 2. Canonical data flow

Every research project follows this path. The durable store is populated as a
run progresses; it is not created only after the final report.

```text
User request
    |
    v
Prompt Architect -> approved charter -> project/run manifest
    |
    v
Source acquisition -> source version -> permitted snapshot + hash
    |
    v
Evidence extraction -> atomic evidence cards -> embeddings + lexical index
    |
    v
Panel theses -> claims -> support/contradiction/derivation edges
    |
    v
All-versus-all reviews -> challenges, grades, rebuttals, reviewer edges
    |
    v
Supervisor + red team -> resolutions, confidence, forecast records
    |
    v
Retrieval packet per report section -> MODEL writer -> citations/BibTeX
    |
    v
Citation/provenance validator -> LaTeX build -> report artifact + manifest
```

The filesystem is append-oriented. Agents should create new versions or
status transitions rather than silently overwriting a source interpretation,
claim, review, or report. The database mirrors those artifacts and adds
machine-enforced identity and relationships.

## 3. Project and run layout

The project folder remains the human-readable boundary. The Prompt Architect
creates it after the user approves the charter; no topic-specific project
folder is pre-created by the repository scaffold.

```text
projects/<project-slug>/
├── project.md
├── intake/
│   ├── rough-request.md
│   ├── charter-draft.md
│   └── charter-approved.md
├── runs/
│   └── run-<id>/
│       ├── manifest.md
│       ├── assignments/
│       ├── raw/
│       │   ├── source-ledger.md
│       │   └── source-captures/       # permitted local copies or pointers
│       ├── panel/<lens>/
│       │   ├── assignment.md
│       │   ├── memo.md
│       │   └── rebuttal.md
│       ├── reviews/<reviewer>/<target>.md
│       ├── audits/
│       ├── synthesis/
│       ├── forecasts/
│       ├── reports/
│       │   ├── evidence-packets/
│       │   ├── citations.bib
│       │   ├── report.tex
│       │   └── report.pdf
│       └── exports/                    # database/search exports when needed
└── README.md                           # optional project-specific guide
```

The database stores a pointer to each artifact, its content hash, and its
parsed records. The file remains the readable source for agent work; the
database is the queryable canonical representation after ingestion. A failed
ingestion must be visible in the run manifest and must not result in an
apparently complete graph.

## 4. Data model

Use stable, time-sortable IDs (ULID or UUIDv7) with a readable type prefix,
for example `SRC_01J...`, `EVID_01J...`, and `CLM_01J...`. Do not use a
per-type sequential counter: concurrent agents and resumed runs make it a
coordination hazard.

Every record carries these common fields, either in typed columns or a
validated JSONB metadata field:

```yaml
id: CLM_01J...
project_id: PRJ_01J...
run_id: RUN_01J...
record_type: claim
artifact_path: projects/example/runs/run-001/panel/systems/memo.md
artifact_hash: sha256:...
version: 1
status: active | superseded | disputed | rejected | draft
created_at: 2026-07-18T00:00:00Z
created_by_agent: domain-analyst
model: MODEL-selected identifier
provider: MODEL provider or CLI surface
execution_mode: isolated | directed_same_runtime | human_assisted
```

### Core tables / node types

| Record | Purpose | Important fields |
|---|---|---|
| `projects` | Long-lived research question and scope | slug, title, charter status, domain, owner, created date |
| `runs` | One execution of a charter | run ID, charter hash/version, MODEL, provider, execution mode, start/end, status |
| `agents` | Role/lens contract used in a run | agent file, lens, prompt hash, MODEL, retrieval policy |
| `sources` | Work-level identity | title, authors, publisher, source class, DOI/URL, publication date |
| `source_versions` | Version/retrieval identity | source ID, retrieved date, locator, snapshot URI, content hash, extraction version, rights status |
| `evidence` | Atomic source-grounded observation | passage, page/section, source version, proposition, limitation, evidence class |
| `claims` | A thesis or proposition asserted by an agent | statement, scope, confidence, falsifier, current status |
| `challenges` | Adversarial objection or alternative explanation | objection, target claim, severity, resolution status |
| `resolutions` | Response to a challenge | response, residual uncertainty, resolver, supporting evidence |
| `reviews` | One reviewer grading one target | reviewer lens, target claim/memo, rubric scores, rationale, bias notes |
| `rebuttals` | Target agent response to reviews | review set, response, accepted/rejected objections, revised claim IDs |
| `forecasts` | Time-bounded, falsifiable prediction | horizon, probability, base rate, resolution criteria, indicators |
| `reports` | Generated artifact and section manifest | report path, run ID, LaTeX hash, status, citation coverage |
| `citations` | Report-to-source/evidence mapping | report section, citation key, source version, page/locator, evidence IDs |
| `retrieval_events` | What context was supplied to a MODEL | query, filters, result IDs, ranking method, timestamp, prompt hash |
| `embeddings` | Vector index for a retrievable object | object ID, model, dimensions, vector, text hash, preprocessing version |

### Evidence is the unit of retrieval

Do not embed only whole papers or whole agent memos. Create atomic, bounded
evidence cards with:

- the exact proposition or observation;
- the source version and page, section, timestamp, table, or figure locator;
- a short context window where needed;
- source class and independence/ecosystem tags;
- explicit limitations and whether the item is fact, measurement, quotation,
  interpretation, or forecast;
- links to the claims, challenges, and reports that use it.

One source can produce many evidence cards. One claim can cite many evidence
cards. A card may be embedded, but its vector never becomes the evidence of
record; the text, locator, source version, and graph lineage remain required.

## 5. Edge vocabulary and rules

Store edges in a first-class table rather than hiding all relationships in
frontmatter arrays. Each edge includes `edge_id`, `from_id`, `edge_type`,
`to_id`, `project_id`, `run_id`, `created_by`, `created_at`, `confidence`, and
an optional rationale or evidence locator.

Initial closed vocabulary:

| Edge | Meaning |
|---|---|
| `belongs_to` | record belongs to a project, run, panel, or report |
| `version_of` | record is a new version of an earlier record |
| `supersedes` | newer record replaces an earlier interpretation |
| `cites` | artifact or report cites a source/version |
| `extracts_from` | evidence was extracted from a source version |
| `supports` | evidence supports a claim or forecast |
| `contradicts` | evidence or claim conflicts with another claim |
| `derived_from` | claim/forecast/implication depends on upstream records |
| `challenges` | review or challenge attacks a target |
| `responds_to` | rebuttal or resolution addresses a challenge/review |
| `resolves` | evidence or event provides a forecast resolution |
| `reviewed_by` | target was assessed by a reviewer/agent |
| `used_in` | evidence or claim was included in a report section |
| `generated_by` | artifact was produced by an agent/MODEL execution |
| `revises` | a later synthesis changes the interpretation without deleting history |

Minimum integrity rules:

1. Every `evidence` record has exactly one `extracts_from` source version.
2. Every material `claim` has at least one supporting or explicitly absent
   evidence link and a counter-evidence field.
3. Every `challenge` targets a claim, forecast, or reviewable thesis.
4. Every `forecast` has a horizon, probability, resolution criteria, and
   falsifying indicators.
5. Every report citation resolves to a source version and, where possible, an
   evidence card with a locator.
6. `investment_implication` records may derive from forecasts and claims only
   after the technical gate; they must not silently turn a weak source into a
   company conclusion.
7. Superseded records remain queryable. “Current” is a status/filter, never a
   destructive delete.

## 6. Premium embedding and search design

### Canonical embedding choice

Use [OpenAI `text-embedding-3-large`](https://developers.openai.com/api/docs/models/text-embedding-3-large)
for the canonical corpus. It is the more capable embedding model and provides
3072 dimensions by default. The embedding guide also supports shortening the
dimensions, but the premium first pass should retain full dimensions until
retrieval evaluations demonstrate that reduction is harmless.

The economical fallback is
[`text-embedding-3-small`](https://developers.openai.com/api/docs/models/text-embedding-3-small),
which is substantially cheaper and smaller. It should be treated as a
separate, explicitly versioned index or an offline triage index, not mixed
silently with large-model vectors.

The linked model pages currently list embedding input at $0.13 per million
tokens for `text-embedding-3-large` and $0.02 per million tokens for
`text-embedding-3-small`; pricing is operational metadata and must be checked
again when implementation begins. For ResearchOS, the quality and provenance
benefit of the large model justifies using it for the canonical evidence index;
small can reduce exploratory or re-embedding cost.

Store one row per embedded object, not one vector per source file:

```yaml
embedding_id: EMB_01J...
object_id: EVID_01J...
object_type: evidence
model: text-embedding-3-large
dimensions: 3072
text_hash: sha256:...
preprocessing_version: evidence-card-v1
created_at: 2026-07-18T00:00:00Z
```

Embed these objects first:

- evidence cards and bounded source passages;
- claims and counter-theses;
- challenges, rebuttals, and resolutions;
- forecasts and falsification criteria;
- methodology notes and report section questions.

Embed whole documents only as a navigation aid. Retrieval should return
atomic evidence objects with their source and graph context, not an
unbounded paper dump.

### Retrieval algorithm

For every report section or agent question:

1. Convert the section into explicit sub-questions and target claim types.
2. Apply hard filters first: project, run/status, date window, source class,
   geography, workload, model family, and permitted rights status.
3. Run lexical search for exact names, model versions, numbers, equations,
   and quoted phrases.
4. Run `pgvector` similarity search over evidence, claims, and challenges.
5. Fuse lexical and vector rankings, then diversify by source, author,
   institution, ecosystem, geography, and stance so one vendor or paper family
   cannot fill the context window.
6. Expand one or two graph hops through `supports`, `contradicts`,
   `derived_from`, `challenges`, and `responds_to`.
7. Rerank the candidate packet with the selected MODEL using structured
   fields: relevance, directness, source quality, independence, recency,
   contradiction value, and locator completeness.
8. Record the complete `retrieval_event`: query, filters, candidate IDs,
   selected IDs, ranking method, MODEL, and prompt hash.
9. Give the report writer a bounded evidence packet, not unrestricted database
   access. Every material paragraph must carry source/evidence IDs internally
   before LaTeX citation keys are rendered.

This is hybrid retrieval augmented by a graph, not “retrieve nearest vectors
and ask for an essay.” The graph is what lets the supervisor retrieve the
strongest opposing thesis and see whether a conclusion rests on five
independent sources or five documents from one ecosystem.

### Hosted File Search

[OpenAI File Search](https://developers.openai.com/api/docs/guides/tools-file-search)
is useful for a quick managed prototype because it combines semantic and
keyword search over vector stores and supports metadata filtering. ResearchOS
should treat it as an optional mirror for exploratory runs. The PostgreSQL /
object-store system remains authoritative for source versions, graph edges,
review lineage, rights, and reproducible report assembly.

## 7. Dissertation-style report pipeline

The report writer should never begin from the entire corpus. It should receive
section-specific evidence packets produced from a frozen run state.

```text
Research question
  -> section outline and claim inventory
  -> evidence packet per section
  -> claim/evidence matrix
  -> MODEL drafts section with inline evidence IDs
  -> citation resolver creates BibTeX and locator notes
  -> citation/provenance validator
  -> LaTeX compile and PDF checks
  -> immutable report artifact and report manifest
```

Required report controls:

- every material factual sentence maps to one or more evidence/source IDs;
- every strong conclusion exposes counter-evidence and residual uncertainty;
- citations resolve to the exact source version used, not merely a homepage;
- the bibliography is generated from the source table, not hand-entered;
- forecast and investment sections are downstream of the technical synthesis;
- unsupported or uncited sentences are reported before compilation;
- report generation records the MODEL, retrieval events, prompt hashes, data
  cutoff, and corpus version;
- the PDF, `.tex`, `.bib`, evidence packets, and manifest share content hashes.

The final report can be written in Markdown first, then rendered to LaTeX.
Use a deterministic LaTeX toolchain such as `tectonic` or `pdflatex` in CI or
the project environment. A report is not complete merely because the PDF
compiles: citation coverage, dangling evidence IDs, and unresolved claims must
also pass validation.

## 8. Source and provenance policy

The source ledger is an index, not the entire evidence store. For every
retrieved source, record:

- canonical locator, title, author/organization, publication date, and source
  class;
- retrieval timestamp and data cutoff;
- snapshot URI when storage is permitted;
- content hash and extraction/parser version;
- page, section, table, figure, timestamp, or line locator for each evidence
  card;
- rights/retention status and any redaction;
- whether the source is primary, first-party, benchmark, academic review,
  expert commentary, social lead, or market data;
- known ecosystem, author, vendor, or dataset correlations;
- what the source does not establish.

Social posts and high-engagement commentary are discovery leads and context,
not automatically evidence. A claim found on social media must be promoted
only when it is corroborated by a primary paper, first-party document,
reproducible measurement, or clearly labeled expert interpretation.

Source snapshots are append-only. If a live page changes, create a new source
version and preserve the old hash. Dynamic prices, model leaderboards,
hardware availability, and usage statistics must carry an as-of date and be
refreshed before an investment conclusion.

## 9. Quality and evaluation gates

Before relying on the datastore for a final report, maintain a small labeled
evaluation set of research questions with expected relevant evidence and known
counter-evidence. Track:

- retrieval recall at 5, 10, and 20;
- exact-source and exact-locator recall;
- support/contradiction recall;
- source and ecosystem diversity of each evidence packet;
- percentage of report claims with direct evidence;
- unsupported-sentence rate and citation coverage;
- stale-source rate and failed snapshot rate;
- dangling-edge and duplicate-identity counts;
- reviewer agreement, reviewer calibration, and same-MODEL correlation;
- forecast calibration after outcomes resolve.

The all-versus-all protocol should preserve the distinction between **number
of review records** and **number of independent minds or execution contexts**.
Every run therefore records `execution_mode`, `independence_level`, and
`reviewer_model/provider`. A 90-file review matrix generated in one runtime is
valuable structured criticism, but it is not equivalent to 90 independent
reviewers.

## 10. Premium versus economical deployment

| Component | Premium default | Economical fallback |
|---|---|---|
| Structured store | Managed PostgreSQL with automated backups, replicas, encryption, and point-in-time recovery | Local PostgreSQL or SQLite for a small single-user pilot |
| Graph representation | Typed relational `edges` table plus graph views/materialized traversals | Frontmatter links with a periodic integrity script |
| Raw source storage | Versioned S3-compatible object storage with lifecycle and checksum policies | Local `source-captures/` where retention is permitted |
| Vector index | `pgvector` in the canonical PostgreSQL database, full precision as evaluated | Local vector index or reduced dimensions |
| Embedding model | `text-embedding-3-large`, full 3072 dimensions initially | `text-embedding-3-small` or reduced dimensions |
| Retrieval | Lexical + vector fusion, graph expansion, diversity reranking, retrieval logs | Vector search plus simple metadata filters |
| Report generation | Section-level evidence packets, citation resolver, validator, LaTeX build, PDF QA | Single Markdown report with manual citations |
| Backups/evaluation | Point-in-time recovery, immutable exports, labeled retrieval set, scheduled regression tests | Git history and occasional database dump |

The premium architecture does not require every premium component on day one.
The schema and artifact contracts should be designed for it now so that a
cheap pilot can migrate without changing agent prompts or research identity.

## 11. Migration of the completed initial run

The completed `edge-vs-datacenter-ai/run-001` should be treated as a valuable
pipeline fixture and migration test, not as a pristine independent panel.

Migration steps:

1. Ingest the project, approved charter, run manifest, agent assignments,
   source ledger, panel memos, 90 directed reviews, rebuttals, audits,
   synthesis, forecasts, and reports as immutable artifacts.
2. Parse source records into `sources` and `source_versions`; add permitted
   captures and hashes when available. A ledger entry without a capture is
   retained with `provenance_status: locator_and_ledger_only`.
3. Split source-grounded observations from memos into evidence cards and attach
   page/section locators wherever the original source supports them.
4. Convert thesis statements into claims and objections into challenges. Link
   review grades to the reviewer, target, rubric, and execution context.
5. Add all explicit graph edges and run link-integrity checks. Preserve the
   original Markdown even when parsed records require a corrected version.
6. Embed evidence cards, claims, challenges, rebuttals, forecasts, and report
   questions using the selected embedding index version.
7. Re-run retrieval evaluation against questions from the technical report and
   check whether the same conclusions survive source diversification.
8. Mark the run metadata as `directed_same_runtime`, `independence_level:
   limited`, and `source_capture_status: ledger_only` until those facts change.

This migration should not rewrite the historical conclusions. It should make
their evidence lineage and limitations queryable.

## 12. Implementation sequence

### Phase 1 — schema and ingestion

- Add a local `pyproject.toml` managed with `uv`.
- Define PostgreSQL migrations for projects, runs, agents, sources,
  source_versions, evidence, claims, challenges, reviews, forecasts, reports,
  citations, edges, retrieval_events, and embeddings.
- Add a Markdown frontmatter parser and an idempotent importer keyed by file
  path plus content hash.
- Add link-integrity, duplicate-ID, and provenance-status reports.

### Phase 2 — source and evidence pipeline

- Add source snapshot adapters and rights-aware retention metadata.
- Extract evidence cards with locators and limitations.
- Add OpenAI embedding generation with batching, retries, cost logging,
  model/dimension versioning, and a re-embedding command.
- Load `pgvector` and PostgreSQL full-text indexes.

### Phase 3 — hybrid retrieval and review

- Implement filtered lexical/vector fusion and graph expansion.
- Add source/ecosystem diversity and contradiction-aware reranking.
- Persist retrieval events and evidence packets.
- Make every panel/reviewer prompt consume a recorded evidence packet.

### Phase 4 — report compiler

- Build claim/evidence matrices per report section.
- Resolve citation keys and locators from the source table.
- Generate Markdown and LaTeX report artifacts.
- Add unsupported-claim, dangling-citation, and PDF build checks.

### Phase 5 — scale only when demonstrated

- Add a separate graph database read projection only if relational graph
  queries become a measured bottleneck.
- Add a dedicated search engine only if PostgreSQL hybrid search fails the
  evaluation set at the corpus scale ResearchOS actually reaches.
- Never introduce a second canonical store through convenience or novelty.

## 13. Remaining decisions

The major architecture decision is resolved: per-project artifacts plus a
canonical PostgreSQL/pgvector/object-storage system, with typed graph edges and
premium embeddings. Remaining implementation decisions are intentionally
smaller:

1. Which managed PostgreSQL and object-storage provider best fits deployment,
   budget, and data-retention requirements?
2. Which source formats may ResearchOS retain in full under the intended use?
3. What exact reviewer rubric and calibration set should govern all-versus-all
   grades?
4. Which LaTeX template and citation style should the first report compiler
   target?
5. What corpus size and retrieval-evaluation threshold would justify a separate
   graph or search service?
