# Documentation Index

Welcome to the MalxLabs-V4 documentation. Here you'll find detailed information about the project, setup, optimization, and research findings.

## 📚 Documentation Guide

### Start Here
- **[Main README](../README.md)** - Project overview, architecture, capabilities, and getting started
- **[Quick Start](QUICKSTART.md)** - Get up and running in 5 minutes

### Detailed Guides

| Document | Purpose |
|----------|---------|
| **[Advanced Configuration](ADVANCED_CONFIGURATION.md)** | Command-line parameters, optimization strategies, performance tuning |
| **[Research Findings](RESEARCH_FINDINGS.md)** | Empirical discoveries: Logic Horizon, hardware bottlenecks, sampling strategies |

### Contributing
- **[Contributing Guide](../CONTRIBUTING.md)** - How to contribute to the project

---

## Quick Navigation

### 🚀 I want to...

- **...set up MalxLabs-V4** → [Quick Start](QUICKSTART.md)
- **...understand the architecture** → [Main README - Model Architecture](../README.md#-model-architecture)
- **...optimize performance** → [Advanced Configuration](ADVANCED_CONFIGURATION.md)
- **...understand why things work this way** → [Research Findings](RESEARCH_FINDINGS.md)
- **...contribute improvements** → [Contributing Guide](../CONTRIBUTING.md)
- **...see performance metrics** → [Main README - Performance](../README.md#-performance-capabilities)
- **...learn about hardware** → [Main README - Hardware](../README.md#-hardware-specification)
- **...see code examples** → [Main README - Use Cases](../README.md#-use-cases)

---

## Key Concepts

### The "Logic Horizon"
Small models (1.5B params) have a limit to effective reasoning chains. See [Research Findings - Logic Horizon](RESEARCH_FINDINGS.md#1-the-logic-horizon-phenomenon) for details.

### Hardware Bottleneck
DDR3 1333MHz RAM limits context windows. Optimal context: 4096 tokens. See [Research Findings - Hardware Bottleneck](RESEARCH_FINDINGS.md#2-hardware-bottleneck-analysis) for analysis.

### Answer Leakage Fix
Min-P 0.1 sampling prevents the model from answering inside reasoning blocks. See [Research Findings - Leakage](RESEARCH_FINDINGS.md#3-the-leakage-phenomenon).

---

## File Structure

```
MalxLabs-V4/
├── README.md                          # Main project overview
├── CONTRIBUTING.md                    # Contribution guidelines
├── .gitignore                         # Git ignore rules
└── docs/
    ├── README.md                      # This file (documentation index)
    ├── QUICKSTART.md                  # Quick setup guide
    ├── ADVANCED_CONFIGURATION.md      # Parameter tuning & optimization
    └── RESEARCH_FINDINGS.md           # Empirical discoveries & analysis
```

---

## Document Summaries

### Quick Start Guide
- Prerequisites and installation
- Step-by-step setup
- Parameter explanations
- First test prompt

**Read time**: 5 minutes

### Advanced Configuration
- Complete parameter reference
- Recommended configurations (speed/quality/balance)
- Memory management strategies
- Sampling strategy tuning
- Troubleshooting guide
- Integration examples

**Read time**: 15-20 minutes

### Research Findings
- Logic Horizon phenomenon
- Hardware bottleneck analysis
- Answer Leakage problem and solution
- Sampling strategy impact
- GPU offloading optimization
- Temperature & reasoning trade-offs
- Testing methodology

**Read time**: 20-25 minutes

---

## Common Questions

**Q: What hardware do I need?**  
A: See [Hardware Specification](../README.md#-hardware-specification). Minimum: GTX 1080, 16GB RAM, i7-4790 or newer.

**Q: What context window should I use?**  
A: 4096 tokens is optimal. See [Research Findings - Hardware Bottleneck](RESEARCH_FINDINGS.md#2-hardware-bottleneck-analysis).

**Q: Why is my output rambling?**  
A: Use `--min-p 0.1`. See [Research Findings - Answer Leakage](RESEARCH_FINDINGS.md#3-the-leakage-phenomenon).

**Q: How do I get 80+ tokens/second?**  
A: Use full GPU offloading with `-ngl 35` and `--flash-attn on`. See [Advanced Configuration](ADVANCED_CONFIGURATION.md).

**Q: Can I use higher context windows?**  
A: Not recommended on legacy hardware. See [Research Findings - Hardware Bottleneck](RESEARCH_FINDINGS.md#2-hardware-bottleneck-analysis).

**Q: What's the best temperature setting?**  
A: 0.6 for balanced reasoning. See [Advanced Configuration - Sampling Strategy](ADVANCED_CONFIGURATION.md#sampling-strategy-tuning).

---

## Quick Reference: Essential Parameters

```bash
-m ~/models/logic_model.gguf    # Model file (required)
-ngl 35                          # Full GPU offloading (essential)
-c 4096                          # Optimal context window
--temp 0.6                       # Temperature (reasoning-focused)
--min-p 0.1                      # Prevents answer leakage
--flash-attn on                  # Performance boost
--mlock                          # Prevent swapping
--reasoning-format deepseek      # Proper formatting
```

See [Advanced Configuration](ADVANCED_CONFIGURATION.md) for complete reference.

---

## Getting Help

- **Setup issues** → Check [Quick Start](QUICKSTART.md)
- **Performance problems** → See [Advanced Configuration - Troubleshooting](ADVANCED_CONFIGURATION.md#troubleshooting)
- **Unexpected behavior** → Review [Research Findings](RESEARCH_FINDINGS.md)
- **Contributing** → Read [Contributing Guide](../CONTRIBUTING.md)

---

**Last Updated**: 2026-05-13  
**Documentation Version**: 1.0
