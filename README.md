# claude-cost

A CLI tool that reads your local Claude Code conversation files and shows you how much API value you're getting from your subscription.

```
Claude Code — Usage Report  (June 2026: 2026-06-01 → 2026-06-16)
──────────────────────────────────────────────────────────────────
  API value               $  579.46
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

Claude Code's built-in cost display only counts input + output tokens. This tool includes **cache read and write tokens**, which are the dominant cost for long-context workflows — often 3–5x the raw token cost. That's the number that matters when comparing against API pay-as-you-go pricing.

## Install

Requires Python 3 (no dependencies beyond stdlib). Conversations are read from `~/.claude/projects/`, which Claude Code writes automatically.

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

Create `~/.config/claude-cost/config.json` to override the subscription cost (default: $100):

```json
{
  "subscription": 100.00
}
```

## Pricing

Rates are loaded per model from a built-in table. Each assistant message in the JSONL files includes the model that handled it, so mixed-model sessions are priced correctly.

| Model | Input | Output | Cache write (5 min) | Cache write (1 hr) | Cache read |
|---|---|---|---|---|---|
| Claude Fable 5 | $10.00 | $50.00 | $12.50 | $20.00 | $1.00 |
| Claude Opus 4.6/4.7/4.8 | $5.00 | $25.00 | $6.25 | $10.00 | $0.50 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $3.75 | $6.00 | $0.30 |
| Claude Haiku 4.5 | $1.00 | $5.00 | $1.25 | $2.00 | $0.10 |

All rates are per million tokens. Unknown models fall back to Sonnet 4.6 rates. To update pricing, edit the `PRICING` dict at the top of the script.

## How it works

Claude Code stores every conversation as a JSONL file under `~/.claude/projects/<project>/`. Each assistant turn includes a `usage` object with token counts broken down by type and a `model` field identifying which Claude model handled that turn. This tool scans all of those files, computes cost per message using model-specific rates, and reports what the same usage would cost without a subscription.

No data leaves your machine.
