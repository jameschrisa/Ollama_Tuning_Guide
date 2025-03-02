# CPU Optimization Guide for Ollama Models

While GPUs get most of the attention when running large language models, many users will be running models primarily on their CPUs. Whether you don't have a compatible GPU or you're deliberately choosing CPU-only operation, this guide will help you squeeze every bit of performance from your processor.

## Understanding CPU-Based Inference

Let's start with a simple analogy: if running an LLM is like solving a giant puzzle, a GPU solves many small pieces in parallel, while a CPU methodically works through each complex piece one at a time. Neither approach is inherently better—they're just different approaches to the same problem.

CPU inference has some distinct advantages:
- Works on virtually any hardware
- More predictable performance
- Often more efficient for smaller models
- No VRAM limitations (just your system RAM)

## Assessing Your CPU Capabilities

Before optimizing, let's understand what we're working with:

```bash
# Check CPU information
lscpu

# On macOS
sysctl -a | grep machdep.cpu

# Check available memory
free -h
```

Look for:
- **Core count:** More cores = more parallel processing
- **Clock speed:** Higher frequency = faster sequential operations
- **Cache size:** Larger L3 cache = better performance with large models
- **AVX/AVX2/AVX-512 support:** Modern vector instructions dramatically improve LLM performance

## Key CPU Optimization Parameters

### Thread Count (`--threads`)

The most important parameter for CPU performance is the thread count:

```bash
# Example: Set thread count to 6
ollama run llama3:8b --threads 6
```

**Finding your optimal thread count:**

1. **Rule of thumb:** Start with (total CPU cores - 2)
2. **For shared systems:** Try using half your available cores
3. **For dedicated servers:** You can use all available cores

**Benchmarking for your CPU:**

```bash
# Try different thread counts and compare
ollama run llama3:8b --threads 4 "Tell me about AI" -f
ollama run llama3:8b --threads 6 "Tell me about AI" -f
ollama run llama3:8b --threads 8 "Tell me about AI" -f
```

Compare the tokens per second (tok/s) between runs to find your optimal setting.

### Batch Size (`--batch`)

The batch size affects how many tokens are processed in parallel:

```bash
# Example: Set a CPU-friendly batch size
ollama run llama3:8b --batch 128
```

For CPU-only operation:
- **Entry-level CPUs:** Try batch sizes between 64-128
- **High-performance CPUs:** Try batch sizes between 128-256
- **Server CPUs:** Can potentially go up to 512

Smaller batch sizes often work better on CPUs compared to GPUs, as CPUs generally have less parallel processing capability.

### Memory Mapping (`--mmap`)

Memory mapping controls whether the model is fully loaded into RAM or accessed from disk:

```bash
# Disable memory mapping to load fully into RAM
ollama run llama3:8b --mmap 0
```

**Guidelines:**
- **If RAM > (model size × 2):** Disable mmap (`--mmap 0`) for better performance
- **If RAM is limited:** Keep mmap enabled (default) to reduce memory usage

### NUMA Settings (`--numa`)

For multi-socket server CPUs, Non-Uniform Memory Access settings can significantly improve performance:

```bash
# Enable NUMA for multi-socket systems
ollama run llama3:8b --numa
```

Only enable this if you have a multi-socket server CPU. For most desktop and laptop users, this setting won't apply.

## CPU-Specific Model Selection

Not all models are created equal when it comes to CPU inference. Here are some models that perform well on CPUs:

| Model | Size | Performance Notes |
|-------|------|------------------|
| Phi-2 | 2.7B | Extremely efficient on CPU |
| Gemma | 2B | Google's model optimized for smaller devices |
| Mistral | 7B | Good balance of capability and efficiency |
| TinyLlama | 1.1B | Ultra-compact, great for weak CPUs |

For CPU-only operation, I recommend starting with smaller models (2B-7B) before attempting larger ones.

## Optimizing by CPU Type

### Budget/Older CPUs (4 cores or less)

```bash
# Configuration for older/budget CPUs
ollama run phi:2.7b --threads 2 --batch 64 --ctx 2048
```

**Strategy:**
- Use the smallest functional model for your task
- Severely limit context length
- Keep batch size small
- Leave threads for system operations

### Mid-Range CPUs (6-8 cores)

```bash
# Configuration for mid-range CPUs
ollama run mistral:7b --threads 4 --batch 128 --ctx 4096
```

**Strategy:**
- 7B models work well
- Moderate context length
- Balanced thread allocation

### High-Performance CPUs (12+ cores)

```bash
# Configuration for high-performance CPUs
ollama run llama3:8b --threads 10 --batch 256 --ctx 8192 --mmap 0
```

**Strategy:**
- Can run larger models effectively
- Increase batch size for better throughput
- Consider fully loading model into RAM

### Server CPUs (Xeon, EPYC, etc.)

```bash
# Configuration for server-class CPUs
ollama run llama3:13b --threads 24 --batch 512 --ctx 16384 --numa
```

**Strategy:**
- Leverage high core counts
- Enable NUMA for multi-socket systems
- Can run larger models with extended context

## CPU Architectures and Optimization

Different CPU architectures benefit from different optimizations:

### x86_64 (Intel/AMD)

Modern x86 processors benefit greatly from AVX instructions:

```bash
# Check for AVX support
grep -i avx /proc/cpuinfo
```

For Intel CPUs newer than 2011 or AMD CPUs newer than 2015, Ollama automatically uses AVX/AVX2 optimizations.

### ARM (Apple Silicon, Qualcomm, etc.)

ARM-based CPUs like Apple's M1/M2/M3 series have excellent performance characteristics:

```bash
# Optimize for M1/M2/M3 Macs
ollama run llama3:8b --threads 8
```

Apple Silicon particularly excels at running smaller models (7B-13B) and often outperforms similarly-priced x86 CPUs.

## Advanced: CPU + GPU Hybrid Operation

If you have a GPU with limited VRAM, consider a hybrid approach:

```bash
# Run some layers on GPU, rest on CPU
ollama run llama3:8b --gpu 16 --threads 6
```

This splits the model between your GPU and CPU, which can be more efficient than running entirely on either one.

## Monitoring CPU Performance

While running Ollama, monitor your CPU usage to ensure you're effectively using the hardware:

```bash
# Install htop if you don't have it
sudo apt install htop  # Ubuntu/Debian
brew install htop      # macOS with Homebrew

# Monitor CPU usage
htop
```

Look for:
- **CPU Utilization:** Should be high across all allocated cores during generation
- **Memory Usage:** Watch for any unexpected swapping
- **Temperature:** Keep below 85°C for sustained operation

## Common CPU Issues and Solutions

### Issue: High System Latency During Generation

**Solution:** Reduce thread count to leave cores for system operations
```bash
# If you were using 8 threads, try 6
ollama run llama3:8b --threads 6
```

### Issue: Generation Takes Forever to Start

**Solution:** Consider reducing model size or enabling memory mapping
```bash
# Enable memory mapping if loading takes too long
ollama run llama3:8b --mmap 1
```

### Issue: Memory Usage Keeps Growing

**Solution:** Limit context size and consider a smaller model
```bash
ollama run phi:2.7b --ctx 2048
```

### Issue: Poor Performance Despite Good CPU

**Solution:** Check if your model is quantized appropriately and verify thread settings
```bash
# Try a different quantization level
ollama run llama3:8b-q4 --threads 8
```

## Creating CPU-Optimized Custom Models

You can create CPU-optimized model configurations using custom Modelfiles:

```
FROM mistral:7b

# CPU-optimized parameters
PARAMETER num_thread 6
PARAMETER num_batch 128
PARAMETER num_ctx 4096
PARAMETER num_gpu 0

# Other parameters
PARAMETER temperature 0.7
PARAMETER top_p 0.9
```

Save as `Modelfile` and create your custom model:

```bash
ollama create cpu-mistral -f Modelfile
```

## Real-World CPU Optimization Examples

### Example 1: Office Laptop

**Hardware:** Core i5 (4 cores), 16GB RAM
**Use case:** Occasional document summarization

```bash
ollama run phi:2.7b --threads 2 --batch 64 --ctx 4096
```

**Result:** The smaller phi-2 model runs smoothly while leaving resources for other office tasks. The limited thread count prevents system slowdowns.

### Example 2: Home Server

**Hardware:** Ryzen 9 (16 cores), 64GB RAM
**Use case:** Family chatbot with multiple users

```bash
ollama run llama3:13b --threads 12 --batch 256 --ctx 8192 --mmap 0
```

**Result:** By loading the entire model into RAM and using most available cores, this setup provides responsive performance for the household.

### Example 3: Headless Server

**Hardware:** Dual Xeon (48 cores total), 256GB RAM
**Use case:** API endpoint for document processing

```bash
ollama run mixtral:8x7b --threads 40 --batch 512 --ctx 32768 --numa
```

**Result:** The powerful server CPU can handle even a mixture-of-experts model with very long context, providing high-quality responses for document analysis.

## Next Steps

Now that you've optimized your CPU settings, consider:

- [RAM Management Strategies](ram-management.md) for balancing system resources
- [Custom Modelfiles](../custom-modelfiles.md) to bake your optimized CPU parameters into a custom model
- [Parameter Sweep Testing](../parameter-sweep.md) to fine-tune your specific hardware

Remember that CPU optimization is iterative—what works best depends on your specific processor, workload, and model. Don't be afraid to experiment with different parameters and measure the results!
