# Deliberation Graph

Deliberation Graph is an agent skill and standard-library Python state manager
for difficult decisions where one plausible approach is not enough. It makes an
agent externalize branches, assumptions, evidence, critiques, scores, and the
final synthesis in a durable project-local run.

The host agent still does the thinking. The scripts provide transactional state,
validation, and readable exports. They do not call a model API, execute hidden
reasoning, or claim that more branches automatically mean a better answer.

## Use It When

- Architecture or migration work has several credible strategies.
- Debugging has competing causal hypotheses.
- A consequential plan needs hard gates and a fallback.
- A research synthesis needs traceable evidence and rejected alternatives.
- The risk of locking onto the first answer is higher than the cost of explicit
  branch work.

Skip it for mechanical edits, one-command lookups, straightforward calculations,
or tasks with one already-proven implementation path.

## What The Package Includes

- `SKILL.md` - the operating contract and trigger guidance.
- `scripts/deliberation.py` - CLI for creating runs, adding branches, moving
  phases, attaching evidence, recording critiques, synthesizing, and reporting.
- `scripts/graph_core.py` - SQLite-backed state and graph implementation.
- `assets/brief.template.json` - starter decision brief.
- `schemas/` - JSON schemas for brief, branch, run, graph, and synthesis data.
- `references/` - branch strategy, criticism, evidence, scoring, and
  context-ledger integration notes.
- `scripts/self_test.py` - regression suite for phase gates, graph integrity,
  scoring, exports, and hostile cases.

## Run Model

Each run is stored under the target project:

```text
.deliberation/runs/<run-id>/
├── run.sqlite3
├── run.json
├── graph.json
└── report.md
```

SQLite is the authority. JSON and Markdown files are deterministic exports.

## Minimal Workflow

Create a decision brief from the template:

```bash
cp assets/brief.template.json /tmp/brief.json
$EDITOR /tmp/brief.json
```

Create and advance a run:

```bash
python3 scripts/deliberation.py --project /path/to/project create \
  --run-id storage-architecture \
  --mode deep \
  --brief-file /tmp/brief.json

python3 scripts/deliberation.py --project /path/to/project phase \
  --run-id storage-architecture \
  --target branching \
  --expected-run-revision 0
```

Add branches whose assumptions actually differ:

```bash
python3 scripts/deliberation.py --project /path/to/project add-branch \
  --run-id storage-architecture \
  --strategy "SQLite transactional authority" \
  --distinguishing-assumption "Writers share a local filesystem" \
  --optimization-target "consistency under concurrent writers" \
  --known-tradeoff "single-file database dependency"
```

Use `python3 scripts/deliberation.py --help` for the full lifecycle.

## Install As An Agent Skill

```bash
git clone https://github.com/ap7x42a/deliberation-graph.git
cp -a deliberation-graph ~/.codex/skills/deliberation-graph
```

For project-local skill surfaces, copy the directory into the location your
runtime uses, such as `.agents/skills/deliberation-graph`.

## Verify The Package

```bash
python3 scripts/self_test.py
python3 scripts/write_manifest.py --check
sha256sum -c SHA256SUMS.txt
```

`SHA256SUMS.txt` is a drift manifest, not a provenance signature.

## Limits

This package records externalized decision material: approaches, assumptions,
evidence, critiques, risks, and synthesis. It does not preserve private
chain-of-thought, sandbox execution, or let a score override hard constraints.
