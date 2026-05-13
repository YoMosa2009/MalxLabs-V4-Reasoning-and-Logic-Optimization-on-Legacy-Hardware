# Quick Start Guide

## Prerequisites

- **llama.cpp**: Compiled and ready to use
- **Model File**: `logic_model.gguf` (DeepSeek-R1-1.5B, Q4_K_M quantization)
- **Hardware**: 
  - NVIDIA GPU (GTX 1080 or better, 8GB VRAM minimum)
  - 16GB RAM
  - Modern CPU (Skylake or newer recommended)

## Installation

### 1. Setup llama.cpp

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make
```

### 2. Download/Place Model

Place your `logic_model.gguf` in the models directory:
```bash
mkdir -p ~/models
# Copy your GGUF model to ~/models/logic_model.gguf
```

### 3. Run Local Inference

```bash
cd ~/llama.cpp && ./build/bin/llama-cli \
  -m ~/models/logic_model.gguf \
  -ngl 35 \
  -c 4096 \
  --flash-attn on \
  --mlock \
  --temp 0.6 \
  --min-p 0.1 \
  --reasoning-format deepseek \
  --jinja \
  -p "<｜User｜>[INSERT PROMPT]<｜Assistant｜>"
```

## Parameter Explanation

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `-m` | Model path | Specifies the GGUF model file |
| `-ngl` | 35 | GPU layers (full GPU offloading for GTX 1080) |
| `-c` | 4096 | Context window size (optimized for legacy hardware) |
| `--flash-attn` | on | Enables flash attention for speed |
| `--mlock` | — | Lock model in RAM to prevent swapping |
| `--temp` | 0.6 | Temperature (controls randomness) |
| `--min-p` | 0.1 | Min-P sampling (prevents rambling) |
| `--reasoning-format` | deepseek | Formats for DeepSeek reasoning |
| `--jinja` | — | Enables Jinja template processing |

## First Test

Try a simple test prompt:

```
User: Calculate the sum of 5 + 7