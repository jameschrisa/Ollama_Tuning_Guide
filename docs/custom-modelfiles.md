# Custom Modelfiles with Optimized Parameters

One of the most powerful features of Ollama is the ability to create custom model configurations using Modelfiles. Think of a Modelfile as a "recipe" for your model – it specifies not just the base model, but also hyperparameters, system prompts, and other settings that determine how the model operates.

This guide will teach you how to create Modelfiles with optimized parameters for your specific use cases.

## Modelfile Basics

A Modelfile is a simple text file that uses a Dockerfile-like syntax. Here's a basic example:

```
FROM llama3:8b
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
SYSTEM "You are a helpful AI assistant."
```

Let's break down the components:

- `FROM` - Specifies the base model
- `PARAMETER` - Sets runtime parameters
- `SYSTEM` - Defines the system prompt (instructions to the model)

## Creating and Using a Modelfile

1. Create a text file named `Modelfile` (no extension)
2. Add your configuration
3. Create your custom model using:
   ```bash
   ollama create mymodel -f Modelfile
   ```
4. Run your model:
   ```bash
   ollama run mymodel
   ```

## Optimizing Parameters in Modelfiles

### Essential Parameters to Consider

```
FROM llama3:8b

# Performance parameters
PARAMETER num_ctx 8192        # Context window size
PARAMETER num_batch 512       # Batch size for processing
PARAMETER num_gpu 32          # Number of layers to run on GPU
PARAMETER num_thread 8        # CPU threads to use
PARAMETER mirostat 1          # Use Mirostat sampling (0, 1, or 2)
PARAMETER mirostat_eta 0.1    # Learning rate for Mirostat
PARAMETER mirostat_tau 5.0    # Target entropy for Mirostat

# Generation parameters
PARAMETER temperature 0.7     # Sampling temperature
PARAMETER top_k 40            # Top-k sampling
PARAMETER top_p 0.9           # Nucleus sampling
PARAMETER repeat_penalty 1.1  # Penalty for repeating tokens
```

### Example: Optimized for Code Generation

```
FROM codellama:13b

# Performance tuning
PARAMETER num_ctx 16384       # Larger context for code files
PARAMETER num_batch 256
PARAMETER num_gpu 40
PARAMETER num_thread 8

# Generation parameters for code
PARAMETER temperature 0.3     # Lower temperature for more precise code
PARAMETER top_p 0.85
PARAMETER repeat_penalty 1.2  # Higher repeat penalty to avoid duplicated code
PARAMETER stop "</s>"         # Stop token
PARAMETER stop "```"          # Stop at code block end

# System prompt for code generation
SYSTEM """
You are an expert programmer with deep knowledge of software engineering best practices.
When providing code:
1. Write clean, well-documented code
2. Include comments explaining complex logic
3. Follow language-specific best practices
4. Consider edge cases and error handling
"""
```

### Example: Optimized for Creative Writing

```
FROM llama3:70b

# Performance tuning for lengthy creative content
PARAMETER num_ctx 32768       # Large context for creative writing
PARAMETER num_batch 512
PARAMETER num_gpu 60

# Generation parameters for creative content
PARAMETER temperature 0.9     # Higher temperature for creativity
PARAMETER top_p 0.95           # Allow more diverse token selection
PARAMETER repeat_penalty 1.05  # Lower repeat penalty for flowing text

# System prompt for creative writing
SYSTEM """
You are a creative writing assistant with a flair for vivid description and engaging narrative.
When writing:
1. Use sensory details to create immersive scenes
2. Develop distinctive character voices
3. Balance dialogue with description
4. Maintain consistent tone and style throughout the piece
"""
```

## Advanced Modelfile Techniques

### Embedding Files Within Modelfiles

You can include external files directly in your Modelfile using the `TEMPLATE` directive:

```
FROM llama3:8b

# Include a custom prompt template
TEMPLATE """
<system>
You are a helpful AI assistant.
</system>

<user>
{{.Input}}
</user>

<assistant>
"""

# Performance parameters
PARAMETER temperature 0.7
```

### Multi-Modal Model Configuration

For models that support image inputs:

```
FROM llava:13b

# Performance tuning for multimodal
PARAMETER num_ctx 4096
PARAMETER num_gpu 32

# Use a specific prompt format for image inputs
TEMPLATE """
<image>
{{.Input}}
</image>

Please describe this image in detail.
"""
```

### Specialized Academic Research Model

```
FROM llama3:70b

# Performance optimization for deep research
PARAMETER num_ctx 16384
PARAMETER num_gpu 60
PARAMETER num_batch 512

# Fine-tune generation for academic content
PARAMETER temperature 0.4
PARAMETER top_p 0.85
PARAMETER repeat_penalty 1.15

# Comprehensive system prompt for academic assistance
SYSTEM """
You are a research assistant with expertise across multiple academic disciplines.
When responding:
1. Prioritize scientific accuracy and nuance
2. Cite relevant research where appropriate
3. Acknowledge limitations and uncertainties
4. Distinguish between established facts and emerging hypotheses
5. Structure responses with clear organization (definitions, explanations, examples, implications)
"""
```

## Hardware-Specific Modelfiles

### Low-End Hardware Optimization

```
FROM phi:2.7b

# Conservative resource usage
PARAMETER num_ctx 2048        # Smaller context to save memory
PARAMETER num_batch 128       # Modest batch size
PARAMETER num_gpu 0           # CPU only mode
PARAMETER num_thread 4        # Limited thread usage

# Compensate with generation parameters
PARAMETER temperature 0.6
PARAMETER top_p 0.8
```

### High-End Workstation Optimization

```
FROM llama3:70b

# Maximize resource utilization
PARAMETER num_ctx 32768
PARAMETER num_batch 1024
PARAMETER num_gpu 80
PARAMETER num_thread 16
PARAMETER f16 true            # Use half-precision
PARAMETER numa true           # Enable NUMA for multi-socket systems
```

## Prompt Engineering in Modelfiles

The `SYSTEM` directive is where most of your prompt engineering happens. This sets the initial context and behavior for the model:

```
FROM mistral:7b

PARAMETER temperature 0.7
PARAMETER num_ctx 4096

SYSTEM """
You are an AI assistant specialized in cybersecurity.

Your expertise includes:
- Network security and protocols
- Malware analysis and prevention
- Security best practices
- Threat modeling and risk assessment

When providing security advice:
1. Always prioritize security best practices
2. Consider both technical and human factors
3. Avoid providing information that could enable harmful activities
4. Recommend defense-in-depth approaches
"""
```

## Testing and Iterating on Modelfiles

Creating the perfect Modelfile often requires experimentation. Here's a systematic approach:

1. **Start with a baseline**:
   ```bash
   ollama create baseline -f Baseline.Modelfile
   ```

2. **Create variants with different parameters**:
   ```bash
   ollama create variant1 -f Variant1.Modelfile
   ollama create variant2 -f Variant2.Modelfile
   ```

3. **Run comparative tests**:
   ```bash
   # Test with identical prompts
   echo "Your test prompt here" | ollama run baseline > baseline_result.txt
   echo "Your test prompt here" | ollama run variant1 > variant1_result.txt
   ```

4. **Measure performance metrics**:
   ```bash
   # Time the executions
   time ollama run baseline "Your benchmark prompt" -f
   time ollama run variant1 "Your benchmark prompt" -f
   ```

## Sharing and Deploying Custom Models

Once you've created an optimized Modelfile, you can share it with others:

1. **Export your model**:
   ```bash
   ollama export mymodel > mymodel.tar
   ```

2. **Share the Modelfile for others to build from**:
   ```bash
   # Extract just the Modelfile
   tar -xf mymodel.tar Modelfile
   ```

3. **Import on another system**:
   ```bash
   ollama import mymodel.tar
   ```

## Real-World Modelfile Examples

### Production API Service Model

```
FROM llama3:70b

# High-throughput configuration
PARAMETER num_ctx 8192
PARAMETER num_batch 1024
PARAMETER num_gpu 60

# Consistent, deterministic outputs
PARAMETER temperature 0.3
PARAMETER top_p 0.8
PARAMETER seed 42            # Fixed seed for reproducibility

# Tight resource management
PARAMETER repeat_last_n 256  # Look back window for repeat penalty
PARAMETER repeat_penalty 1.2
PARAMETER num_predict 512    # Limit maximum tokens generated

SYSTEM """
You are an AI assistant operating in an API context. Your responses will be processed by software.
Always respond in the following JSON format:
{
  "response": {
    "answer": "Your direct answer to the question",
    "confidence": 0-1 value representing confidence in the answer,
    "sources": [] // If applicable, list sources
  }
}
"""
```

### Specialized Knowledge Graph Assistant

```
FROM llama3:70b

# Knowledge-focused configuration
PARAMETER num_ctx 32768      # Large context for knowledge bases
PARAMETER num_gpu 70
PARAMETER temperature 0.4

# Custom response formatting
TEMPLATE """
<knowledge_query>
{{.Input}}
</knowledge_query>

Please analyze this query and provide:
1. Direct answer
2. Related concepts
3. Hierarchical categorization
"""
```

## Troubleshooting Common Modelfile Issues

### Issue: Model Runs Out of Memory

**Solution:** Adjust context and GPU parameters:
```
PARAMETER num_ctx 4096       # Reduce from larger value
PARAMETER num_gpu 24         # Reduce number of GPU layers
```

### Issue: Responses Too Random/Inconsistent

**Solution:** Tune generation parameters:
```
PARAMETER temperature 0.4    # Lower temperature
PARAMETER top_p 0.8          # More conservative top_p
PARAMETER seed 42            # Set fixed seed
```

### Issue: Model Ignoring System Prompt

**Solution:** Ensure proper formatting and strengthen the directive:
```
SYSTEM """
You MUST act as a [specific role].
Always follow these guidelines:
- Guideline 1
- Guideline 2
"""
```

## Conclusion

Custom Modelfiles are one of the most powerful features of Ollama, allowing you to precisely tailor models to your specific requirements and hardware capabilities. By thoughtfully combining base model selection, performance parameters, and prompt engineering, you can create models that significantly outperform default configurations.

Remember that the perfect Modelfile is often discovered through experimentation and iteration. Keep testing different configurations, measuring performance, and refining your approach based on real-world usage.

## Next Steps

- Learn about [Automated Hyperparameter Tuning](../automated-tuning.md) to systematically discover optimal configurations
- Explore [Parameter Sweep Testing](../parameter-sweep.md) for scientific comparison of parameters
- Check out [Troubleshooting](../troubleshooting.md) for more specific issue resolution
