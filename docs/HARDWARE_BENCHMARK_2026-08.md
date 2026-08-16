# Hardware Benchmark: Non-GPU Legacy Laptop (2026-08)

## Overview

Every number in this document was actually measured by running MalxLabs-V4 —
**and, for direct comparison, the untuned base model it's fine-tuned from** —
with `llama.cpp` on the hardware listed below. Nothing here is estimated or
copied from the main README's GTX 1080 numbers. It follows the
[CONTRIBUTING.md](../CONTRIBUTING.md) hardware-testing template and directly
addresses the roadmap item *"Benchmark suite for hardware testing"* from
[CHANGELOG.md](../CHANGELOG.md).

This test targets the opposite end of "legacy" from the main README: instead of
an aging **discrete GPU** (GTX 1080, Pascal, 2016), this rig has **no discrete
GPU at all** — a 2019 ultrabook CPU with integrated graphics only. It answers
two questions the README doesn't: *does MalxLabs-V4 still work — not fast, just
work — on hardware with zero CUDA/ROCm path?* And: ***is the SFT fine-tune
actually better than the stock DeepSeek-R1-Distill-Qwen-1.5B it started from,
or just different?*** Both models were downloaded, run, and measured identically
on this rig so the comparison is apples-to-apples, not marketing.

## Hardware Configuration

| Component | Specification |
|---|---|
| **CPU** | Intel Core i5-1035G7 (Ice Lake), 4 cores / 8 threads, 1.2 GHz base |
| **GPU** | Intel Iris Plus Graphics (integrated) — **no discrete GPU, no CUDA/ROCm** |
| **RAM** | 16 GB |
| **OS** | Windows 11 Pro |
| **Inference engine** | `llama.cpp` build **b10448**, official `win-cpu-x64` release binary (icelake-optimized `ggml-cpu-icelake.dll` backend, auto-selected) |
| **Model A (ours)** | [MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/MalxTech/MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B), Q4_K_M GGUF, 1.04 GiB |
| **Model B (base, untuned)** | [DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B), quantized by [bartowski](https://huggingface.co/bartowski/DeepSeek-R1-Distill-Qwen-1.5B-GGUF), Q4_K_M GGUF, 1.04 GiB — **the exact model MalxLabs-V4 was fine-tuned from**, unmodified |

No model files, engine binaries, or caches were written to the system (`C:`)
drive — everything ran from a dedicated data volume, CPU inference only.

## Methodology

1. **Throughput** — `llama-bench` (the project's standardized benchmark tool),
   `-p 512 -n 128 -t 8`, default settings, 3 repetitions (tool-reported ± is
   sample stddev across those repetitions).
2. **Qualitative reproduction** — re-ran the README's own "average of 10
   numbers" prompt verbatim, same sampling settings the README specifies
   (`--temp 0.6 --min-p 0.1 -c 4096`).
3. **Own accuracy spot-check** — 8 hand-written grade-school arithmetic/rate
   word problems (GSM8K-style, not the GSM8K test set itself), single-shot,
   same sampling settings, `-n 500`, run identically against MalxLabs-V4 and
   the untuned base model. Final answer extracted programmatically from
   `ANSWER: <n>` or `\boxed{<n>}` and compared to the known answer. The two
   base-model problems that didn't finish in budget were re-run at `-n 900`
   to check whether they were wrong or just truncated.
4. **Memory** — sampled `Get-Process` working set on the live `llama-cli`
   process mid-generation at `-c 4096`.

## Results

### Throughput — MalxLabs-V4 vs. base, both measured (`llama-bench`)

| Test | MalxLabs-V4 | Base (untuned) | README's GTX 1080 claim |
|---|---|---|---|
| Prompt processing (pp512) | **61.48 ± 4.62 tok/s** | 58.51 ± 8.44 tok/s | not directly comparable (README doesn't split pp/tg) |
| Text generation (tg128) | **14.50 ± 2.24 tok/s** | 12.76 ± 3.15 tok/s | 80+ tok/s w/ `-ngl 35` full GPU offload |

Throughput is essentially a wash — both models are the same architecture and
size, so this is mostly noise (the ± ranges overlap). CPU-only generation lands
at roughly **1/5th** the GTX 1080's claimed speed for either model — expected,
given there's no GPU at all here. **Speed was never going to be where the
fine-tune shows up; accuracy is.**

### Memory footprint (measured)

- **~1.7 GB working set** for the `llama-cli` process at `-c 4096`, Q4_K_M
  weights, CPU-only — comfortably inside this machine's 16 GB. Same order for
  the base model (not separately re-measured; identical weight count/format).

### Benchmark 1: Grade-school accuracy spot-check — MalxLabs-V4 vs. base (measured, n=8)

The real question: **is the SFT fine-tune actually better than the stock model
it started from?** Same 8 hand-written grade-school word problems, same
sampling settings, same `-n 500` token budget, same machine, back to back.

| # | Problem type | Expected | MalxLabs-V4 | Base (untuned) |
|---|---|---|---|---|
| 1 | Two-step addition/division word problem | 72 | ✅ 72 | ✅ 72 |
| 2 | Fraction-of-a-quantity word problem | 3 | ✅ 3 | ✅ 3 |
| 3 | Rate × time word problem | 10 | ✅ 10 | ✅ 10 |
| 4 | Multi-step multiplication word problem | 624 | ✅ 624 | ⚠️ no answer in budget |
| 5 | Sequential arithmetic word problem | 29 | ✅ 29 | ✅ 29 |
| 6 | Rate × time word problem | 180 | ✅ 180 | ✅ 180 |
| 7 | Multiplicative comparison word problem | 42 | ✅ 42 | ✅ 42 |
| 8 | Proportion/scaling word problem | 5 | ✅ 5 | ⚠️ no answer in budget |
| | **Total (matched 500-token budget)** | | **8 / 8 (100%)** | **6 / 8 (75%)** |

**The honest caveat, because it matters:** problems 4 and 8 for the base model
were **not wrong answers** — the base model rambled longer in its `<think>`
block and ran out of the 500-token budget before printing a final line.
Problem 8's transcript shows it had already reached the correct answer (5 cups,
via two independent methods) and was still double-checking when the budget cut
it off. Re-run with a larger budget (`-n 900`), **the base model gets both
right too — 8/8 (100%), identical to MalxLabs-V4.**

**What this actually shows:** on this small sample, the SFT fine-tune didn't
make the model *smarter* at these problems — the base model can solve all 8
just as correctly, given room. What it changed is **token efficiency**:
MalxLabs-V4 consistently reaches a final answer inside a tight budget, while
the untuned base model sometimes needs ~2x the tokens to stop double-checking
itself and commit. On hardware this constrained, where every extra generated
token costs real wall-clock time, that's a genuine, measurable win for the
fine-tune — just not the win a bare "8/8 vs 6/8" headline implies without this
context. The chart below shows the matched-budget numbers with this caveat
annotated directly on it.

One notable format difference: MalxLabs-V4 consistently answered in
`\boxed{<number>}` LaTeX notation despite being prompted for `ANSWER: <number>`
— almost certainly a habit from its `OpenMathInstruct-2` fine-tuning data. The
base model, in this sample, mostly followed the `ANSWER:` format as asked.
Worth knowing if you're scripting against either model's output.

**This is not a formal GSM8K evaluation** for either model — 8 hand-picked
problems, single-shot, one run each — and shouldn't be read as directly
comparable to the published 5-shot GSM8K scores in the chart below. It's a
real, reproducible, matched-conditions spot-check, nothing more.

### Reproducing the README's own example

Re-ran the README's "average of 10 numbers" prompt verbatim. At `-n 400` the
model was still mid `<think>` block when the token budget ran out — consistent
with the "Logic Horizon" finding already documented in
[RESEARCH_FINDINGS.md](RESEARCH_FINDINGS.md#1-the-logic-horizon-phenomenon).
Its draft code at that point also had a real bug worth flagging: it read
`num = input(...)` (a string) and did `sum_numbers += num` — a `TypeError` in
Python 3, since it hadn't yet cast to `int()`/`float()`. The completed version
in the README's own "Output" section does this correctly, so this looks like
a case the model would have self-corrected had generation continued — but it's
a real observed failure mode at the edge of the token budget, not a
hypothetical one.

## Going Harder: Two Additional Head-to-Head Benchmarks

The n=8 grade-school spot-check above turned out to mostly measure *token
efficiency* — the base model could solve nearly everything given enough
budget. To actually test whether the fine-tune improved reasoning itself, not
just conciseness, two harder, more demanding benchmarks were run, same rig,
same methodology, both models back to back.

### Benchmark 2: Harder multi-step math & logic (n=8, matched budget)

Eight problems requiring setting up and solving equations, not just single
arithmetic operations: work-rate ("two pipes filling a tank"), compound
interest, age-in-N-years algebra, ratio decomposition, multi-leg
distance/time, and classic sum/difference and father/son age puzzles. Same
`-n 700` budget for both models, same sampling settings, same rig.

| | MalxLabs-V4 | Base (untuned) |
|---|---|---|
| **Correct @ 700 tokens** | **8 / 8 (100%)** | 4 / 8 (50%) |
| **Correct @ 1300 tokens** (base only, re-run) | — | 7 / 8 (87.5%) |

**The honest detail, because it's the interesting part:** re-running the
base model's 4 misses at nearly double the budget (`-n 1300`), 3 resolved
correctly — the model had reasoned its way to the right answer well before the
cutoff and simply needed more room to finish. But **the 4th (the flour/sugar/
butter ratio problem) never resolved, even at 1300 tokens.** Its transcript
shows the model deriving the correct answer (450g) three separate times,
explicitly, and then each time doubting itself and restarting the
derivation — a genuine repetitive self-verification loop, not a budget
shortfall. This is the same "hallucination loop" / "Logic Horizon" failure
mode already documented in
[RESEARCH_FINDINGS.md](RESEARCH_FINDINGS.md#1-the-logic-horizon-phenomenon),
observed directly in the base model, on a problem MalxLabs-V4 solved cleanly
on the first pass. That's a real, reproducible reasoning-decisiveness
difference attributable to the fine-tune — not just "give it more tokens."

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart4_hardmath_dark.svg">
  <img src="benchmarks/chart4_hardmath_light.svg" alt="Harder math and logic reasoning accuracy: MalxLabs-V4 100% versus base model 50% at matched token budget, base model reaching 87.5% with more budget but one problem never resolving due to a self-verification loop">
</picture>

### Benchmark 3: Execution-verified code generation (n=6)

Six Python functions of real algorithmic substance — primality testing,
efficient (non-naive-recursive) Fibonacci, palindrome checking with
punctuation/case handling, merging sorted lists, binary search, vowel
counting. Unlike the earlier "reproduce the README's example" check, **the
generated code was actually extracted and executed** against test assertions
in a sandboxed subprocess with a timeout — pass/fail is whatever Python's
`assert` says, not a human judgment call.

| | MalxLabs-V4 | Base (untuned) |
|---|---|---|
| **Tests passed** | **5 / 6** | 3 / 6 |

Per-function, with the *actual* cause of each failure (verified by reading
the transcripts, not assumed):

| Function | MalxLabs-V4 | Base (untuned) |
|---|---|---|
| `is_prime` | ✅ | ✅ |
| `fibonacci` | ✅ | ❌ never produced any code — spent the full 700-token budget deliberating between fast-doubling, matrix exponentiation, and memoization without settling on one |
| `is_palindrome` | ❌ real bug: used a literal `.replace('[^a-z]', '')` (a regex pattern as a plain string) instead of `re.sub()`, so punctuation was never actually stripped | ❌ never produced any code — same deliberation pattern as its `fibonacci` attempt |
| `merge_sorted` | ✅ | ✅ |
| `binary_search` | ✅ | ✅ |
| `count_vowels` | ✅ | ❌ real bug: wrote `s.lower().count('aeiou')`, which counts occurrences of the 5-letter substring `"aeiou"`, not individual vowel characters — a fundamentally wrong approach, not a typo |

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart5_codegen_dark.svg">
  <img src="benchmarks/chart5_codegen_light.svg" alt="Execution-verified code generation: MalxLabs-V4 passes 5 of 6 real Python function tests versus base model's 3 of 6, with a per-function pass/fail grid">
</picture>

**Why this benchmark is more convincing than the first spot-check:** these
failures aren't budget artifacts — 2 of the base model's 3 misses never
produced *any* code at all (unlike the math benchmark, more tokens wouldn't
obviously fix "which algorithm should I even use" indecision within a single
generation), and the 3rd is a real, verifiable logic bug. MalxLabs-V4's one
miss is also a real bug, not a truncation — full transparency either way.

## Comparison to Similar Small Reasoning Models

Charts below compare MalxLabs-V4 against its own base model and against
publicly documented models in the same weight class. Parameter counts and the
left-hand GSM8K scores are **sourced from each model's official card / paper**
(cited in the footer); the MalxLabs-V4 vs. base numbers are **first-party
measurements from this document**.

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart1_params_dark.svg">
  <img src="benchmarks/chart1_params_light.svg" alt="Parameter count comparison: MalxLabs-V4 and six similarly-sized small models">
</picture>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart2_gsm8k_dark.svg">
  <img src="benchmarks/chart2_gsm8k_light.svg" alt="Grade-school math accuracy: published GSM8K scores for four small instruct models, plus our own measured head-to-head spot-check of MalxLabs-V4 (100%) versus its untuned base model (75% at matched budget, 100% with more budget)">
</picture>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart3_hardware_dark.svg">
  <img src="benchmarks/chart3_hardware_light.svg" alt="MalxLabs-V4 vs base model real hardware benchmark dashboard on the Intel i5-1035G7 laptop">
</picture>

## Limitations & Honesty Notes

- The 4 "published (5-shot)" bars in the GSM8K chart are **not** independently
  re-run by us — they're cited figures from each model's card/paper. Every
  other number in this document (all 3 benchmarks, both models) is a
  first-party measurement from this rig.
- All three spot-checks (n=8, n=8, n=6) are sanity checks, not comprehensive
  benchmark suites, and none use the actual GSM8K/HumanEval test sets — their
  methodology differs from the published 5-shot GSM8K numbers in the chart, so
  don't compare across those groups directly. A contributor running the real
  GSM8K or HumanEval test sets (see `CONTRIBUTING.md`) against both models
  would be a genuinely useful, more rigorous follow-up.
- Benchmark 1's 6/8-at-matched-budget result for the base model was mostly a
  **budget** effect (documented above). Benchmark 2 re-tested that at a much
  larger budget and still found a real, non-budget failure (the ratio
  problem's self-verification loop). Benchmark 3's failures were checked
  individually against the raw transcripts and are real bugs or genuine
  non-completions, not budget or extraction artifacts — see the per-function
  table above for exactly what went wrong in each case.
- Code extraction for Benchmark 3 looks for the last fenced ` ```python `
  block containing the target function name, falling back to a plain
  `def name(...):` block (with indentation-based scanning) for output that
  never uses markdown fences — both models got the same extraction logic, so
  neither is penalized for formatting style alone.
- `pp512`/`tg128` are `llama-bench` synthetic tests (fixed-length dummy
  prompt/generation), not end-to-end real-prompt latency — reported because
  they're the project's own standardized methodology, same as any llama.cpp
  benchmark table.

## Reproduce This

```bash
# CPU-only build, no GPU/CUDA required
llama-bench -m MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B.Q4_K_M.gguf -p 512 -n 128 -t 8

# Same test against the base model, for direct comparison
llama-bench -m DeepSeek-R1-Distill-Qwen-1.5B-Q4_K_M.gguf -p 512 -n 128 -t 8

# Single-turn generation (avoid interactive stdin loop when scripting/piping)
llama-cli -m <model>.gguf \
  -c 4096 --temp 0.6 --min-p 0.1 -no-cnv -st -n 500 \
  -p "<your prompt>"
```

*Measured 2026-08-15.*
