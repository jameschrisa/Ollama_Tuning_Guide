# Automated Hyperparameter Tuning

Manual parameter sweep testing is valuable, but what if you could automate the process of finding optimal settings for your Ollama models? Automated hyperparameter tuning uses systematic, algorithmic approaches to find the best possible configuration for your specific hardware and use case.

Think of it as having an expert assistant who methodically tests thousands of combinations while you focus on your actual work. This guide will show you how to set up automated tuning systems for Ollama.

## Why Automate Hyperparameter Tuning?

Automated approaches offer several advantages over manual parameter sweeps:

1. **Efficiency** - Test far more combinations than would be practical manually
2. **Optimization algorithms** - Use smarter search strategies than grid search
3. **Multi-objective optimization** - Balance competing goals like speed vs. quality
4. **Reproducibility** - Document the entire optimization process
5. **Continuous improvement** - Easily re-run as your needs or hardware change

## Key Hyperparameter Tuning Algorithms

Several algorithms are particularly well-suited for tuning Ollama parameters:

### Bayesian Optimization

Instead of testing every combination, Bayesian optimization builds a probabilistic model of the parameter space, focusing exploration on promising areas:

- **Pros:** Sample-efficient, works well with expensive evaluations
- **Cons:** More complex to implement
- **Best for:** When each evaluation takes significant time (like large models)

### Tree-structured Parzen Estimator (TPE)

A specialized form of Bayesian optimization that models the conditional probability of parameters given performance:

- **Pros:** Handles mixed discrete/continuous parameters well
- **Cons:** Requires more samples than some methods
- **Best for:** General-purpose parameter tuning with moderate evaluation costs

### Random Search

Randomly samples the parameter space, which often outperforms grid search with the same number of evaluations:

- **Pros:** Simple, surprisingly effective, easily parallelizable
- **Cons:** Inefficient for large parameter spaces
- **Best for:** Initial exploration or simpler tuning tasks

## Setting Up Your Tuning Environment

Before implementing automated tuning, prepare your environment:

1. **Install required libraries**
   ```bash
   pip install optuna ray hyperopt ax-platform
   ```

2. **Create a dedicated benchmark script**
   This will run Ollama with specific parameters and return performance metrics.

3. **Define your objective function**
   What are you optimizing for? Speed, memory efficiency, or a combination?

4. **Set up measurement and logging**
   Ensure consistent, reproducible measurements.

## Implementation with Popular Frameworks

Let's look at how to implement automated tuning with popular frameworks:

### Optuna Implementation

[Optuna](https://optuna.org/) is a hyperparameter optimization framework specifically designed for machine learning. Here's a basic implementation for Ollama:

```python
import optuna
import subprocess
import re
import json
import time

def run_ollama_benchmark(model, ctx, batch, gpu_layers, threads, prompt):
    """Run Ollama with given parameters and return performance metrics"""
    cmd = f"ollama run {model} '{prompt}' -f --ctx {ctx} --batch {batch} --gpu {gpu_layers} --threads {threads}"
    
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    output = result.stdout
    
    # Extract metrics using regex
    tokens_per_sec = float(re.search(r"eval rate: ([\d.]+) tokens/sec", output).group(1))
    first_token_ms = float(re.search(r"first token: ([\d.]+) ms", output).group(1))
    
    return {
        "tokens_per_sec": tokens_per_sec,
        "first_token_ms": first_token_ms
    }

def objective(trial):
    """Optuna objective function for Ollama parameter tuning"""
    # Define the parameter space
    ctx = trial.suggest_categorical("ctx", [2048, 4096, 8192, 16384])
    batch = trial.suggest_categorical("batch", [64, 128, 256, 512, 1024])
    gpu_layers = trial.suggest_int("gpu_layers", 0, 32, 8)  # Assuming 32 total layers
    threads = trial.suggest_int("threads", 1, 12)  # Modify based on your CPU
    
    # Define the benchmark prompt
    prompt = "Explain the theory of relativity in detail, including its historical context and major implications."
    
    # Run benchmark
    metrics = run_ollama_benchmark("llama3:8b", ctx, batch, gpu_layers, threads, prompt)
    
    # Calculate a combined score (higher is better)
    # You can adjust this formula based on your priorities
    score = metrics["tokens_per_sec"] * 1.0 - metrics["first_token_ms"] * 0.01
    
    # Log additional information
    trial.set_user_attr("tokens_per_sec", metrics["tokens_per_sec"])
    trial.set_user_attr("first_token_ms", metrics["first_token_ms"])
    
    return score

# Create a study object and optimize
study = optuna.create_study(direction="maximize", 
                           study_name="ollama_parameter_tuning",
                           storage="sqlite:///ollama_optuna.db")
study.optimize(objective, n_trials=50)

# Print results
print("Best parameters:", study.best_params)
print("Best value:", study.best_value)
print("Best trial user attributes:", study.best_trial.user_attrs)

# Save results
with open("ollama_optuna_results.json", "w") as f:
    json.dump({
        "best_params": study.best_params,
        "best_value": study.best_value,
        "best_metrics": study.best_trial.user_attrs
    }, f, indent=2)
```

This script:
1. Defines a function to run Ollama with specific parameters
2. Creates an objective function that Optuna will optimize
3. Configures the parameter search space
4. Runs 50 trials to find optimal parameters

### Ray Tune Implementation

[Ray Tune](https://docs.ray.io/en/latest/tune/index.html) is excellent for distributed hyperparameter tuning:

```python
import ray
from ray import tune
from ray.tune.search.optuna import OptunaSearch
import subprocess
import re
import os
import json

### Ax Platform Implementation

[Ax](https://ax.dev/) is Facebook's platform for adaptive experimentation, particularly well-suited for multi-objective optimization:

```python
from ax.service.ax_client import AxClient
import subprocess
import re
import json
import time

def run_ollama_benchmark(model, ctx, batch, gpu_layers, threads, prompt):
    """Run Ollama with given parameters and return performance metrics"""
    cmd = f"ollama run {model} '{prompt}' -f --ctx {ctx} --batch {batch} --gpu {gpu_layers} --threads {threads}"
    
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    output = result.stdout
    
    # Extract metrics using regex
    tokens_per_sec = float(re.search(r"eval rate: ([\d.]+) tokens/sec", output).group(1))
    first_token_ms = float(re.search(r"first token: ([\d.]+) ms", output).group(1))
    total_time_ms = float(re.search(r"total: ([\d.]+) ms", output).group(1))
    
    return {
        "tokens_per_sec": tokens_per_sec,
        "first_token_ms": first_token_ms,
        "total_time_ms": total_time_ms
    }

# Initialize Ax client
ax_client = AxClient()

# Define the parameter space
ax_client.create_experiment(
    name="ollama_parameter_tuning",
    parameters=[
        {"name": "ctx", "type": "choice", "values": [2048, 4096, 8192, 16384]},
        {"name": "batch", "type": "choice", "values": [64, 128, 256, 512, 1024]},
        {"name": "gpu_layers", "type": "range", "bounds": [0, 32], "value_type": "int"},
        {"name": "threads", "type": "range", "bounds": [1, 12], "value_type": "int"},
    ],
    objectives={
        "tokens_per_sec": "maximize", 
        "first_token_ms": "minimize"
    },
    objective_thresholds=[
        {"metric": "first_token_ms", "bound": 500, "relative": False, "op": "<="}
    ]
)

# Run trials
model = "llama3:8b"
prompt = "Explain the theory of relativity in detail, including its historical context and major implications."

for i in range(30):
    print(f"Running trial {i+1}/30...")
    parameters, trial_index = ax_client.get_next_trial()
    
    ctx = parameters["ctx"]
    batch = parameters["batch"]
    gpu_layers = parameters["gpu_layers"]
    threads = parameters["threads"]
    
    metrics = run_ollama_benchmark(model, ctx, batch, gpu_layers, threads, prompt)
    
    ax_client.complete_trial(trial_index=trial_index, raw_data=metrics)
    
    time.sleep(2)  # Brief pause between trials

# Get Pareto-optimal parameters
pareto_optimal_parameters = ax_client.get_pareto_optimal_parameters()

print("Pareto-optimal parameters:")
for arm_name, metrics in pareto_optimal_parameters[0].items():
    parameters = ax_client.get_trial_parameters(trial_index=int(arm_name.split("_")[1]))
    print(f"Parameters: {parameters}")
    print(f"Metrics: {metrics}")

# Save results
with open("ollama_ax_results.json", "w") as f:
    json.dump({
        "pareto_optimal": [
            {
                "parameters": ax_client.get_trial_parameters(trial_index=int(arm_name.split("_")[1])),
                "metrics": metrics
            }
            for arm_name, metrics in pareto_optimal_parameters[0].items()
        ]
    }, f, indent=2)
```

This implementation:
1. Defines the parameter space for Ollama
2. Creates a multi-objective experiment (maximize speed while minimizing latency)
3. Runs 30 trials with different parameter combinations
4. Identifies Pareto-optimal configurations (solutions that cannot be improved in one objective without sacrificing another)

## Multi-Objective Optimization

Most real-world tuning involves trade-offs between competing objectives:

1. **Generation speed** vs. **Memory usage**
2. **First token latency** vs. **Total completion time**
3. **Response quality** vs. **Resource utilization**

Instead of a single "best" configuration, multi-objective optimization produces a set of Pareto-optimal solutions representing different trade-offs.

### Defining a Custom Objective Function

For Ollama tuning, you might want to create a weighted objective function that balances different goals:

```python
def combined_score(metrics, weights):
    """Calculate a weighted score from multiple metrics"""
    score = (
        weights["speed"] * metrics["tokens_per_sec"] -
        weights["latency"] * metrics["first_token_ms"] / 100 -
        weights["memory"] * metrics["peak_memory_gb"]
    )
    return score
```

Adjust the weights based on your priorities:
- For chat applications: Higher weight on latency
- For content generation: Higher weight on throughput
- For resource-constrained devices: Higher weight on memory

## Measuring Response Quality

Performance metrics are easy to measure, but what about response quality? Here's how to incorporate quality metrics into your tuning:

1. **Create evaluation prompts** with known "good" responses
2. **Define similarity metrics** (BLEU, ROUGE, semantic similarity)
3. **Implement quality checking** in your tuning loop

```python
def evaluate_response_quality(response, reference):
    """Evaluate the quality of a model response against a reference"""
    from rouge import Rouge
    
    rouge = Rouge()
    scores = rouge.get_scores(response, reference)
    
    # Return ROUGE-L F1 score as quality metric
    return scores[0]["rouge-l"]["f"]
```

## Advanced: Distributed Hyperparameter Tuning

For faster tuning across multiple machines:

```python
# Ray Tune code for distributed tuning
# Add to the Ray Tune example above

ray.init(address="auto")  # Connect to existing Ray cluster

# Configure distributed resources
tune.run(
    run_ollama_benchmark,
    config=search_space,
    metric="score",
    mode="max",
    num_samples=200,  # More samples with distributed resources
    resources_per_trial={"cpu": 4, "gpu": 0.5},
    name="ollama_distributed_tuning"
)
```

This allows you to:
1. Run many more trials in parallel
2. Test on different hardware configurations
3. Share tuning results across a team

## Creating an Automated Tuning Pipeline

For production environments, create a complete tuning pipeline:

1. **Scheduling** - Run tuning jobs periodically
2. **Versioning** - Track parameter configurations over time
3. **Notification** - Alert when better parameters are found
4. **Deployment** - Automatically update model configurations

Here's a simple scheduler using cron:

```bash
#!/bin/bash
# ollama_tune_scheduler.sh

# Directory for results
RESULTS_DIR="$HOME/ollama_tuning_results/$(date +%Y%m%d_%H%M%S)"
mkdir -p $RESULTS_DIR

# Run tuning script
python3 ollama_tuning.py --output $RESULTS_DIR

# Compare with previous best
if [ -f "$HOME/ollama_tuning_results/current_best.json" ]; then
    NEW_SCORE=$(jq '.best_value' $RESULTS_DIR/results.json)
    OLD_SCORE=$(jq '.best_value' $HOME/ollama_tuning_results/current_best.json)
    
    if (( $(echo "$NEW_SCORE > $OLD_SCORE" | bc -l) )); then
        # Better configuration found
        cp $RESULTS_DIR/results.json $HOME/ollama_tuning_results/current_best.json
        
        # Create Modelfile with new parameters
        BEST_CTX=$(jq '.best_params.ctx' $RESULTS_DIR/results.json)
        BEST_BATCH=$(jq '.best_params.batch' $RESULTS_DIR/results.json)
        BEST_GPU=$(jq '.best_params.gpu_layers' $RESULTS_DIR/results.json)
        BEST_THREADS=$(jq '.best_params.threads' $RESULTS_DIR/results.json)
        
        echo "FROM llama3:8b" > $HOME/ollama_tuning_results/Modelfile
        echo "PARAMETER num_ctx $BEST_CTX" >> $HOME/ollama_tuning_results/Modelfile
        echo "PARAMETER num_batch $BEST_BATCH" >> $HOME/ollama_tuning_results/Modelfile
        echo "PARAMETER num_gpu $BEST_GPU" >> $HOME/ollama_tuning_results/Modelfile
        echo "PARAMETER num_thread $BEST_THREADS" >> $HOME/ollama_tuning_results/Modelfile
        
        # Create optimized model
        ollama create optimized-llama -f $HOME/ollama_tuning_results/Modelfile
        
        # Send notification
        echo "Better Ollama configuration found! Updating optimized-llama model." | \
        mail -s "Ollama Tuning Update" your.email@example.com
    fi
fi
```

Add to crontab to run weekly:
```
0 0 * * 0 $HOME/ollama_tune_scheduler.sh
```

## Hardware-Aware Tuning

Different hardware configurations need different parameter settings. Create hardware-specific tuning:

```python
def get_hardware_profile():
    """Detect hardware configuration"""
    import psutil
    import GPUtil
    
    cpu_count = psutil.cpu_count(logical=False)
    total_ram = psutil.virtual_memory().total / (1024**3)  # GB
    
    gpu_info = {}
    try:
        gpus = GPUtil.getGPUs()
        if gpus:
            gpu_info = {
                "name": gpus[0].name,
                "vram": gpus[0].memoryTotal / 1024  # GB
            }
    except:
        pass
    
    return {
        "cpu_count": cpu_count,
        "ram_gb": total_ram,
        "gpu": gpu_info
    }

# Adjust parameter space based on hardware
hardware = get_hardware_profile()

# Example: Adjust GPU layers based on available VRAM
if "gpu" in hardware and hardware["gpu"]:
    vram_gb = hardware["gpu"]["vram"]
    
    if vram_gb < 6:
        max_gpu_layers = 16
    elif vram_gb < 12:
        max_gpu_layers = 32
    else:
        max_gpu_layers = 48
else:
    # No GPU or couldn't detect
    max_gpu_layers = 0

# Update search space
search_space["gpu_layers"] = tune.randint(0, max_gpu_layers + 1, 8)
```

## Evaluating Real-World Tasks

Beyond synthetic benchmarks, evaluate on representative real-world tasks:

1. **Task-specific prompts** - Use prompts that match your actual use cases
2. **Varied complexity** - Include both simple and complex examples
3. **Multiple evaluation runs** - Average across several runs for stability

Example task categories:
- Text summarization
- Code completion
- Conversational response
- Content generation
- Reasoning tasks

## Case Study: Tuning for a Specific Application

Let's walk through a complete tuning example for a specific use case: an AI coding assistant.

**Requirements:**
- Fast first token latency (< 200ms)
- Accurate code completion
- Moderate memory footprint

**Tuning approach:**

1. **Define the objective function:**
```python
def code_assistant_score(metrics, response_quality):
    """Custom scoring for code assistant use case"""
    # Heavily penalize high latency
    if metrics["first_token_ms"] > 200:
        latency_penalty = 5.0
    else:
        latency_penalty = metrics["first_token_ms"] / 200.0
    
    # Balanced score formula
    score = (
        3.0 * response_quality +  # Quality is most important
        1.0 * metrics["tokens_per_sec"] / 100.0 -  # Speed matters
        2.0 * latency_penalty -  # Latency is critical
        1.0 * metrics["peak_memory_gb"] / 2.0  # Memory is moderately important
    )
    
    return score
```

2. **Create code-specific evaluation prompts:**
```python
code_evaluation_prompts = [
    "Write a Python function to find the nth Fibonacci number using dynamic programming.",
    "Create a React component that displays a sortable table with pagination.",
    "Implement a binary search tree in C++ with insert, delete and search methods."
]
```

3. **Run the tuning process:**
```python
# Run tuning with code-specific objective and evaluation
best_params = run_tuning(
    evaluation_prompts=code_evaluation_prompts,
    scoring_function=code_assistant_score,
    n_trials=100,
    model="codellama:13b"
)
```

4. **Create a custom Modelfile:**
```
FROM codellama:13b

PARAMETER num_ctx 8192
PARAMETER num_batch 256
PARAMETER num_gpu 28
PARAMETER num_thread 6
PARAMETER temperature 0.2
PARAMETER top_p 0.85
PARAMETER repeat_penalty 1.1

# Custom system prompt for code assistant
SYSTEM """
You are an expert programming assistant. When writing code:
1. Prioritize correctness and readability
2. Follow language-specific best practices
3. Include helpful comments explaining complex logic
4. Consider edge cases and error handling
"""
```

The result is a finely-tuned model specifically optimized for code completion tasks on your hardware.

## Tuning Beyond Standard Parameters

In addition to the core Ollama parameters, consider tuning:

1. **Generation parameters:**
   - `--temperature` (0.1 to 1.0)
   - `--top-p` (0.1 to 1.0)
   - `--repeat-penalty` (1.0 to 1.5)

2. **System prompt engineering:**
   - Length and detail
   - Instruction clarity
   - Examples vs. rules

3. **Memory management:**
   - `--mmap` (0 or 1)
   - `--numa` (for multi-socket systems)

## Next Steps

After implementing automated hyperparameter tuning:

- [Custom Modelfiles with Optimized Parameters](../custom-modelfiles.md) to save your discovered configurations
- [Hardware-Specific Optimization](../hardware-specific-optimization.md) to adapt to different environments
- [Parameter Sweep Testing](../parameter-sweep.md) for more targeted manual testing

Remember that hyperparameter tuning is an ongoing process. As models evolve and your hardware changes, periodically re-run your tuning pipeline to maintain optimal performance.


def run_ollama_benchmark(config):
    """Run Ollama with given parameters and return performance metrics"""
    model = config["model"]
    ctx = config["ctx"]
    batch = config["batch"]
    gpu_layers = config["gpu_layers"]
    threads = config["threads"]
    prompt = config["prompt"]
    
    cmd = f"ollama run {model} '{prompt}' -f --ctx {ctx} --batch {batch} --gpu {gpu_layers} --threads {threads}"
    
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    output = result.stdout
    
    # Extract metrics using regex
    tokens_per_sec = float(re.search(r"eval rate: ([\d.]+) tokens/sec", output).group(1))
    first_token_ms = float(re.search(r"first token: ([\d.]+) ms", output).group(1))
    total_time_ms = float(re.search(r"total: ([\d.]+) ms", output).group(1))
    
    # Calculate combined score
    score = tokens_per_sec * 1.0 - first_token_ms * 0.01
    
    tune.report(
        score=score,
        tokens_per_sec=tokens_per_sec,
        first_token_ms=first_token_ms,
        total_time_ms=total_time_ms
    )

# Initialize Ray
ray.init()

# Define the search space
search_space = {
    "model": "llama3:8b",
    "ctx": tune.choice([2048, 4096, 8192, 16384]),
    "batch": tune.choice([64, 128, 256, 512, 1024]),
    "gpu_layers": tune.randint(0, 33, 8),  # 0 to 32 in steps of 8
    "threads": tune.randint(1, 13),  # 1 to 12
    "prompt": "Explain the theory of relativity in detail, including its historical context and major implications."
}

# Configure the search algorithm
algo = OptunaSearch()

# Run the hyperparameter search
analysis = tune.run(
    run_ollama_benchmark,
    config=search_space,
    metric="score",
    mode="max",
    num_samples=50,
    search_alg=algo,
    resources_per_trial={"cpu": 4, "gpu": 0.5},  # Adjust based on your hardware
    name="ollama_ray_tune"
)

# Get best configuration
best_config = analysis.get_best_config(metric="score", mode="max")
best_result = analysis.best_result

print("Best parameters:", best_config)
print("Best metrics:", best_result)

# Save results
with open("ollama_ray_tune_results.json", "w") as f:
    json.dump({
        "best_config": best_config,
        "best_metrics": {
            "score": best_result["score"],
            "tokens_per_sec": best_result["tokens_per_sec"],
            "first_token_ms": best_result["first_token_ms"],
            "total_time_ms": best_result["total_time_ms"]
        }
    }, f, indent=2)
