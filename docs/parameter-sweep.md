# Parameter Sweep Testing

Optimizing Ollama models isn't just about following general guidelines—it's about finding the specific configuration that works best for your unique combination of hardware, models, and use cases. This is where parameter sweep testing comes in.

Think of parameter sweep testing as systematically experimenting with different settings to find your model's "sweet spot." It's like fine-tuning an instrument—methodical adjustments until you achieve optimal performance.

## Why Conduct Parameter Sweeps?

Even small changes in hyperparameters can lead to significant differences in:

- Generation speed (tokens per second)
- Memory usage
- Response quality
- CPU/GPU utilization
- First token latency

Rather than guessing what might work best, parameter sweeps give you empirical evidence to base your decisions on.

## Setting Up Your Testing Environment

Before starting parameter sweeps, create a controlled testing environment:

1. **Minimize background processes**
   - Close unnecessary applications
   - Disable auto-updates or scheduled tasks
   - Monitor system resource usage with tools like `htop` or Task Manager

2. **Standardize test inputs**
   - Create benchmark prompts for consistent testing
   - Use prompts relevant to your typical usage
   - Include both short and long prompts

3. **Prepare measurement tools**
   - Use Ollama's built-in benchmarking (`-f` flag)
   - Set up scripts to record and analyze results
   - Consider tracking temperature to account for thermal throttling

## Core Parameters to Test

While you could test every possible parameter, focus on these high-impact settings first:

### 1. Context Length (`--ctx`)

Context length dramatically affects memory usage and capabilities:

```bash
# Test script example
for ctx in 2048 4096 8192 16384; do
  echo "Testing context length: $ctx"
  ollama run llama3:8b "Write a detailed analysis of renewable energy trends" -f --ctx $ctx | tee -a ctx_results.txt
done
```

**Sweep range suggestion:**
- Start: 2048
- End: Your maximum memory-supported context
- Increments: Double each step (2048, 4096, 8192, etc.)

### 2. Batch Size (`--batch`)

Batch size affects throughput and memory consumption:

```bash
# Test script example
for batch in 64 128 256 512 1024; do
  echo "Testing batch size: $batch"
  ollama run llama3:8b "Explain quantum computing in detail" -f --batch $batch | tee -a batch_results.txt
done
```

**Sweep range suggestion:**
- Start: 64
- End: 1024
- Increments: Double each step (64, 128, 256, etc.)

### 3. GPU Layers (`--gpu`)

Find the optimal CPU/GPU balance:

```bash
# Test script example
for gpu in 8 16 24 32; do
  echo "Testing GPU layers: $gpu"
  ollama run llama3:8b "Describe the history of artificial intelligence" -f --gpu $gpu | tee -a gpu_results.txt
done
```

**Sweep range suggestion:**
- Start: 0 (CPU only)
- End: Total model layers
- Increments: 8 or 16 layers at a time

### 4. Thread Count (`--threads`)

Optimize CPU thread allocation:

```bash
# Test script example
for threads in 2 4 6 8 10 12; do
  echo "Testing thread count: $threads"
  ollama run llama3:8b "Explain the process of photosynthesis" -f --threads $threads | tee -a thread_results.txt
done
```

**Sweep range suggestion:**
- Start: 1
- End: Your CPU core count
- Increments: 2 threads at a time

## Creating a Comprehensive Sweep Test Script

Here's a more sophisticated bash script for conducting parameter sweeps:

```bash
#!/bin/bash

# Define the model to test
MODEL="llama3:8b"

# Define the test prompt
PROMPT="Write a detailed analysis of climate change impacts on agriculture, including potential mitigation strategies and economic considerations."

# Define parameter ranges to test
CTX_VALUES=(2048 4096 8192)
BATCH_VALUES=(128 256 512)
GPU_VALUES=(0 16 32)
THREAD_VALUES=(4 8)

# Create results directory
RESULTS_DIR="ollama_sweep_results_$(date +%Y%m%d_%H%M%S)"
mkdir -p $RESULTS_DIR

# Log system info
echo "System information:" > $RESULTS_DIR/system_info.txt
uname -a >> $RESULTS_DIR/system_info.txt
if command -v nvidia-smi &> /dev/null; then
    nvidia-smi >> $RESULTS_DIR/system_info.txt
fi
echo "CPU info:" >> $RESULTS_DIR/system_info.txt
lscpu >> $RESULTS_DIR/system_info.txt
echo "Memory info:" >> $RESULTS_DIR/system_info.txt
free -h >> $RESULTS_DIR/system_info.txt

# CSV header for results
echo "ctx,batch,gpu,threads,first_token_ms,tokens_per_second,total_tokens,total_time_ms" > $RESULTS_DIR/results.csv

# Run tests
for ctx in "${CTX_VALUES[@]}"; do
  for batch in "${BATCH_VALUES[@]}"; do
    for gpu in "${GPU_VALUES[@]}"; do
      for threads in "${THREAD_VALUES[@]}"; do
        echo "Testing: ctx=$ctx, batch=$batch, gpu=$gpu, threads=$threads"
        
        # Run the test and capture output
        OUTPUT_FILE="$RESULTS_DIR/output_ctx${ctx}_batch${batch}_gpu${gpu}_threads${threads}.txt"
        ollama run $MODEL "$PROMPT" -f --ctx $ctx --batch $batch --gpu $gpu --threads $threads > $OUTPUT_FILE
        
        # Extract metrics
        FIRST_TOKEN=$(grep "first token:" $OUTPUT_FILE | awk '{print $3}')
        TOKENS_PER_SEC=$(grep "eval rate:" $OUTPUT_FILE | awk '{print $3}')
        TOTAL_TOKENS=$(grep "tokens:" $OUTPUT_FILE | awk '{print $2}')
        TOTAL_TIME=$(grep "total:" $OUTPUT_FILE | awk '{print $2}')
        
        # Append to CSV
        echo "$ctx,$batch,$gpu,$threads,$FIRST_TOKEN,$TOKENS_PER_SEC,$TOTAL_TOKENS,$TOTAL_TIME" >> $RESULTS_DIR/results.csv
        
        # Small pause between tests to let system stabilize
        sleep 5
      done
    done
  done
done

echo "Parameter sweep complete. Results saved to $RESULTS_DIR/results.csv"
```

Save this as `ollama_sweep.sh`, make it executable (`chmod +x ollama_sweep.sh`), and run it.

## Analyzing Sweep Results

After collecting data, analyze it to find optimal parameters:

1. **Graph the results**
   - Plot tokens per second vs. different parameters
   - Look for inflection points where performance levels off

2. **Consider multiple metrics**
   - First token latency
   - Tokens per second
   - Memory usage
   - Response quality

3. **Find the Pareto frontier**
   - Identify configurations that offer the best tradeoff between metrics
   - Some parameters may have diminishing returns past certain points

## Example Analysis: Finding the Sweet Spot

Here's an example of analyzing batch size results:

| Batch Size | Tokens/Sec | First Token (ms) | Memory (GB) |
|------------|------------|------------------|-------------|
| 64         | 28.5       | 85               | 5.2         |
| 128        | 42.3       | 92               | 5.4         |
| 256        | 58.7       | 110              | 5.9         |
| 512        | 62.1       | 135              | 6.8         |
| 1024       | 64.5       | 210              | 8.3         |

**Analysis:**
- Performance increases significantly from 64→256
- Minimal gains from 512→1024
- First token latency increases dramatically at larger batch sizes
- Memory usage grows substantially at 1024

**Conclusion:** Batch size 256 or 512 offers the best balance for this particular setup.

## Visualizing Parameter Sweep Results

Data visualization helps identify patterns and optimal settings. Here's how to interpret common visualization patterns:

### Batch Size vs. Performance

When plotting batch size against tokens per second, you'll often see a curve that rises quickly at first, then plateaus:

```
Tokens/sec
    ^
    |                  *-------*-------*
    |             *----
    |        *----
    |    *---
    |---*
    +---------------------------------->
       64   128   256   512  1024  2048   Batch Size
```

The optimal batch size is typically near where the curve begins to flatten, as larger batches continue to consume more memory with diminishing performance returns.

### GPU Layers vs. Performance

For GPU layer allocation, the performance curve often looks like:

```
Tokens/sec
    ^
    |             *-------*
    |         *---         *
    |     *---              *
    |  *--                   *
    |-*                       *---
    +---------------------------------->
       8    16    24    32    40    48   GPU Layers
```

Too few layers on GPU underutilizes it, while too many can cause VRAM bottlenecks, creating a peak in the middle that represents your optimal setting.

### Context Length vs. Memory Usage

Context length has a nearly linear relationship with memory usage:

```
Memory (GB)
    ^
    |                             *
    |                      *
    |               *
    |        *
    |   *
    +---------------------------------->
      2048  4096  8192 16384 32768    Context Length
```

Choose the largest context your system can handle without excessive swapping or performance degradation.

## Advanced: Multi-Parameter Optimization

Parameters often interact with each other. After testing individual parameters, try combinations of your best-performing values:

```bash
# Example of testing parameter combinations
ollama run llama3:8b "Explain the theory of relativity" -f --ctx 8192 --batch 256 --gpu 32 --threads 8
```

Make a grid of your top 2-3 values for each parameter to find optimal combinations.

## Use Case-Specific Parameter Sweeps

Different tasks may benefit from different optimizations. Consider creating task-specific sweep tests:

### Chat Performance Test

```bash
# Chat-specific prompt
PROMPT="Hello, can you tell me about the benefits of regular exercise? After that, I'd like to know about nutrition as well."
```

### Code Generation Test

```bash
# Code-specific prompt
PROMPT="Write a Python function that implements the merge sort algorithm. Include comments explaining the algorithm and analyze its time complexity."
```

### Long-Form Content Test

```bash
# Long-form content prompt
PROMPT="Write a comprehensive essay on the history and impact of the internet, covering its technical origins, key milestones, and how it transformed society."
```

## Creating Parameter Profiles

After identifying optimal settings for different use cases, create profiles for easy switching:

```
# Chat Profile Modelfile
FROM llama3:8b
PARAMETER num_ctx 4096
PARAMETER num_batch 256
PARAMETER num_gpu 24
PARAMETER num_thread 6
```

Save as `chat_profile.Modelfile` and create your profile:

```bash
ollama create chat-optimized -f chat_profile.Modelfile
```

## Real-World Parameter Sweep Scenarios

### Scenario 1: Budget Gaming PC

**Hardware:**
- CPU: 6-core i5
- RAM: 16GB
- GPU: RTX 3060 6GB

**Sweep focus:**
- GPU layers (8-32)
- Context length (2048-8192)
- Thread count (2-6)

**Result:** Optimal configuration
```bash
ollama run llama3:8b --gpu 24 --ctx 4096 --threads 4 --batch 256
```

### Scenario 2: AI Workstation

**Hardware:**
- CPU: 16-core Ryzen 9
- RAM: 64GB
- GPU: RTX 4090 24GB

**Sweep focus:**
- Model size (7B vs 13B vs 70B)
- Batch size (256-1024)
- Context length (8192-32768)

**Result:** Optimal configuration
```bash
ollama run llama3:70b --gpu 60 --ctx 16384 --threads 12 --batch 512
```

## Reproducible Sweep Testing

For scientific rigor, ensure your sweeps are reproducible:

1. **Control for thermal conditions**
   - Allow hardware to reach steady temperatures between tests
   - Monitor for thermal throttling

2. **Run multiple trials**
   - Average across 3-5 runs of each parameter setting
   - Calculate standard deviation to measure consistency

3. **Document everything**
   - System specifications
   - Ambient conditions
   - Background processes
   - Exact prompt text used

## Automated Analysis of Sweep Results

Process your CSV results file with tools like Python's pandas and matplotlib:

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Load results
results = pd.read_csv('ollama_sweep_results/results.csv')

# Group by batch size and calculate mean performance
batch_perf = results.groupby('batch')['tokens_per_second'].mean().reset_index()

# Plot
plt.figure(figsize=(10, 6))
sns.lineplot(x='batch', y='tokens_per_second', data=batch_perf, marker='o')
plt.title('Effect of Batch Size on Generation Speed')
plt.xlabel('Batch Size')
plt.ylabel('Tokens per Second')
plt.grid(True)
plt.savefig('batch_performance.png')
plt.show()
```

## Sharing and Collaborating on Parameter Sweeps

Consider contributing your sweep results to the community:

1. **Document your methodology**
   - Exact commands used
   - Hardware specifications
   - Prompts tested

2. **Share your scripts**
   - Upload sweep scripts to GitHub
   - Include analysis notebooks

3. **Create comparison tables**
   - Make your results easy to reference for specific hardware/model combinations

## Next Steps

After conducting parameter sweeps:

- [Automated Hyperparameter Tuning](automated-tuning.md) for more sophisticated optimization
- [Custom Modelfiles](../custom-modelfiles.md) to save your optimal configurations
- [Troubleshooting](../troubleshooting.md) if you encounter unexpected results

Remember that parameter optimization is an ongoing process. As models evolve and your hardware changes, periodically re-run sweeps to ensure you're maintaining optimal performance.
