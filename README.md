# LLM Model Selection for Java→Kotlin Migration

**Thesis: choosing a model is a business decision, not a leaderboard lookup.**

This project is an evaluation harness that compares four LLMs on automated
Java→Kotlin migration and turns the measurements into a defensible model choice.
The deliverable is the *methodology* — how to select a model with measured
evidence — not the migration itself.

Built as a course capstone.

---

## The problem

You need to migrate a Java codebase to Kotlin with an LLM. Four candidates are on
the table. The obvious move is to pick whichever ranks highest on a public coding
benchmark — but does benchmark rank actually predict which model is best *for this
job*? This harness answers that empirically instead of by reputation.

**Models evaluated**

| Model | Access | SciCode (AA) | Predicted rank |
|---|---|--:|--:|
| Claude Sonnet 4.5 | Anthropic API | 45 | 1 |
| gpt-5-mini | OpenAI API | 39 | 2 |
| Gemini 2.5 Flash | OpenRouter | 39 | 3 |
| Qwen3 Coder | OpenRouter | 36 | 4 |

Predictions were **locked before running any eval** so the benchmark ranking
couldn't be retrofitted to the outcome.

---

## Methodology

**36 measured runs** = 4 models × 3 snippets × 3 trials. Each snippet is a
(Java source, Kotlin behavioral test) pair chosen to probe something Kotlin does
differently from Java, so a model that merely transliterates syntax gets caught:

| Snippet | Difficulty | Probes |
|---|---|---|
| `calculator` | easy | basic method signatures |
| `user_lookup` | medium | null safety / nullable handling |
| `order_filter` | hard | idiomatic collection ops vs. manual loops |

Each trial runs a five-stage pipeline:

1. **Generate** — send Java, get Kotlin back, capturing real token usage.
2. **Compile** — shell out to `kotlinc`; does it even build?
3. **Behavioral test** — glue a test onto the Kotlin, run it, check it *works*.
4. **Idiom judge** — Claude scores the output 1–5 for Kotlin idiom (LLM-as-judge),
   independent of correctness.
5. **Aggregate** — collect correctness, idiom, speed, and real cost per model.

The results feed a **weighted decision matrix**: metrics are normalized 0–1 and
weighted by a business profile, so the recommended model changes with what a team
values. A Gradio UI exposes the weights as live sliders.

---

## Findings

| Model | Pass rate | Idiom (1–5) | Avg sec | Avg cost | Avg output tokens | Benchmark | Predicted rank |
|---|--:|--:|--:|--:|--:|--:|--:|
| **claude-sonnet-4-5** | 1.00 | **4.67** | 2.03 | $0.0021 | 85 | 45 | 1 |
| gpt-5-mini | 1.00 | 4.56 | 8.31 | $0.0015 | **734** | 39 | 2 |
| gemini-2.5-flash | 1.00 | 4.22 | **1.33** | $0.0003 | 76 | 39 | 3 |
| qwen3-coder | 0.67 | 4.11 | 1.50 | **$0.0001** | 87 | 36 | 4 |

**What the data says:**

- **Correctness is table stakes — and Qwen misses it.** Three models pass 100%,
  but **Qwen drops to 67%, failing all three `order_filter` (hard) trials** while
  passing every easy/medium one. Reliability breaks down exactly where the task
  gets harder.
- **The benchmark did not predict real-world value.** Idiom quality, speed, and
  cost do *not* track the SciCode ranking.
- **gpt-5-mini (benchmark rank 2) is the worst value:** ~734 avg output tokens
  from hidden reasoning overhead — by far the slowest (8.31s) and comparatively
  expensive for this task.
- **The winner flips with business priorities:** quality-first → **Claude**
  (top idiom, 100% correct); cost/speed-first → **Gemini** (100% reliable, fastest
  at 1.33s, and cheapest of the fully-reliable models). **Qwen is cheaper still
  ($0.0001) but too inconsistent to recommend** — it failed every hard conversion.

**Conclusion: model selection is a business decision, not a leaderboard lookup.**

---

## How to run

**Prerequisites**

- Python 3.10+
- The Kotlin compiler `kotlinc` on your PATH (`brew install kotlin`)
- API keys for OpenAI, Anthropic, and OpenRouter

**Setup**

```bash
pip install -r requirements.txt
cp .env.example .env      # then fill in your real keys
```

**Run**

Open `eval.ipynb` and run the sections top to bottom.

> ⚠️ **The eval loop (Section 7) makes paid API calls.** The results of record are
> already saved in `eval_results_full.csv` and `eval_summary.csv`. Section 12
> reloads from those CSVs, so you can explore the decision matrix and Gradio UI
> **without re-running the eval**. Reload from the CSVs — don't regenerate them.

---

## Repository structure

```
.
├── eval.ipynb                # the analysis, in 12 documented sections
├── eval_results_full.csv     # all 36 trials — source of truth
├── eval_summary.csv          # per-model summary
├── requirements.txt          # pinned Python dependencies
├── .env.example              # template for API keys (real .env is gitignored)
├── .gitignore
├── CLAUDE.md                 # project guardrails
└── README.md
```

---

## Caveats

- Small sample (n = 36); results indicate direction, not statistical certainty.
- Idiom scoring is single-judge (Claude); a different judge may score differently.
- Benchmark score is AA's SciCode (the one metric with coverage across all four
  models at evaluation time), not a Kotlin-specific benchmark.
- Course capstone — own work only.
