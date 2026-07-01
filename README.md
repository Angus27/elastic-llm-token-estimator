# Elastic Security — LLM Token Estimator

A single-file, offline-first tool for estimating monthly LLM token consumption and cost for Elastic Security's two AI features: **AI Assistant** and **Attack Discovery**.

No server. No dependencies. Open the HTML file in any browser.

---

## What it does

Adjust sliders (or type values directly) to model your environment. The tool instantly calculates:

- **Total tokens/month** — input and output split
- **Estimated monthly cost** — broken down by feature, based on your configured per-token pricing
- **Token composition** — a stacked bar showing exactly where tokens come from (system prompt, RAG, alert context, conversation history, user messages, model responses, AD alert data)
- **Per-call breakdowns** — itemised token counts per AI Assistant turn and per Attack Discovery run
- **Key sensitivities** — auto-generated analysis of the biggest cost drivers and the highest-impact levers for reduction

---

## Inputs

### Organisation scale
| Input | Description |
|---|---|
| Security analysts | Total number of analysts with access to AI Assistant |
| Active analysts per day | Percentage who use the tool on any given day |

> All estimates use **30.44 calendar days/month** (365 ÷ 12). Attack Discovery runs continuously — there is no working-days cap.

### AI Assistant — session behaviour
| Input | Description |
|---|---|
| Sessions per analyst per day | Distinct AI Assistant conversations |
| Turns per session | User/model exchanges per conversation |
| Avg user message | Tokens in a typical analyst query |
| Avg model response | Tokens in a typical model reply |

### AI Assistant — context per turn
These tokens are injected by Elastic's backend on every API call. Analysts don't see them, but they typically dominate usage.

| Input | Description |
|---|---|
| System prompt | Fixed instructions sent on every call |
| RAG / KB chunks | Knowledge base snippets retrieved per turn |
| Alert / event context | Alert fields and values injected per turn |
| History depth | Number of prior exchanges re-sent in each call |

### Attack Discovery
| Input | Description |
|---|---|
| Runs per day | How frequently AD analyses the alert queue |
| Alerts per run | Batch size fed to the model |
| Tokens per alert | Average field/value payload per alert |
| System / task prompt | Fixed instructions for the AD model call |
| Output per run | Tokens in the generated attack summaries |

### Model pricing
Defaults to **Claude Sonnet 4.6** ($3.00/1M input · $15.00/1M output). Change these fields to match your deployed model.

---

## Features

- **Editable number inputs** — type a value directly alongside any slider; they stay in sync and clamp to the slider's valid range
- **Persistent state** — all inputs are saved to `localStorage` and restored on next open
- **Reset to defaults** — single button restores all inputs to baseline values
- **JSON export** — downloads a structured snapshot of all inputs and computed outputs
- **CSV export** — downloads a spreadsheet-ready file with the same data, separated into inputs and outputs

---

## Methodology notes

### History token calculation

The history cost per turn is calculated using the true average history depth across a session, not just the slider value. Early turns in a session have fewer prior turns available than the configured maximum, so the effective average depth is:

```
if turns ≤ hist + 1:  avg_depth = (turns - 1) / 2
otherwise:            avg_depth = [hist*(hist+1)/2 + (turns-1-hist)*hist] / turns
```

Each prior turn in history contributes `user_message + model_response` tokens (the full exchange).

### Days per month

`365 / 12 = 30.44` — used for all monthly projections, including Attack Discovery which runs on calendar days, not working days.

---

## Files

```
elastic-llm-estimator.html   — the entire tool (HTML + CSS + JS, ~600 lines)
elastic-icon.png             — Elastic logo used in the header
README.md                    — this file
```

---

## Usage

```bash
# Open directly in your browser — no build step, no server needed
open elastic-llm-estimator.html
```

Or drag the file into any browser window.

---

## Limitations

- Estimates only — actual token usage depends on model version, Elastic configuration, and real analyst behaviour
- Does not account for context window overflow — verify that your configured per-turn input totals are within your model's context limit
- Does not model multi-step Attack Discovery pipelines (assumes a single LLM call per run)
- Cost estimates assume a flat per-token rate with no volume discounts or commitment pricing

---

## Contributing

This is a single-file tool by design — keep it self-contained and offline-capable. Improvements should not introduce external runtime dependencies.
