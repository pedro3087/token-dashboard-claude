# Token Dashboard

A local dashboard for Claude Code token usage, costs, and session history. Reads the JSONL transcripts Claude Code writes to `~/.claude/projects/` and turns them into per-prompt cost analytics, tool/file heatmaps, subagent attribution, cache analytics, project comparisons, and a rule-based tips engine.

**Everything runs locally.** No data leaves your machine — no telemetry, no API calls for user data, no login required.

## Prerequisites

- **Python 3.8+** (stdlib only — no `pip install` step)
- **Claude Code**, already installed and with at least one session on your machine. The dashboard reads the JSONL transcripts Claude Code writes to your home directory.

## Quick start

```bash
git clone https://github.com/<your-handle>/token-dashboard.git
cd token-dashboard
python cli.py dashboard
```

Opens http://localhost:8080 in your browser. The first run does a full scan of every JSONL under `~/.claude/projects/` (this can take a minute if you have a lot of history), then the dashboard refreshes every 30 seconds. Stop with `Ctrl+C`.

## Where the data comes from

Claude Code writes one JSONL file per session here:

| OS | Path |
|---|---|
| macOS / Linux | `~/.claude/projects/<project-slug>/<session-id>.jsonl` |
| Windows | `C:\Users\<you>\.claude\projects\<project-slug>\<session-id>.jsonl` |

The dashboard never modifies those files — it only reads them and keeps a local SQLite cache at `~/.claude/token-dashboard.db`.

To point at a different location (useful if your JSONLs live elsewhere or you're moving this to a lab machine):

```bash
python cli.py dashboard --projects-dir /path/to/projects --db /path/to/cache.db
```

or via environment variables:

```bash
CLAUDE_PROJECTS_DIR=/path/to/projects TOKEN_DASHBOARD_DB=/path/to/cache.db python cli.py dashboard
```

## CLI reference

```bash
python cli.py scan          # populate / refresh the local DB, then exit
python cli.py today         # today's totals (terminal)
python cli.py stats         # all-time totals (terminal)
python cli.py tips          # active suggestions (terminal)
python cli.py dashboard     # scan + serve the UI at http://localhost:8080

# dashboard flags
python cli.py dashboard --no-open   # don't auto-open the browser
python cli.py dashboard --no-scan   # skip the initial scan (use cached DB only)
```

Change the port by setting `PORT=9000` in the environment.

## What you'll see

- **Overview** — all-time input/output/cache tokens, sessions, turns, estimated cost on your chosen plan.
- **Prompts** — your most expensive user prompts, drill-down into the assistant response, tool calls, and result sizes.
- **Sessions** — turn-by-turn view of any session.
- **Projects** — per-project comparison (tokens, sessions, active files).
- **Skills** — which skills you invoke most, how often.
- **Tips** — rule-based suggestions for reducing token usage (e.g. repeated file reads, oversized tool results, low cache hit rate).
- **Settings** — switch pricing between API / Pro / Max / Max-20x.

## Troubleshooting

**"No data" or empty charts.** Run `python cli.py scan` once to populate the DB, then reload the dashboard. First scan can take a while.

**Port 8080 already in use.** `PORT=9000 python cli.py dashboard` (or any free port).

**Numbers look wrong / stuck.** The DB lives at `~/.claude/token-dashboard.db`. Delete it and re-run `python cli.py scan` to rebuild from scratch.

**Running the dashboard twice at the same time.** Don't — both processes will fight over the SQLite DB. Stop all instances before starting a new one.

## Accuracy note

Claude Code writes each assistant response 2-3 times to disk while it streams (the same API message gets snapshotted as output grows). The dashboard dedupes these by `message.id` so the final tally matches what the API actually billed. If you compare against another tool that sums every JSONL row, expect this dashboard's numbers to be lower — and closer to reality.

## Tech stack

- Python 3 (stdlib only) for the CLI, scanner, and HTTP server
- SQLite for the local cache
- Vanilla JS + ECharts for the UI, no build step
- Dark theme, hash-based router, server-sent events for live refresh

## License

MIT.
