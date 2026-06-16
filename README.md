# claude-cost

A script that reads your local Claude Code conversations and shows what the same usage would cost at API rack rates — including cache tokens, which Claude Code's built-in display doesn't count.

**For users of the [Claude Code CLI](https://claude.ai/code).** Requires no API key and sends no data anywhere — it reads the JSONL files Claude Code writes to `~/.claude/projects/` on your machine.

---

Claude Code's built-in cost display only counts input + output tokens. Cache reads and writes are the dominant cost for long-context workflows — often 3–5× the raw token cost. This tool counts all of it and compares the total against your subscription price.

```
Claude Code — Usage Report  (June 2026: 2026-06-01 → 2026-06-16)
──────────────────────────────────────────────────────────────────
  API value               $  579.46   ← what this usage costs at rack rates
  Subscription            $  100.00/month
  ROI                          5.8x

  Token breakdown
  Output          11.6M  $15.00/MTok  = $ 174.56
  Cache reads    727.1M  $ 0.30/MTok  = $ 218.12
  Cache writes    31.1M  $ 6.00/MTok  = $ 186.50
  Input             93K  $ 3.00/MTok  = $   0.28

  By project
  my-app                  36s  $ 216.81  ████████████████████
  backend-api             23s  $ 192.97  ██████████████████░░
  infra-tooling           19s  $ 117.13  ███████████░░░░░░░░░
  misc                     8s  $  42.26  ████░░░░░░░░░░░░░░░░
```

## Install

Requires Python 3 (no dependencies beyond stdlib).

```bash
curl -o ~/.local/bin/claude-cost https://raw.githubusercontent.com/suryamp/claude-cost/main/claude-cost
chmod +x ~/.local/bin/claude-cost
```

Or clone and symlink:

```bash
git clone https://github.com/suryamp/claude-cost
ln -s "$PWD/claude-cost/claude-cost" ~/.local/bin/claude-cost
```

Make sure `~/.local/bin` is on your `$PATH`. On most systems it already is; if not, add this to your shell rc:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

```
claude-cost           # all tracked conversations
claude-cost --month   # current calendar month
claude-cost --week    # last 7 days
```

## Configuration

Create `~/.config/claude-cost/config.json` to set your subscription cost (default: $100):

```json
{
  "subscription": 100.00
}
```

## Pricing

Rates are loaded per model from the table below. Each assistant message in the JSONL files records which model handled it, so mixed-model sessions are priced correctly. Unknown models fall back to Sonnet 4.6 rates.

*Last verified: 2026-06-16. Source: [anthropic.com/pricing](https://www.anthropic.com/pricing)*

| Model | Input | Output | Cache write (5 min) | Cache write (1 hr) | Cache read |
|---|---|---|---|---|---|
| Claude Fable 5 | $10.00 | $50.00 | $12.50 | $20.00 | $1.00 |
| Claude Opus 4.6/4.7/4.8 | $5.00 | $25.00 | $6.25 | $10.00 | $0.50 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $3.75 | $6.00 | $0.30 |
| Claude Haiku 4.5 | $1.00 | $5.00 | $1.25 | $2.00 | $0.10 |

All rates are per million tokens. To update pricing, edit the `PRICING` dict at the top of the script.
