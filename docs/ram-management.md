# RAM Management Strategies for Ollama

Memory management is one of the biggest challenges when running large language models locally. Even with quantized models, you can quickly hit memory limits if you're not careful. This guide will help you balance the memory needs of Ollama models with your available RAM.

Think of RAM management like arranging furniture in a small apartment – with smart planning, you can make even a limited space work efficiently, but you need to be intentional about what goes where.

## Understanding Memory Usage in Ollama

Let's start by understanding what consumes memory when running an LLM:

1. **Model Weights** - The parameters of the neural network itself
2. **KV Cache** - Stores past token computations (scales with context length)
3. **Temporary Buffers** - Working memory for calculations during generation
4. **Operating System** - Your computer needs memory for other processes

## Checking Your Available Memory

Before optimizing, let's see what you're working with:

```bash
# On Linux
free -h

# On macOS
vm_stat

# On Windows (PowerShell)
Get-CimInstance Win32_OperatingSystem | Select-Object FreePhysicalMemory,TotalVisibleMemorySize
```

## Estimating Model Memory Requirements

Here's a rough estimate of memory requirements for different model sizes with various quantization levels:

| Model Size | Q4 Quantized | Q5 Quantized | Q8 Quantized | FP16 |
|------------|--------------|--------------|--------------|------|
| 3B         | ~2GB         | ~2.5GB       | ~3GB         | ~6GB |
| 7B         | ~4GB         | ~5GB         | ~7GB         | ~14GB|
| 13B        | ~8GB         | ~9GB         | ~13GB        | ~26GB|
| 34B        | ~20GB        | ~23GB        | ~34GB        | ~68GB|
| 70B        | ~40GB        | ~47GB        | ~70GB        | ~140GB|

> **Note:** These are base model sizes only! Your actual usage will be higher due to the KV cache and other runtime allocations.

## Key Memory Management Parameters

### Context Length (`--ctx`)

The context length has a *massive* impact on memory usage:

```bash
# Example: Reduce context to save memory
ollama run llama3:8b --ctx 2048
```

**Memory impact by context size:**
- 2048 tokens: Baseline memory usage
- 4096 tokens: ~2× more KV cache than 2048
- 8192 tokens: ~4× more KV cache than 2048
- 16384 tokens: ~8× more KV cache than 2048

**When to adjust:**
- **Reduce** when you need to conserve memory
- **Increase** only when you truly need longer context (e.g., for document analysis)

### Memory Mapping (`--mmap`)

Memory mapping controls whether the model is fully loaded into RAM or accessed from disk:

```bash
# Example: Enable memory mapping to reduce RAM usage
ollama run llama3:8b --mmap 1
```

**How it works:**
- When enabled (default), the model weights are memory-mapped from disk
- When disabled, the entire model is loaded into RAM

**Tradeoffs:**
- Enabled (`--mmap 1`): Lower memory usage, potentially slower performance
- Disabled (`--mmap 0`): Higher memory usage, potentially faster performance

**When to adjust:**
- **Enable** (default) when RAM is limited
- **Disable** when you have abundant RAM and want maximum performance

### GPU Layers (`--gpu`)

While primarily a GPU parameter, the GPU/CPU split also affects system RAM usage:

```bash
# Example: Offload more layers to GPU to reduce CPU RAM usage
ollama run llama3:8b --gpu 32
```

**Memory impact:**
- More layers on GPU = Less system RAM usage
- Fewer layers on GPU = More system RAM usage

**When to adjust:**
- If system RAM is limited but you have available VRAM, increase GPU layers
- If VRAM is limited but you have abundant system RAM, decrease GPU layers

## Memory Management Strategies by RAM Availability

### Low RAM (8GB)

With 8GB of RAM, you need to be very conservative:

```bash
# Configuration for 8GB RAM systems
ollama run phi:2.7b --ctx 2048 --mmap 1
```

**Strategy:**
- Stick to small models (3B-7B parameters)
- Use Q4 quantization when available
- Keep context length minimal (2048 or less if possible)
- Always use memory mapping
- Close other memory-intensive applications
- Consider using swap space (but expect slower performance)

### Medium RAM (16GB)

With 16GB, you have more flexibility:

```bash
# Configuration for 16GB RAM systems
ollama run llama3:8b --ctx 4096 --mmap 1
```

**Strategy:**
- 7B-13B models work well
- Use moderate context lengths (4096-8192)
- Memory mapping recommended but optional for 7B models
- If you have a GPU, balance layers between GPU and CPU

### High RAM (32GB+)

With 32GB or more, you can run most models comfortably:

```bash
# Configuration for 32GB+ RAM systems
ollama run llama3:70b --ctx 8192 --mmap 1
```

**Strategy:**
- Can run larger models (up to 70B)
- Can use longer contexts (8192-16384)
- Memory mapping optional but still recommended for largest models
- If you have a GPU, optimize for performance rather than memory usage

### Server RAM (64GB+)

With server-level RAM, few limitations apply:

```bash
# Configuration for server RAM levels
ollama run llama3:70b --ctx 32768 --mmap 0
```

**Strategy:**
- Run any model size with confidence
- Can use very long contexts when needed
- Can disable memory mapping for maximum performance
- Consider running multiple models simultaneously

## Advanced Memory Optimization Techniques

### Shared Memory for Multiple Models

If running multiple models, you can take advantage of shared libraries:

```bash
# Load one model first
ollama run llama3:8b --keepalive 300000

# In another terminal, load a related model
ollama run codellama:7b
```

Models from the same family often share code, reducing the total memory footprint.

### Using Swap Space Effectively

If you're memory-constrained, configure swap space appropriately:

```bash
# On Linux, check swap configuration
swapon -s

# Create a 8GB swap file if needed
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Swap will be slower than RAM but can prevent out-of-memory crashes.

### Model Unloading and Reloading

Be mindful of the `--keepalive` parameter, which controls how long a model stays in memory:

```bash
# Keep model loaded for only 10 seconds after completion
ollama run llama3:8b --keepalive 10000
```

Lower values free up memory sooner, higher values keep the model ready for subsequent queries.

## Monitoring Memory Usage

While running Ollama, keep an eye on memory consumption:

```bash
# On Linux
watch -n 1 'free -h'

# On macOS
top -o mem

# On Windows (PowerShell)
while($true) {Get-Process ollama | Select-Object WorkingSet; Start-Sleep 1}
```

## Real-World RAM Management Examples

### Example 1: Budget Laptop

**Hardware:** 8GB RAM, older CPU, no dedicated GPU
**Use case:** Personal assistant for text queries

```bash
ollama run phi:2.7b --ctx 2048 --mmap 1 --gpu 0
```

**Result:** The Phi-2 model is small enough to run comfortably within 8GB, especially with limited context. Memory mapping ensures the system remains responsive.

### Example 2: Gaming Desktop

**Hardware:** 16GB RAM, RTX 3060 (12GB VRAM)
**Use case:** Chat and creative writing

```bash
ollama run llama3:8b --ctx 4096 --gpu 24 --mmap 1
```

**Result:** By offloading many layers to the GPU, you reduce system RAM usage while maintaining good performance. The moderate context size balances memory usage with conversational needs.

### Example 3: Developer Workstation

**Hardware:** 64GB RAM, RTX 4090 (24GB VRAM)
**Use case:** Programming assistant with long context

```bash
ollama run codellama:34b --ctx 16384 --gpu 60 --mmap 0
```

**Result:** With abundant RAM and VRAM, you can run a large specialized model with extended context, fully loaded into RAM for maximum performance.

## Common Memory Issues and Solutions

### Issue: "Out of Memory" Errors

**Solution:** Reduce model size, context length, or enable memory mapping
```bash
# Try a smaller model
ollama run mistral:7b-q4 --ctx 2048
```

### Issue: System Becoming Sluggish

**Solution:** Lower the keepalive time and close unused models
```bash
# Release memory more quickly after completion
ollama run llama3:8b --keepalive 5000
```

### Issue: Swapping Causing Poor Performance

**Solution:** Choose a smaller model or reduce context window
```bash
# More conservative parameters to avoid swapping
ollama run phi:2.7b --ctx 1024
```

### Issue: Initial Model Loading Takes Too Long

**Solution:** For frequently used models, consider creating a service to keep them loaded
```bash
# Script to preload model at system startup
ollama run llama3:8b --keepalive 86400000  # Keep alive for 24 hours
```

## Creating a Memory-Optimized Modelfile

You can bake memory optimizations into a custom Modelfile:

```
FROM mistral:7b

# Memory-optimized parameters
PARAMETER num_ctx 4096
PARAMETER mmap 1

# Other parameters
PARAMETER temperature 0.7
```

Save as `Modelfile` and create your custom model:

```bash
ollama create memory-optimized-mistral -f Modelfile
```

## RAM Requirements by Use Case

Different use cases have different optimal memory configurations:

### Basic Chat Application
- **Minimum RAM:** 8GB
- **Recommended model:** Phi-2 (2.7B), Mistral (7B)
- **Context needs:** 2048-4096 tokens
- **Example setup:** `ollama run mistral:7b --ctx 4096 --mmap 1`

### Code Assistant
- **Minimum RAM:** 16GB
- **Recommended model:** CodeLlama (7B-13B)
- **Context needs:** 8192-16384 tokens
- **Example setup:** `ollama run codellama:13b --ctx 8192 --mmap 1`

### Document Analysis
- **Minimum RAM:** 32GB
- **Recommended model:** Llama 3 (70B), Mixtral (8x7B)
- **Context needs:** 16384-32768 tokens
- **Example setup:** `ollama run llama3:70b --ctx 16384 --mmap 1`

## Next Steps

Now that you understand how to manage RAM with Ollama, consider exploring:

- [Hardware-Specific Optimization](hardware-specific-optimization.md) for a broader view of hardware considerations
- [CPU Optimization Guide](cpu-optimization.md) for performance tuning on CPU-bound systems
- [GPU Optimization Guide](gpu-optimization.md) for leveraging your GPU to reduce system RAM pressure

Remember that memory optimization is about finding the right balance for your specific hardware and use case. Start with conservative settings, then gradually increase parameters as you better understand your system's capabilities.
