# GPU Optimization Guide for Ollama Models

One of the biggest advantages of Ollama is the ability to leverage your GPU to dramatically speed up inference. This guide will help you extract maximum performance from your GPU while running quantized language models.

## Understanding GPU Acceleration in Ollama

When Ollama runs a model with GPU acceleration, it distributes the model's neural network layers between your GPU and CPU. The `--gpu` parameter controls exactly how many layers are offloaded to the GPU.

Think of your model as a multi-story building - you're deciding how many floors to place on your GPU's "land" versus your CPU's "land."

## Determining Your GPU's Capabilities

Before optimizing, you should understand your GPU's capabilities:

### Check Your GPU Model and VRAM

```bash
# For NVIDIA GPUs
nvidia-smi

# For AMD GPUs
rocm-smi

# For Apple Silicon
system_profiler SPDisplaysDataType
```

### Estimated VRAM Requirements by Model Size

| Model Size | Q4 Quantized | Q5 Quantized | Q8 Quantized | FP16 |
|------------|--------------|--------------|--------------|------|
| 7B         | ~4GB         | ~5GB         | ~7GB         | ~14GB|
| 13B        | ~8GB         | ~9GB         | ~13GB        | ~26GB|
| 34B        | ~20GB        | ~23GB        | ~34GB        | ~68GB|
| 70B        | ~40GB        | ~47GB        | ~70GB        | ~140GB|

> **Note:** These are approximate values. Actual memory usage will also depend on context length and batch size.

## Optimal GPU Layer Allocation

The goal is to find the sweet spot where you're using as much of your GPU as possible without running out of VRAM.

### Step-by-Step GPU Optimization

1. **Start with a conservative approach:**
   ```bash
   # For a 7B model on a 6GB VRAM GPU, try half the layers:
   ollama run llama3:8b --gpu 16
   ```

2. **Gradually increase GPU layers:**
   Increase the `--gpu` value until you either reach the model's total layer count or encounter "Out of Memory" errors.

3. **Find your ceiling and step back:**
   If you encounter an OOM error at `--gpu 28`, try `--gpu 26` to give yourself some headroom.

### Example GPU Layer Allocations by Card

| GPU Model        | VRAM  | 7B Model | 13B Model | 70B Model |
|------------------|-------|----------|-----------|-----------|
| RTX 3060         | 12GB  | 32 (all) | 25-28     | 8-10      |
| RTX 4070         | 12GB  | 32 (all) | 25-30     | 8-12      |
| RTX 3080         | 10GB  | 32 (all) | 20-24     | 6-8       |
| RTX 4090         | 24GB  | 32 (all) | 40 (all)  | 24-28     |
| A100             | 40GB  | 32 (all) | 40 (all)  | 45-55     |
| Apple M1 Pro/Max | 16GB* | 32 (all) | 32-35     | Not recommended |
| Apple M2 Ultra   | 24GB* | 32 (all) | 40 (all)  | 20-24     |

> **Note:** Apple Silicon shares memory between system and GPU, so reserve at least 8GB for the system.

## Optimizing Context Length with GPU

When using GPU acceleration, context length becomes even more important as it directly impacts VRAM usage:

```bash
# For a 6GB GPU with a 7B model, try:
ollama run llama3:8b --gpu 16 --ctx 4096

# For a 24GB GPU with a 70B model, you might be able to use:
ollama run llama3:70b --gpu 24 --ctx 8192
```

## Using Multiple GPUs

If you have multiple GPUs, Ollama can leverage them for running different models simultaneously, but not for splitting a single model as of the latest version:

```bash
# Run one model on GPU 0
CUDA_VISIBLE_DEVICES=0 ollama run llama3:8b --gpu 32

# Run another model on GPU 1
CUDA_VISIBLE_DEVICES=1 ollama run mistral:7b --gpu 32
```

## Special Considerations by GPU Type

### NVIDIA GPUs

For NVIDIA GPUs, ensure you have the latest CUDA drivers installed:

```bash
# Check CUDA version
nvcc --version
```

Optimal performance typically requires CUDA 11.8 or newer.

### AMD GPUs

AMD support in Ollama uses ROCm:

```bash
# Check ROCm version
rocm-smi --showversion
```

Ensure you're using ROCm 5.4 or newer for best compatibility.

### Apple Silicon

Apple Silicon benefits from specific optimizations:

```bash
# For M1/M2 chips, metal acceleration is used automatically
ollama run llama3:8b
```

The Metal Performance Shaders (MPS) backend is used automatically, but you may still want to control layer allocation with `--gpu`.

## Advanced: Batch Size Optimization for GPUs

GPUs excel at parallel processing, so optimizing batch size can significantly improve throughput:

```bash
# Try larger batch sizes on capable GPUs
ollama run llama3:8b --gpu 32 --batch 512
```

For high-end GPUs (RTX 4090, A100, etc.), try batch sizes between 512-1024 for optimal throughput.

## Monitoring GPU Performance

While running Ollama, monitor your GPU utilization to ensure you're effectively using the hardware:

```bash
# For NVIDIA
watch -n 1 nvidia-smi

# For AMD
watch -n 1 rocm-smi
```

Look for:
- **GPU Utilization:** Should be >80% during generation for optimal use
- **VRAM Usage:** Should be high but not causing throttling
- **Temperature:** Keep below 85°C for NVIDIA or 95°C for AMD cards

## Common GPU Issues and Solutions

### Issue: Out of Memory Errors

**Solution:** Reduce the number of GPU layers, context size, or batch size:
```bash
# Try reducing GPU layers
ollama run llama3:70b --gpu 20 --ctx 4096 --batch 128
```

### Issue: Low GPU Utilization

**Solution:** Increase batch size or GPU layers:
```bash
ollama run llama3:8b --gpu 32 --batch 512
```

### Issue: Slow First Generation

**Solution:** Use the keepalive option to keep the model in memory:
```bash
ollama run llama3 --keepalive 30000  # Keep model loaded for 30 seconds
```

## Next Steps

Once you've optimized your GPU settings, consider:

- [RAM Management Strategies](ram-management.md) for balancing system resources
- [Use Case Optimization](../chat-optimization.md) for further tuning based on your specific needs
- [Custom Modelfiles](../custom-modelfiles.md) to bake your optimized parameters into a custom model

Remember that GPU optimization is as much art as science - the "perfect" settings depend on your specific hardware, model, and use case. Don't be afraid to experiment!
