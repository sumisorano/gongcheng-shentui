# 功成身退

> *Gongcheng Shentui — Retire with merit. The Way of Heaven.* — Tao Te Ching

**Skill lifecycle management for Claude Code. Keep your context lean by retiring completed skills.**

> [🇬🇧 English](README_EN.md) | [🇨🇳 中文](README.md)

---

## The Problem

Once loaded, a Skill's SKILL.md stays in the conversation forever — even after it's done. Every subsequent turn pays for that dead weight.

In long sessions with multiple skills: load `code-review` in turn 5, `deploy-check` in turn 8, `api-docs` in turn 12... three skills add up to ~10–20k tokens of rules you no longer need — sometimes more than the conversation itself.

**`ctx-lifecycle` gives completed skills an honorable exit.**

---

## Design Philosophy

Anthropic designed Skills with three layers of **Progressive Disclosure**:

| Layer | Content | Loaded |
|-------|---------|--------|
| 1 | frontmatter (name + description) | Always scanned |
| 2 | SKILL.md body | On trigger |
| 3 | references / scripts | On reference |

Each layer is heavier and lazier than the last. That's how skills **arrive**.

**`ctx-lifecycle` adds layer 4 — how they leave:**

| Layer | Content | Reclaimed |
|-------|---------|-----------|
| 4 | Completed SKILL.md body | Full reclamation, leaving one-line summary |

Arrival is progressive. Departure is progressive. **Full skill lifecycle = Progressive Disclosure × ctx-lifecycle.**

---

## Quick Start

### Install

Copy the two command files into your Claude Code project:

```bash
cp -r .claude/commands/ your-project/.claude/commands/
```

### Usage

```bash
# Step 1: Diagnose
/ctx-check
```

- 🟢 "Context is clean" → Keep going
- 🔴 Lists completed skills → Proceed to step 2:

```bash
# Step 2: Reclaim
/ctx-clean
```

### Declare lifecycle in your Skills

Use the template in `lifecycle-template/SKILL.md` to declare lifecycle metadata:

```yaml
lifecycle:
  type: one-shot                # one-shot / session-long
  completion_signal: "..."      # What signals "done"
  retention_summary: "..."      # One-line summary after cleanup
```

See [`lifecycle-template/GUIDE.md`](lifecycle-template/GUIDE.md) for details.

---

## How It Works

| Command | Type | What it does |
|---------|------|-------------|
| `/ctx-check` | Diagnostic | Analyzes conversation history, identifies loaded skills and their status |
| `/ctx-clean` | Reclamation | Wraps `/compact` with a retention manifest — cleans completed skills first |

Both commands are pure prompt engineering — no external tools, no Claude Code modifications.

### Lightweight Gate

`/ctx-check` skips full diagnostics when the conversation is < 10 turns with < 2 skills loaded. It just says "Context is clean, no need to retire skills yet."

### Safety

- 🛡️ Active skills are never touched
- 🛡️ Uncertain skills are kept (better safe than sorry)
- 🛡️ Reclaimed skills leave a one-line summary — they can be re-loaded anytime

---

## ROI

> **Benchmark:** 4 completed skills reclaimed (~800 tokens each in rules + `/compact` conversation compression), releasing ~18k–22k tokens per cleanup. Assuming 3 sessions/day, 22 working days/month.

One-click reclamation saves **~20k tokens per cleanup**. What does that mean in real money?

### Monthly savings (single user)

Assuming 3 sessions/day, 22 working days/month:

| Model | Input price / 1M | Tokens saved / month | Money saved / month | Yearly |
|-------|-----------------|---------------------|--------------------|--------|
| Claude Opus 4 | $15.00 | 1.32M | **$19.80** | $238 |
| Claude Sonnet 4 | $3.00 | 1.32M | **$3.96** | $48 |
| DeepSeek V4 Flash | ¥0.5 (off-peak) → ¥1.0 (peak) | 1.32M | **¥0.66~1.32** | ¥8~16 |

### Team savings

| Model | 10-person team / month | 10-person team / year |
|-------|----------------------|---------------------|
| Claude Opus | **$198** | **$2,376** 🚀 |
| Claude Sonnet | **$39.60** | **$475** |
| DeepSeek | ¥6.6~13.2 | ¥80~160 |

### Beyond money

| Benefit | Detail | Why it matters |
|---------|--------|---------------|
| ⏱️ **Speed** | From >60s timeout down to <15s — **4× faster** | Waiting breaks flow. At 60s you check your phone; at 15s you stay in the zone |
| 🧠 **Effective context** | Dead skills don't crowd the window; the model focuses on what you actually need | Same conversation turn, but the model reads your request instead of 5 skill rulebooks |
| 🛡️ **No wall-hitting** | Stay fast at 50+ turns without losing context to a new session | Without ctx-lifecycle, you hit timeout long before you're done. Starting fresh means losing all intermediate context |
| 🪙 **Quota efficiency** | Pro/Max users get more done within their monthly limit | Your quota pays for reasoning, not re-transmitting dead skill rules every turn |
| ♻️ **Compound savings** | Each cleanup saves ~20k once, but **keeps saving on every subsequent turn** | Reclaim 4 skills × ~800 tokens each × 20 turns = 64k extra tokens saved beyond the initial compaction |
| 🎯 **Output quality** | Less noise in the context window = fewer hallucinated responses | Sometimes the model thinks a dead skill's task is still active, leading to off-topic or confused answers |

> 💡 DeepSeek is cheap — the real win is time, not token cost. For Claude Opus users, each 20k tokens saved ≈ $0.30.

---

## Strategy

Tools alone aren't enough — you need to know **when** to use them.

See [`docs/STRATEGY.md`](docs/STRATEGY.md) (Chinese) for the full strategy guide. Key rules of thumb:

| Question | Answer |
|----------|--------|
| When to run `/ctx-check`? | When **≥ 3 skills** are loaded |
| When to run `/ctx-clean`? | When `/ctx-check` recommends it |
| Still slow after 30+ turns? | New session + summary |

---

## Project Structure

```
gongcheng-shentui/
├── README.md                       # 中文版 (Chinese)
├── README_EN.md                    # 英文版 (English, you are here)
├── CHANGELOG.md                    # Version history
├── DESIGN.md                       # Design rationale (Chinese)
├── CONTRIBUTING.md                 # Contribution guide (Chinese)
├── CODE_OF_CONDUCT.md              # Code of conduct
├── SECURITY.md                     # Security policy
├── LICENSE                         # GPL-3.0
├── docs/
│   └── STRATEGY.md                 # Strategy guide (Chinese)
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── PULL_REQUEST_TEMPLATE.md
├── .claude/
│   └── commands/
│       ├── ctx-check.md            # /ctx-check command
│       └── ctx-clean.md            # /ctx-clean command
├── test-reports/
│   └── 功成身退-压力测试报告.md     # Test report (Chinese)
└── lifecycle-template/
    ├── SKILL.md                    # Lifecycle template
    ├── GUIDE.md                    # Template guide
    └── examples/
        └── code-review.md
```

---

## Comparison

| Dimension | Dynamic Context Pruning | ctx-lifecycle |
|-----------|------------------------|---------------|
| Focus | Generic context pruning | **Skill lifecycle management** |
| Strategy | Dedup, overwrite, error cleanup | **Task-completion-based reclamation** |
| Philosophy | Context optimization | **Load when needed, leave when done** |

---

## Roadmap

- [x] Product positioning & naming
- [x] `/ctx-check` + `/ctx-clean` commands
- [x] Skill lifecycle template + guide
- [x] Verified — 5-skill stress test passed ✅ (~20k tokens reclaimed)
- [ ] Demo GIF

---

## License

GPL-3.0

---

> Retire with merit. Skill should be no different.
