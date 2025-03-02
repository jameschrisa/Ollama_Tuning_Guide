# Core Parameters Explained

When working with Ollama models, understanding the key hyperparameters is essential for optimal performance. Think of these parameters as the control levers that determine how efficiently your model runs and how it uses your system resources.

## Key Hyperparameters in Ollama

| Parameter | CLI Flag | Description | Default |
|-----------|----------|-------------|---------|
| Context Length | `--ctx` | Maximum token context window | Model-specific |
| Batch Size | `-b` or `--batch` | Number of tokens processed simultaneously | 512 |
| GPU Layers | `-g` or `--gpu` | Number of layers to offload to GPU | 0 (CPU only) |
| Thread Count | `-t` or `--threads` | Number of CPU threads to use | System-dependent |
| Keep in Memory | `--keepalive` | How long to keep model loaded (ms) | 0 |
| NUMA Configuration | `--numa` | Enable Non-Uniform Memory Access | Disabled |
| Temperature | `--temp` | Sampling temperature (creativity) | 0.8 |
| Top-P | `--top-p` | Nucleus sampling threshold | 0.9 |
| Memory Mapping | `--mmap` | Use memory mapping for model weights | Enabled |

## Detailed Explanation

### Context Length (`--ctx`)

**What it does:** Determines how many tokens the model can "remember" during a conversation or text generation task.

**Technical details:** The context window acts as the model's working memory. Larger values allow for longer conversations and more comprehensive responses, but significantly increase memory usage.

**Adjustment guidelines:**
- **Increase when:** You need the model to maintain longer conversation history or process larger documents
- **Decrease when:** You're facing memory constraints or only need short exchanges
- **Memory impact:** Memory usage scales approximately linearly with context length

```bash
# Example: Setting a 4096 token context window
ollama run llama3 --ctx 4096
```

### Batch Size (`--batch`)

**What it does:** Controls how many tokens are processed in parallel during generation.

**Technical details:** Higher batch sizes can improve throughput but require more memory. This is different from the number of tokens generated in total.

**Adjustment guidelines:**
- **Increase when:** You want faster generation on capable hardware
- **Decrease when:** You're experiencing memory limitations
- **Performance impact:** Higher values generally mean faster generation, up to hardware limits

```bash
# Example: Setting batch size to 256
ollama run llama3 --batch 256
```

### GPU Layers (`--gpu`)

**What it does:** Determines how many of the model's neural network layers run on your GPU instead of CPU.

**Technical details:** Modern LLMs have dozens of layers. This parameter lets you specify how many should be offloaded to the GPU, with the remainder staying on the CPU.

**Adjustment guidelines:**
- **Increase when:** You have a capable GPU with sufficient VRAM
- **Decrease when:** You're experiencing GPU memory issues
- **Set to 0:** To run exclusively on CPU
- **Set to total layer count:** To run fully on GPU

```bash
# Example: Run all layers on GPU
ollama run llama3 --gpu 32

# Example: Split between GPU and CPU
ollama run llama3 --gpu 16

# Example: CPU only
ollama run llama3 --gpu 0
```

### Thread Count (`--threads`)

**What it does:** Sets how many CPU threads the model uses for computation.

**Technical details:** More threads can improve performance, but only up to a point. Setting too many threads can actually degrade performance.

**Adjustment guidelines:**
- **General rule:** Start with (total CPU cores - 2) to leave resources for your OS
- **Decrease when:** Running multiple models simultaneously or other applications need CPU resources

```bash
# Example: Use 6 CPU threads
ollama run llama3 --threads 6
```

### Temperature (`--temp`)

**What it does:** Controls the randomness/creativity in the model's responses.

**Technical details:** Higher values make output more random, creative, and diverse. Lower values make output more deterministic and focused.

**Adjustment guidelines:**
- **Increase (0.8-1.0):** For creative writing, brainstorming, or casual conversation
- **Decrease (0.1-0.4):** For factual responses, code generation, or logical reasoning

```bash
# Example: Lower temperature for more deterministic responses
ollama run llama3 --temp 0.2

# Example: Higher temperature for creative responses
ollama run llama3 --temp 0.9
```

### Top-P (Nucleus Sampling) (`--top-p`)

**What it does:** Filters the token selection to only consider the most likely tokens whose cumulative probability exceeds the top-p value.

**Technical details:** A value of 0.9 means "only consider tokens from the top 90% of probability mass."

**Adjustment guidelines:**
- **Decrease (0.5-0.7):** For more focused and conservative outputs
- **Increase (0.9-1.0):** For more diverse and exploratory text generation

```bash
# Example: More focused token selection
ollama run llama3 --top-p 0.7
```

### NUMA Configuration (`--numa`)

**What it does:** Enables Non-Uniform Memory Access optimization for multi-CPU systems.

**Technical details:** NUMA optimizes memory access patterns on systems with multiple CPU sockets, where memory access time depends on the memory location relative to the processor.

**Adjustment guidelines:**
- **Enable when:** Running on a multi-socket server or high-end workstation
- **Don't use when:** Running on a standard desktop or laptop with a single CPU

```bash
# Example: Enable NUMA on a multi-CPU system
ollama run llama3 --numa
```

### Memory Mapping (`--mmap`)

**What it does:** Controls whether the model weights are memory-mapped from disk or fully loaded into RAM.

**Technical details:** Memory mapping allows the system to load parts of the model from disk as needed, reducing memory usage but potentially impacting performance.

**Adjustment guidelines:**
- **Disable when:** You have sufficient RAM and want maximum performance
- **Enable when:** You want to conserve RAM and are willing to accept some performance impact

```bash
# Example: Disable memory mapping to load model fully into RAM
ollama run llama3 --mmap 0
```

## Parameter Combinations and Tradeoffs

Understanding how these parameters interact is crucial for effective optimization:

1. **Memory vs. Speed:** Larger context sizes and batch sizes improve capabilities and speed but increase memory usage

2. **GPU vs. CPU Balance:** The optimal GPU layer allocation depends on your specific hardware - there's no one-size-fits-all setting

3. **Response Quality vs. Speed:** Temperature and top-p settings affect not just the style of responses but can impact generation speed

## Next Steps

Now that you understand the core parameters, proceed to:

- [Performance Impact of Each Parameter](performance-impact.md) - For detailed benchmarks
- [Hardware-Specific Optimization](../cpu-optimization.md) - For guidance tailored to your system

Remember that the optimal configuration depends on your specific hardware and use case. Experimentation is key to finding the best settings for your situation.
