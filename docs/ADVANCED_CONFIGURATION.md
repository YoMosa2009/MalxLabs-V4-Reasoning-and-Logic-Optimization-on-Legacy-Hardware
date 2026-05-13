# Advanced Configuration & Optimization

## Command-Line Parameter Reference

### Core Parameters

```bash
# Model and context
-m <path>              # Path to GGUF model file (required)
-c <tokens>            # Context window size (default: 2048, max: 4096 on legacy hw)
-n <tokens>            # Number of tokens to generate (default: 128)

# GPU acceleration
-ngl <layers>          # Number of layers to offload to GPU (use 35 for full offload)
-ts <size>             # Tensor split for multi-GPU (not needed for single GPU)

# Sampling parameters
--temp <value>         # Temperature for sampling (0.0-2.0, default: 0.7, optimal: 0.6)
--min-p <value>        # Min-P threshold for sampling (0.0-1.0, optimal: 0.1)
--top-p <value>        # Top-P (nucleus sampling, default: 0.9)
--top-k <value>        # Top-K filtering (default: 40)
--repeat-penalty <v>   # Repetition penalty (default: 1.1)

# Memory and performance
--mlock                # Lock model in RAM (prevents swapping)
--mmap                 # Memory-map model file (recommended)
--flash-attn           # Enable Flash Attention (on/off)

# Prompt and formatting
--reasoning-format     # Specify reasoning format (deepseek)
--jinja                # Enable Jinja template processing
-p <prompt>            # Initial system prompt
```

---

## Recommended Configurations

### 1. Balanced Performance (Recommended)

```bash
./build/bin/llama-cli \
  -m ~/models/logic_model.gguf \
  -ngl 35 \
  -c 4096 \
  --temp 0.6 \
  --min-p 0.1 \
  --top-p 0.95 \
  --top-k 40 \
  --repeat-penalty 1.1 \
  --flash-attn on \
  --mlock \
  --mmap \
  --reasoning-format deepseek \
  --jinja
```

**Use Case**: General-purpose reasoning and code generation  
**Speed**: 80+ tokens/second  
**Quality**: High  
**Memory**: ~4.5GB VRAM

---

### 2. Maximum Speed Configuration

```bash
./build/bin/llama-cli \
  -m ~/models/logic_model.gguf \
  -ngl 35 \
  -c 2048 \
  --temp 0.4 \
  --min-p 0.1 \
  --top-p 0.9 \
  --top-k 20 \
  --repeat-penalty 1.0 \
  --flash-attn on \
  --mlock
```

**Use Case**: Time-critical applications, real-time interaction  
**Speed**: 90+ tokens/second  
**Context**: Limited to 2048 tokens  
**Memory**: ~3GB VRAM

---

### 3. Maximum Quality Configuration

```bash
./build/bin/llama-cli \
  -m ~/models/logic_model.gguf \
  -ngl 35 \
  -c 4096 \
  --temp 0.5 \
  --min-p 0.05 \
  --top-p 0.98 \
  --top-k 50 \
  --repeat-penalty 1.15 \
  --flash-attn on \
  --mlock \
  --mmap \
  --reasoning-format deepseek \
  --jinja
```

**Use Case**: Complex reasoning, critical calculations  
**Speed**: 70+ tokens/second  
**Quality**: Maximum diversity and reasoning depth  
**Memory**: ~4.5GB VRAM

---

## Memory Management for Legacy Hardware

### RAM Constraints (DDR3 1333MHz)

The 16GB DDR3 1333MHz RAM is the bottleneck. Strategies to optimize:

#### 1. Model in GPU Memory

- ✅ Keep all 35 layers on GPU (`-ngl 35`)
- ✅ Prevents CPU-GPU transfers
- ✅ Achieves 80+ tokens/second

#### 2. System Memory Optimization

```bash
# Linux: Check current memory usage
free -h

# Pre-allocate GPU memory
# Enable mlock to prevent swapping
--mlock

# Use memory mapping
--mmap
```

#### 3. Context Window Trade-offs

| Context | RAM Cost | DDR3 Impact | Max Speed |
|---------|----------|-------------|-----------|
| 2048 | 1.5GB | Minimal | 90+ tokens/s |
| 4096 | 3-4GB | Low | 80+ tokens/s |
| 8192 | 6-8GB | Moderate | 40-60 tokens/s |
| 16384 | 12-16GB | **Critical** | 5-10 tokens/s (hangs) |

**Recommendation**: Stay at 4096 for best balance.

---

## Sampling Strategy Tuning

### Understanding the Sampling Pipeline

```
Model Output Logits
    ↓
Temperature Scaling (--temp)
    ↓
Top-K Filtering (--top-k)
    ↓
Top-P (Nucleus) Filtering (--top-p)
    ↓
Min-P Filtering (--min-p)
    ↓
Repetition Penalty (--repeat-penalty)
    ↓
Final Token Selection
```

### Temperature Tuning for Different Tasks

| Task | Optimal Temp | Reasoning |
|------|--------------|-----------|
| Math Problems | 0.4 - 0.5 | Deterministic, logical |
| Code Generation | 0.5 - 0.6 | Balance precision with variety |
| Logic Puzzles | 0.5 - 0.7 | Some creativity needed |
| Creative Writing | 0.7 - 0.9 | Higher diversity |
| Data Analysis | 0.3 - 0.4 | Precise, conservative |

### Min-P Threshold Adjustment

**Current: 0.1** (Optimal for reasoning)

- **Too Low (0.0 - 0.05)**: Answer leakage occurs, model answers in `<think>` blocks
- **Sweet Spot (0.08 - 0.12)**: Clean thinking/output separation
- **Too High (0.15+)**: Reduces output diversity, over-conservative

---

## Performance Profiling

### Measuring Tokens Per Second

```bash
# llama.cpp reports metrics automatically
# Look for output like:
# "eval time = 4500.00 ms / 60 tokens (75.00 ms / token)"
# This shows ~13 tokens/second for generation
# Or ~80 tokens/s during fast batching

# Manual profiling
time ./build/bin/llama-cli -m model.gguf -p "test" -n 100
```

### Benchmarking Your Setup

```bash
#!/bin/bash
# benchmark.sh

echo "=== MalxLabs-V4 Performance Benchmark ==="
echo "Context: 4096, Temp: 0.6, Min-P: 0.1"
echo ""

./build/bin/llama-cli \
  -m ~/models/logic_model.gguf \
  -ngl 35 \
  -c 4096 \
  -n 500 \
  --temp 0.6 \
  --min-p 0.1 \
  --flash-attn on \
  --mlock \
  -p "You are a helpful assistant. Answer the following question:"
```

---

## Troubleshooting

### Issue: Prefill Hangs (Context 8k+)

**Symptom**: Long delays before generation starts with high context

**Solution**:
```bash
# Reduce context to 4096
-c 4096

# Or enable memory mapping
--mmap
```

### Issue: "Rambling" in Output

**Symptom**: Model generates repetitive or circular reasoning

**Solution**:
```bash
# Lower temperature
--temp 0.4

# Tighten Min-P
--min-p 0.1

# Reduce Max tokens
-n 200
```

### Issue: Out of VRAM Errors

**Symptom**: "CUDA out of memory" error

**Solution**:
```bash
# Reduce context window
-c 2048

# Or reduce layers on GPU (not recommended)
-ngl 30
```

### Issue: Low Throughput (< 50 tokens/s)

**Symptom**: Slow inference even with GPU offloading

**Check**:
1. Verify GPU offloading: `-ngl 35` in command
2. Enable Flash Attention: `--flash-attn on`
3. Check for disk I/O: Ensure model is cached
4. Monitor CPU usage: Should be minimal

---

## Integration Examples

### Python Integration (via llama-cpp-python)

```python
from llama_cpp import Llama

llm = Llama(
    model_path="~/models/logic_model.gguf",
    n_gpu_layers=35,
    n_ctx=4096,
    temperature=0.6,
    top_p=0.95,
    min_p=0.1,
    verbose=True
)

# Generate text
output = llm("What is 5 + 7?", max_tokens=100)
print(output)
```

### JavaScript/Node.js Integration

```javascript
const { Llama } = require('node-llama-cpp');

const model = new Llama({
  modelPath: 'models/logic_model.gguf',
  gpuLayers: 35,
  contextSize: 4096,
  temperature: 0.6,
  minP: 0.1,
});

const response = model.generate("What is 5 + 7?");
console.log(response);
```

---

## Further Optimization Potential

### Potential Improvements (Not Yet Tested)

1. **Continuous Batching**: Process multiple prompts in sequence
2. **Speculative Decoding**: Use smaller model for draft, refine with V4
3. **Quantization Variants**: Test Q3_K_M vs current Q4_K_M
4. **Custom Prompt Templates**: Optimize for specific domains
5. **Token Caching**: Reuse KV cache across similar queries

