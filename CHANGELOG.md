# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased] - 2026-08-15

### Added
- **Benchmark suite for hardware testing** (planned item from the roadmap
  below, now delivered): real, measured CPU-only inference benchmark on a
  second machine with no discrete GPU (Intel i5-1035G7 / Iris Plus).
- **Real head-to-head vs. the untuned base model** (deepseek-ai/DeepSeek-R1-
  Distill-Qwen-1.5B via bartowski's GGUF): same hardware, same llama.cpp
  settings, same 8-problem accuracy spot-check, run back to back. Throughput
  is a wash (within noise of each other); MalxLabs-V4 reaches a correct final
  answer within a 500-token budget 8/8 vs. the base model's 6/8 — documented
  honestly, including that the base model's two "misses" were budget
  truncation, not wrong answers (it reaches 8/8 too with more budget).
- `docs/HARDWARE_BENCHMARK_2026-08.md` — full methodology, throughput
  (`llama-bench` pp512/tg128), memory footprint, the MalxLabs-V4-vs-base
  spot-check, and a reproduction of the README's own example prompt.
- `docs/benchmarks/` — light/dark SVG comparison charts (parameter count vs.
  6 similarly-sized small models; published GSM8K scores + our own measured
  MalxLabs-V4-vs-base spot-check; real hardware throughput dashboard),
  embedded in the README via `<picture>`.
- New README section: [📊 Benchmarks & Comparisons](README.md#-benchmarks--comparisons).

## [1.0.0] - 2026-05-13

### Added
- Initial release of MalxLabs-V4
- DeepSeek-R1-1.5B model fine-tuned for technical reasoning
- Full documentation suite:
  - Comprehensive README with architecture, hardware specs, and performance metrics
  - Quick Start guide for setup and deployment
  - Advanced Configuration guide with parameter reference
  - Research Findings documentation detailing empirical discoveries
  - Contributing guidelines
- .gitignore for Python, models, and build artifacts
- CHANGELOG tracking (this file)
- Documentation archive of original research notes

### Features
- Full GPU offloading support (35 layers on GTX 1080)
- Flash Attention optimization for 15-20% speed boost
- Min-P 0.1 sampling to prevent answer leakage
- Optimal 4096-token context window for legacy hardware
- 80+ tokens per second inference on GTX 1080
- High-quality mathematical reasoning capabilities
- Proficient code generation in Luau and Python

### Optimizations
- Context window tuning for DDR3 1333MHz RAM constraints
- Temperature setting (0.6) optimized for reasoning
- GPU/CPU task distribution minimizing memory bandwidth bottlenecks

### Documentation
- Main README with visual badges and structured sections
- Quick Start setup guide
- Advanced Configuration with multiple optimization profiles
- Research Findings with empirical discovery analysis
- Comprehensive parameter reference guide
- Troubleshooting section
- Integration examples (Python, JavaScript)

---

## Future Roadmap

### Planned Features
- [ ] Additional fine-tuning for domain-specific tasks
- [ ] Quantization variants (Q3_K_M, Q5_K_M comparison)
- [ ] Continuous batching for multiple prompt processing
- [ ] Custom prompt templates for specific domains
- [ ] Performance profiling tools
- [x] Benchmark suite for hardware testing — see `docs/HARDWARE_BENCHMARK_2026-08.md`

### Research Areas
- [ ] Speculative decoding implementation
- [ ] Token caching across similar queries
- [ ] Extended context window strategies
- [ ] Multi-model ensemble approaches
- [ ] Alternative quantization methods

---

## Version History

### v1.0.0 (Current)
- Initial stable release
- All core features operational
- Complete documentation
- Tested on legacy hardware (GTX 1080, i7-4790)

---

## Notes

- All changes are focused on optimizing reasoning and logic on legacy hardware
- Performance metrics are based on GTX 1080, i7-4790, 16GB DDR3 1333MHz configuration
- Documentation reflects empirical testing and real-world findings

