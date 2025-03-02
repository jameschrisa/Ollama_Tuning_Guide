# Hardware-Specific Optimization

When it comes to running large language models with Ollama, your hardware makes all the difference. Think of your computer as a kitchen - you wouldn't prepare a five-course meal in a tiny apartment kitchen, and similarly, you need the right computing resources for the model you want to run.

This guide will help you match your optimization strategy to your specific hardware, ensuring you get the best possible performance within your constraints.

## Understanding Your Hardware Profile

Before diving into specific optimizations, take a moment to assess what you're working with:

```bash
# Check CPU info
lscpu

# Check available memory
free -h

# Check GPU info (if you have an NVIDIA GPU)
nvidia-smi

# For AMD GPUs
rocm-smi

# For Apple Silicon
system_profiler SPHardwareDataType
```

## Hardware Categories and Strategies

Let's break down optimization strategies by hardware category:

### Entry-Level Hardware
**Profile:** Laptops, older desktops, systems with 8GB RAM or less, integrated graphics

**Strategy:** Run smaller models with conservative parameters
- Focus on 3B-7B parameter models (Phi-2, Mistral 7B, Llama3 8B)
- Prefer CPU-only operation unless you have dedicated GPU memory
- Limit context length to match available memory
- Consider using higher quantization levels (Q4 or Q5)

**Example configuration:**
```bash
# For a laptop with 8GB RAM, integrated graphics
ollama run phi:2.7b --ctx 2048 --gpu 0 --batch 128
```

### Mid-Range Hardware
**Profile:** Gaming PCs, newer laptops, systems with 16GB RAM, GPUs with 6-12GB VRAM

**Strategy:** Balance model size with performance parameters
- 7B-13B parameter models perform well
- Split model layers between GPU and CPU
- Moderate context windows (4K-8K tokens)
- Balance between throughput and memory usage

**Example configuration:**
```bash
# For a desktop with 16GB RAM, RTX 3060 (12GB VRAM)
ollama run llama3:8b --ctx 4096 --gpu 24 --batch 256
```

### High-End Hardware
**Profile:** Workstations, AI-focused builds, systems with 32GB+ RAM, GPUs with 16GB+ VRAM

**Strategy:** Maximize capabilities while maintaining responsiveness
- Can run larger 34B-70B parameter models effectively
- Higher context windows possible (8K-32K tokens)
- Prioritize quality over pure efficiency
- Fine-tune for specific use cases

**Example configuration:**
```bash
# For a workstation with 64GB RAM, RTX 4090 (24GB VRAM)
ollama run llama3:70b --ctx 8192 --gpu 60 --batch 512
```

### Server/Multi-GPU Systems
**Profile:** Dedicated servers, multi-GPU workstations, systems with 128GB+ RAM

**Strategy:** Optimize for maximum throughput and multi-user scenarios
- Run multiple models simultaneously
- Allocate specific GPUs to specific workloads
- Enable advanced features like NUMA
- Optimize for continuous operation

**Example configuration:**
```bash
# For a multi-GPU server
CUDA_VISIBLE_DEVICES=0 ollama run llama3:70b --ctx 16384 --gpu 80 --batch 1024 --numa
```

## Matching Models to Hardware

Not every model runs well on every system. Here's a quick guide to finding the right match:

| Hardware Level | Recommended Models | Avoid |
|----------------|-------------------|-------|
| Entry-Level | Phi-2 (2.7B), Gemma (2B), Tiny Llama (1.1B) | Anything >13B |
| Mid-Range | Llama3 (8B), Mistral (7B), Neural Chat (7B) | Models >34B |
| High-End | Llama3 (70B), Mixtral (8x7B), Stable LM (70B) | None, but watch context length |
| Server | Any model, including ensembles and multiple parallel models | None |

## Hardware-Specific Parameter Guidelines

Different hardware components benefit from different parameter adjustments:

### CPU Optimization

If your system is CPU-limited (or you're running CPU-only):

```bash
# Optimize for CPU performance
ollama run mistral:7b --gpu 0 --threads 6 --batch 64
```

Key parameters for CPU-focused systems:
- `--threads`: Set to (total cores - 2) for dedicated systems, or half your cores for shared systems
- `--batch`: Keep lower than on GPU systems (64-256 range)
- `--mmap`: Consider setting to 0 if you have sufficient RAM for the whole model

Learn more in our detailed [CPU Optimization Guide](cpu-optimization.md).

### GPU Optimization

If you have a capable GPU:

```bash
# Optimize for GPU performance
ollama run llama3:8b --gpu 32 --batch 512
```

Key parameters for GPU-focused systems:
- `--gpu`: Higher values offload more computation to GPU
- `--batch`: GPU can handle larger batches (256-1024 range)
- Context length: Be careful not to exceed VRAM capacity

Learn more in our detailed [GPU Optimization Guide](gpu-optimization.md).

### Memory Management

If your system is memory-constrained:

```bash
# Optimize for limited memory
ollama run phi:2.7b --ctx 2048 --mmap 1
```

Key parameters for memory-constrained systems:
- Model size: Choose smaller models when memory is limited
- `--ctx`: Reduce context window to save memory
- `--mmap`: Enable memory mapping to reduce RAM usage

Learn more in our detailed [RAM Management Strategies](ram-management.md).

## Hardware-Specific Troubleshooting

Each hardware profile comes with its own common issues:

### Entry-Level Hardware Issues

**Issue: Model runs out of memory**
```bash
# Try a smaller model with reduced context
ollama run phi:2.7b --ctx 1024
```

**Issue: Generation is too slow**
```bash
# Reduce batch size and increase threads
ollama run phi:2.7b --batch 64 --threads 4
```

### Mid-Range Hardware Issues

**Issue: GPU memory errors**
```bash
# Reduce GPU layers
ollama run llama3:8b --gpu 16 --ctx 4096
```

**Issue: System becomes unresponsive**
```bash
# Balance CPU thread allocation
ollama run llama3:8b --gpu 24 --threads 4
```

### High-End Hardware Issues

**Issue: Not seeing expected performance gains**
```bash
# Increase batch size to utilize GPU better
ollama run llama3:70b --gpu 60 --batch 1024
```

**Issue: First generation is slow but subsequent ones are fast**
```bash
# Enable keepalive to maintain model in memory
ollama run llama3:70b --gpu 60 --keepalive 300000
```

## Creating Hardware-Optimized Modelfiles

You can bake hardware-specific optimizations directly into custom models:

```
# Example Modelfile for mid-range hardware
FROM llama3:8b

# Hardware optimization for mid-range system
PARAMETER num_ctx 4096
PARAMETER num_batch 256
PARAMETER num_gpu 24
PARAMETER num_thread 6

# Other parameters...
PARAMETER temperature 0.7
```

Save this as `Modelfile` and create your hardware-optimized model:

```bash
ollama create mid-range-llama -f Modelfile
```

## Next Steps

Now that you understand how to match optimization strategies to your hardware, dive deeper into specific optimizations:

- [CPU Optimization Guide](cpu-optimization.md) - For maximizing CPU performance
- [GPU Optimization Guide](gpu-optimization.md) - For leveraging your GPU effectively
- [RAM Management Strategies](ram-management.md) - For balancing memory usage

Remember, hardware optimization is an iterative process. Start with these guidelines, monitor performance, and adjust as needed for your specific system and use cases.
