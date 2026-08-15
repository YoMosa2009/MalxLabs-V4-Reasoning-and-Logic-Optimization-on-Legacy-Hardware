# Hardware Benchmark: Non-GPU Legacy Laptop (2026-08)

## Overview

Every number in this document was actually measured by running MalxLabs-V4 with
`llama.cpp` on the hardware listed below — not estimated or copied from the main
README's GTX 1080 numbers. It follows the [CONTRIBUTING.md](../CONTRIBUTING.md)
hardware-testing template and directly addresses the roadmap item *"Benchmark
suite for hardware testing"* from [CHANGELOG.md](../CHANGELOG.md).

This test targets the opposite end of "legacy" from the main README: instead of
an aging **discrete GPU** (GTX 1080, Pascal, 2016), this rig has **no discrete
GPU at all** — a 2019 ultrabook CPU with integrated graphics only. It answers a
different question than the README: *does MalxLabs-V4 still work — not fast,
just work — on hardware with zero CUDA/ROCm path?*

## Hardware Configuration

| Component | Specification |
|---|---|
| **CPU** | Intel Core i5-1035G7 (Ice Lake), 4 cores / 8 threads, 1.2 GHz base |
| **GPU** | Intel Iris Plus Graphics (integrated) — **no discrete GPU, no CUDA/ROCm** |
| **RAM** | 16 GB |
| **OS** | Windows 11 Pro |
| **Inference engine** | `llama.cpp` build **b10448**, official `win-cpu-x64` release binary (icelake-optimized `ggml-cpu-icelake.dll` backend, auto-selected) |
| **Model** | [MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B](https://huggingface.co/MalxTech/MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B), Q4_K_M GGUF, 1.04 GiB |

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
   same sampling settings, `-n 500`. Final answer extracted programmatically
   from `ANSWER: <n>` or `\boxed{<n>}` and compared to the known answer.
4. **Memory** — sampled `Get-Process` working set on the live `llama-cli`
   process mid-generation at `-c 4096`.

## Results

### Throughput (measured, `llama-bench`)

| Test | Speed | Compare to README's GTX 1080 claim |
|---|---|---|
| Prompt processing (pp512) | **61.48 ± 4.62 tok/s** | not directly comparable (README doesn't split pp/tg) |
| Text generation (tg128) | **14.50 ± 2.24 tok/s** | README claims 80+ tok/s w/ `-ngl 35` full GPU offload |

CPU-only generation lands at roughly **1/5th** the GTX 1080's claimed
generation speed — expected, given there is no GPU at all here. The model is
still fully usable for interactive single-turn reasoning at this speed.

### Memory footprint (measured)

- **~1.7 GB working set** for the `llama-cli` process at `-c 4096`, Q4_K_M
  weights, CPU-only — comfortably inside this machine's 16 GB.

### Own accuracy spot-check (measured, n=8)

**8 / 8 correct.** Full per-problem detail:

| # | Problem type | Expected | Model answer | Correct |
|---|---|---|---|---|
| 1 | Two-step addition/division word problem | 72 | 72 | ✅ |
| 2 | Fraction-of-a-quantity word problem | 3 | 3 | ✅ |
| 3 | Rate × time word problem | 10 | 10 | ✅ |
| 4 | Multi-step multiplication word problem | 624 | 624 | ✅ |
| 5 | Sequential arithmetic word problem | 29 | 29 | ✅ |
| 6 | Rate × time word problem | 180 | 180 | ✅ |
| 7 | Multiplicative comparison word problem | 42 | 42 | ✅ |
| 8 | Proportion/scaling word problem | 5 | 5 | ✅ |

**This is not a formal GSM8K evaluation** — 8 hand-picked problems, single-shot,
one run — and shouldn't be read as directly comparable to the published 5-shot
GSM8K scores in the comparison chart below. It's a real, reproducible spot-check
of basic arithmetic reasoning on this exact hardware, nothing more.

One notable format quirk: despite being prompted for `ANSWER: <number>`, the
model consistently answered in `\boxed{<number>}` LaTeX notation instead —
almost certainly a habit from its `OpenMathInstruct-2` fine-tuning data. Worth
knowing if you're scripting against this model's output.

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

## Comparison to Similar Small Reasoning Models

Charts below compare MalxLabs-V4 against publicly documented models in the same
weight class. Parameter counts and benchmark scores are **sourced from each
model's official card / paper** (linked in each chart's footer) — not
independently re-run by us except where explicitly labeled "measured."

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart1_params_dark.svg">
  <img src="benchmarks/chart1_params_light.svg" alt="Parameter count comparison: MalxLabs-V4 and six similarly-sized small models">
</picture>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart2_gsm8k_dark.svg">
  <img src="benchmarks/chart2_gsm8k_light.svg" alt="GSM8K published benchmark score comparison across small instruct models">
</picture>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="benchmarks/chart3_hardware_dark.svg">
  <img src="benchmarks/chart3_hardware_light.svg" alt="MalxLabs-V4 real hardware benchmark dashboard on the Intel i5-1035G7 laptop">
</picture>

## Limitations & Honesty Notes

- We did **not** download or benchmark the comparison models ourselves on this
  hardware — their numbers are published figures, cited per chart. Only
  MalxLabs-V4's own numbers here are first-party measurements.
- The 8-problem spot-check is a sanity check, not a benchmark suite. A
  contributor running the actual GSM8K test set (see `CONTRIBUTING.md`) against
  MalxLabs-V4 would be a genuinely useful follow-up.
- `pp512`/`tg128` are `llama-bench` synthetic tests (fixed-length dummy
  prompt/generation), not end-to-end real-prompt latency — reported because
  they're the project's own standardized methodology, same as any llama.cpp
  benchmark table.

## Reproduce This

```bash
# CPU-only build, no GPU/CUDA required
llama-bench -m MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B.Q4_K_M.gguf -p 512 -n 128 -t 8

# Single-turn generation (avoid interactive stdin loop when scripting/piping)
llama-cli -m MalxLabs-DeepSeek-R1-Distill-Qwen-1.5B.Q4_K_M.gguf \
  -c 4096 --temp 0.6 --min-p 0.1 -no-cnv -st -n 500 \
  -p "<your prompt>"
```

*Measured 2026-08-15.*
