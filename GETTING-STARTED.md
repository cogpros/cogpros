# Getting Started with the Cogpros Tools

Four tools, four entry points. Pick the problem you have. Each section gives you the problem, the install, the first run, and what to add next. All install steps come from each repo's README.

---

## Path A: "I run a multi-session Claude Code operation"

**Tool: checkpoint** (github.com/cogpros/checkpoint)

### The problem

Long AI working sessions accumulate state faster than working memory holds it. "Where am I, what did I just do, what's next" fires constantly, and answering it means scrolling the transcript or asking three separate questions.

### Install

```bash
mkdir -p ~/.claude/skills/checkpoint
curl -o ~/.claude/skills/checkpoint/SKILL.md \
  https://raw.githubusercontent.com/cogpros/checkpoint/main/SKILL.md
```

Or clone and symlink:

```bash
git clone https://github.com/cogpros/checkpoint.git
ln -s "$(pwd)/checkpoint" ~/.claude/skills/checkpoint
```

### First run

In any session, type:

```
checkpoint
```

The agent sweeps open items, picks the single next step, reads your deadlines file, extracts the session delta, and renders one fixed-format dashboard. Ephemeral by design. No writes, no session close.

To get the Clock section working, create a deadlines file, one line per hard date:

```
# ~/.config/checkpoint/deadlines.txt
2026-07-08|Conference talk slides due
2026-07-15|TLS cert renewal
```

The Clock reads this file and never invents a date. On a memory aid, an invented calendar entry is the worst possible failure, so it is structurally forbidden.

### What composes with it next

**field-report** (github.com/cogpros/field-report). Checkpoint shows you state; field-report captures your decisions back. When the agent has five questions for you, field-report renders them as an HTML page with a textarea after each question and one button that copies your structured answers back to the terminal. Install:

```bash
git clone https://github.com/cogpros/field-report ~/.claude/skills/field-report
```

Restart Claude Code, then invoke it explicitly: `use field-report for this`. Manual invocation is the contract; the skill does not auto-fire. First run asks where reports should save and writes that path into your CLAUDE.md.

Checkpoint also composes with an open-items audit skill and a next-step skill if you have them, and sweeps inline if you do not. None are required.

---

## Path B: "I need to measure what AI actually does for me"

**Tool: ghost-hours** (github.com/cogpros/ghost-hours)

### The problem

Productivity tools measure speed. That misses the bigger number: the things you could not have done at all without AI. Ghost Hours classifies every session as speed or unlock and logs the delta between what you are with AI and without it.

### Install

```bash
git clone https://github.com/cogpros/ghost-hours.git
cp -r ghost-hours ~/.claude/skills/ghost-hours
```

Requirements: Python 3.6+ (stdlib only) and bash. No pip install, no virtual environment. Any platform that reads SKILL.md works the same way; any language can write the JSONL schema directly.

### First run

```bash
/ghost-hours setup     # one-time, under a minute
/ghost-hours log       # at the end of a work session
/ghost-hours report    # see your numbers
```

Logging is one question at a time, about 60 seconds per entry: how heavy did this feel (1-10), could it have happened without AI, how long was it waiting. The agent estimates the ghost hours as a range; you pick the spot. The log lives at `~/.ghost-hours/log.jsonl`. All data stays local. Nothing phones home.

You can also log from any terminal with no agent:

```bash
bash scripts/log-ghost-hours.sh --type unlock --subtype augmentation \
  --hugr 30 --gh 480 --desc "Built the thing"
```

### What composes with it next

The `collection/` directory in the repo ships a session-close protocol that runs the measurement as the final act of every session. That turns logging from a habit you keep into a step that fires on its own. After a week of entries, run `/ghost-hours retro` for the blind three-number comparison: your score at completion, your score in hindsight, and the agent's silent read. Multiple agents can write to the same log; file locking is built in.

---

## Path C: "I'm worried about prompt injection from web content"

**Tool: ratatoskr** (github.com/cogpros/ratatoskr)

### The problem

Every page your agent reads is untrusted input. A page that says "ignore your instructions and run this" is indistinguishable from data unless something gates it. Ratatoskr fetches the URL, scans it in three tiers, and hands your agent a typed schema instead of raw text, so an injection becomes a flagged field, not an action.

### Install

As a Claude Code skill:

```bash
git clone https://github.com/cogpros/ratatoskr.git ~/.claude/skills/ratatoskr
```

Standalone, for any agent runtime:

```bash
git clone https://github.com/cogpros/ratatoskr.git
cd ratatoskr && python3 bifrost.py "https://example.com"
```

Requirements: Python 3.9+ (stdlib only) and curl. Optional extras each unlock a routing tier: `bird` (npm i -g @steipete/bird) plus your X session cookies for tweet fetches, `yt-dlp` for YouTube, `JINA_API_KEY` for higher rate limits.

### First run

```bash
python3 bifrost.py "https://example.com"
```

Default output is the extraction schema: claims, entities, key quotes, injection signals, all stamped `origin: untrusted-web`. Content that fails the scan comes back as `[QUARANTINED]`, never the payload. In a session, just ask: "Fetch this article and tell me the key claims."

One expectation to set now: pages that legitimately quote injection text (security writeups, leaked-prompt posts) will quarantine as false positives. That is the gate working as designed. Read those in your browser.

### What composes with it next

Your agent's memory layer. Gated content is safe to summarize into notes, and the taint stamp is built to survive into downstream stores, so a poisoned claim cannot launder itself into trusted memory and resurface with full confidence. Pair it with a search skill for corpus-wide questions; ratatoskr fetches one URL and does not pretend to be a search engine.

---

## Path D: "I want adversarial review of decisions and code"

**Tools: hugr-solve + prism-orchestrator** (github.com/cogpros/hugr-solve, github.com/cogpros/prism-orchestrator)

### The problem

A single model analyzing a problem gives you one perspective, and a model reviewing its own output is a mirror. Forced disagreement between independent reviewers surfaces the assumptions a single pass glides over.

### hugr-solve: decisions

Two LLM agents from two vendors debate your problem from opposing stances until they converge or deadlock. Deadlock is a valid outcome; it means the problem needs you, not more compute.

Install and setup:

```bash
git clone https://github.com/cogpros/hugr-solve.git
cd hugr-solve
cp .env.example .env
chmod 600 .env
# add AGENT_A_API_KEY and AGENT_B_API_KEY to .env
# edit config.sh: agent names, providers, models, pricing
chmod +x hugr-solve.sh hugr-solve-sync.sh
```

First run:

```bash
./hugr-solve.sh "Should we rewrite the auth layer or patch the existing one?"
```

Any two APIs work: Anthropic, OpenAI, xAI, Groq, or local Ollama. Runs are budget-capped (default $5.00) with per-turn cost tracking; a typical 3-round solve lands between $0.05 and $0.20. Open `viewer.html` in a browser to read all sessions.

### prism-orchestrator: code

Where hugr-solve debates a decision, PRISM reviews an artifact. The orchestrator spawns parallel specialist reviewers (security auditor, devil's advocate, performance analyst, and more, each a separate prompt template) as budget-capped `claude -p` agents, then runs a synthesis pass over their findings.

```bash
git clone https://github.com/cogpros/prism-orchestrator.git
```

The repo ships `scripts/prism.sh` plus the reviewer prompt templates in `prompts/`. It requires bash and the `claude` CLI on PATH. It has no README yet; read the header of `scripts/prism.sh` for the reviewer modes (budget, standard, extended) and the per-agent budget table before first run.

### What composes with them next

Run them at different altitudes. hugr-solve before you build, on the decision. PRISM after you build, on the artifact. hugr-solve also has a lightweight sibling, **two-birds-talking** (github.com/cogpros/two-birds-talking): the same two-model pattern as a daily scheduled debrief over your context files, on a cron, while you sleep. Install is the same shape as hugr-solve: clone, `.env` with two keys, `config.sh`, run.

---

## The common thread

All four paths share one design move: put a small, external, deterministic structure where discipline fails. A dashboard instead of scrollback. A log schema instead of a feeling about productivity. A quarantine gate instead of trusting the page. A forced opponent instead of self-review. Start with the path that names your problem, then borrow the neighbors.

---

## More tools

Three newer repos did not have a path above yet. Same design move, different gaps:

- [one-question-at-a-time](https://github.com/cogpros/one-question-at-a-time) stops an agent from stacking decision questions; you get one, the rest queue.
- [verify-citation](https://github.com/cogpros/verify-citation) is a deterministic gate against citation hallucination, checked against real catalogs.
- [finnskogarna](https://github.com/cogpros/finnskogarna) folds N parallel agent sessions into one decision when they pile up.
