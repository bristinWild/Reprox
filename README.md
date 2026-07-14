# Reprox

> **Persistent memory for coding agents.** Reprox gives Codex a structured, relationship-aware memory of your codebase - recall before action, over MCP.

Built for **OpenAI Build Week** · Track: **Developer Tools** · Built with **Codex + GPT-5.6**

---

## The problem: brilliant engineers with amnesia

Coding agents like Codex are extraordinary - and they wake up every session knowing nothing about your repository. The industry's current answers:

| Approach | What it is | Why it falls short |
| --- | --- | --- |
| **AGENTS.md** | A note file the agent reads on startup | Hand-maintained, goes stale the day after it's written, holds about a page. A sticky note, not a memory. |
| **Fresh exploration (grep/read)** | The agent re-scans the repo every session | This is *rediscovery, not memory*. It burns time and tokens re-deriving the architecture on every task - and it cannot recover what isn't in the working tree: the history, the decisions, the PR context, the known gaps. |

The result: every task starts with the agent re-learning your codebase from scratch, and everything it can't re-learn - *why* the code is the way it is - stays invisible.

## The solution: recall before action

**Reprox** connects coding agents to a persistent, local memory graph of the repository:

- **[Cliper](https://github.com/bristinWild/cliper-sdk)** (`cliper init` / `cliper sync`) builds the memory: the repository is scanned into **13 typed memory objects** - files, commits, pull requests, architecture, dependencies, responsibilities, timeline, and **gaps** (undocumented or risky corners, ranked by severity) - connected by **explicit relationships** (`commit → changed files`, `PR → resolves issue`, `gap → found in file`).
- The **LocalJson provider** stores this graph as plain JSON files under `.cliper/memory/` - fully local, no cloud, no API keys, versionable.
- **Reprox exposes the graph to agents over MCP.** Codex (or any MCP client) gains recall tools: search the memory, list known gaps, walk relationships. The agent starts every task already knowing the codebase - no re-exploration, no stale notes.
- **The Reprox dispatcher** closes the loop: `reprox fix <gap>` briefs Codex with the gap and its related memories, Codex executes the fix locally and commits, and `cliper sync` updates the memory. Memory briefs the agent; the agent changes the code; the memory re-learns.

Code never leaves your machine. Only knowledge moves.

```
┌─────────────────────────── your laptop ───────────────────────────┐
│                                                                    │
│  git repo ──cliper init/sync──▶ .cliper/memory/*.json              │
│                                   (13 memory types + relationships)│
│                                        │                           │
│                                        ▼                           │
│                              Reprox MCP server                     │
│                     search_memory · list_gaps · get_related        │
│                                        │                           │
│                                        ▼                           │
│   reprox fix <n> ──briefs──▶  Codex (GPT-5.6)  ──edits + commit──▶ repo
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

## MCP tools

| Tool | What it returns |
| --- | --- |
| `search_memory(query)` | Ranked recall over the repository's memory graph |
| `list_gaps()` | Known gaps (undocumented/risky areas), ranked by severity |
| `get_gap(id)` | One gap with full context |
| `get_related(memory_id)` | Every memory connected to this one, via explicit relationships |
| `get_architecture()` | Module boundaries and structural memories |
| `get_timeline()` | How the repository evolved - commits, releases, PRs |

## Quickstart

### 1. Build the memory (once per repo)

```bash
npm install -g cliper-memory
cliper auth local-json          # zero-dependency local provider - no accounts
cd your-repo
cliper init                     # scans the repo → .cliper/memory/
```

### 2. Plug Reprox into Codex

```bash
npm install -g reprox
```

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.reprox]
command = "reprox"
args = ["mcp"]
```

Codex now has memory. Ask it anything about the repo - it recalls instead of re-exploring.

### 3. Fix what the codebase knows is broken

```bash
reprox gaps                     # list known gaps, ranked
reprox fix 3                    # brief Codex with gap #3 + related memories
                                # → review diff → approve → local commit
cliper sync                     # memory re-learns the change (~2s if unchanged)
```

Safety defaults: requires a clean working tree, always shows the diff before applying, `--dry-run` supported.

## For judges - test without rebuilding

1. Clone this repo and run the Quickstart against the included `sample-repo/` (pre-initialized - `.cliper/memory/` is committed there so no Cognee/network setup is needed).
2. Side-by-side check: ask Codex "what are the riskiest undocumented parts of this repo?" with and without the Reprox MCP server configured. Without: exploration and guesses. With: ranked gap memories, instantly.
3. `reprox fix 1 --dry-run` shows the full brief → plan → diff pipeline without writing.

## How we built it with Codex + GPT-5.6

<!-- REQUIRED by hackathon rules - fill with your real session details before submitting -->
- **Codex sessions**: the MCP server, memory-graph loader, and dispatcher were built in Codex (primary session ID via `/feedback`: `<SESSION_ID>`). Codex accelerated <specific examples: tool schema scaffolding, MCP stdio plumbing, test generation>.
- **Key decisions made with Codex**: <e.g. tool granularity (6 focused tools vs 1 mega-tool), brief format for the dispatcher, clean-tree safety gate>.
- **GPT-5.6 at runtime**: the dispatcher's task briefs are executed by Codex running GPT-5.6; gap-fix planning quality is a direct function of the model.

## Prior work vs. new work (hackathon scope)

- **Pre-existing** (before the submission period): the Cliper CLI/SDK and its Cognee cloud provider - [bristinWild/cliper-sdk](https://github.com/bristinWild/cliper-sdk), public commit history.
- **Built during the submission period, with Codex**: the LocalJson provider completion (explicit provider selection, offline mode - `cliper-memory@0.2.x`), the entire Reprox MCP server, the dispatcher, and this repository. All dated commits + Codex session logs available.

## About Cliper

Cliper is the open-source memory layer Reprox builds on (`npm i -g cliper-memory`, MIT). It treats a repository the way a senior engineer's head does - not as text, but as typed knowledge with relationships - and keeps that knowledge alive with content-hashed incremental sync (an unchanged repo syncs in ~2 seconds with zero API calls). The same memory graph already powers a Slack agent, a mobile app, and a web app; Reprox brings it to the surface that matters most: the coding agent itself.

*Git stores source code. Cliper stores engineering knowledge. Reprox hands it to your agent.*

## Roadmap

- Remote dispatch: trigger `reprox fix` from Slack/web/mobile (the Cliper agent bridge already carries live presence)
- Memory write-back: agents recording their own decisions as memories
- Multi-provider recall (Neo4j, pgvector, Supermemory - community providers in progress)

## License

MIT
