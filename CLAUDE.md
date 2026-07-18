# Project: LLM Model Selection for Java-to-Kotlin Migration

## What this is
An evaluation harness that compares 4 LLMs (Claude Sonnet 4.5, gpt-5-mini,
Gemini 2.5 Flash, Qwen3 Coder) on automated Java-to-Kotlin migration. The
deliverable is the METHODOLOGY of selecting a model with measured evidence,
not the migration itself. Built as a course capstone, documented publicly on
LinkedIn and GitHub.

## Core finding (do NOT change these results)
- Correctness is table stakes, and it separates the field: Claude/gpt-5-mini/Gemini pass 100%, but Qwen drops to 67% (failed all 3 hard order_filter trials, passed every easy/medium one).
- The benchmark (artificialanalysis SciCode) predicted Claude > gpt-5-mini > Gemini > Qwen.
- Reality fractured: idiom quality, speed, and cost do NOT track the benchmark.
- gpt-5-mini (benchmark rank 2) is worst value: 734 avg output tokens from hidden reasoning overhead, and by far the slowest (8.31s).
- The decision matrix flips winner by business weights: quality-first -> Claude, cost/speed-first -> Gemini (100% reliable, fastest at 1.33s, cheapest of the reliable models). Qwen is cheaper still but too inconsistent to recommend.
- Thesis: model selection is a business decision, not a leaderboard lookup.

## How the harness works
1. generate_kotlin_with_usage: Java in, Kotlin out, captures token usage
2. compile_kotlin: subprocess to kotlinc, returns (success, error_text)
3. run_kotlin_test: glues a test onto the Kotlin, compiles, runs, checks behavior
4. judge_idiom: LLM-as-judge (Claude) scores 1-5 on Kotlin idiom
5. run_full_eval: 4 models x 3 snippets x 3 trials = 36 rows into a DataFrame
6. Weighted decision matrix + Gradio slider UI that flips the winner by business priorities

## Data files (DO NOT regenerate, they cost API calls)
- eval_results_full.csv: all 36 trials
- eval_summary.csv: per-model summary
These are the source of truth. Reload from them, never re-run the eval.

## What I want you to do
Clean this notebook into a professional GitHub repo:
- Reorganize cells into clear logical sections with markdown headers
- Add docstrings and comments where missing
- Write a strong README.md (problem, methodology, findings table, how to run)
- Add a requirements.txt
- Add a .gitignore that excludes .env
- Suggest a clean repo structure

## Hard constraints
- NEVER commit or expose .env or API keys
- NEVER change the eval results, numbers, or findings above
- NEVER re-run the eval loop (it costs money and the CSVs are the truth)
- Keep all code runnable; do not break working functions
- This is MY course work: do not add or redistribute course materials, only my own build
