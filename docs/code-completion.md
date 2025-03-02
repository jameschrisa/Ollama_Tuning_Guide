# Optimizing Ollama Models for Code Completion

Code completion has unique demands compared to other LLM tasks. When you're using Ollama models to help with programming, you need precise, contextually aware, and syntactically correct suggestions—not just plausible-looking code.

In this guide, I'll walk you through optimizing your Ollama setup specifically for coding tasks. Think of this like setting up your IDE—with the right configurations, you'll dramatically improve your development workflow.

## Code Completion Requirements

When working with code, you're generally looking for:

1. **Precision and correctness** - Syntax errors or logical bugs are costly
2. **Contextual awareness** - Understanding your existing codebase
3. **Fast response time** - Coding is interactive and iterative
4. **Relevant documentation** - Explaining the code is as important as writing it
5. **Flexible language support** - Help with various programming languages

## Recommended Models for Code Completion

Some models are specifically trained or fine-tuned for code:

| Model | Size | Strengths | Best For |
|-------|------|-----------|----------|
| CodeLlama | 7B | Efficient, broad language support | General coding assistance |
| CodeLlama | 13B | Better reasoning, higher accuracy | More complex programming |
| CodeLlama | 34B | State-of-the-art code generation | Production-level code, complex algorithms |
| WizardCoder | 7B | Strong Python performance | Python-focused development |
| Mistral | 7B | Good all-rounder with efficiency | Balanced coding assistance |
| Llama 3 | 70B | Advanced reasoning for complex logic | Architectural design, tough debugging |

> **Pro Tip:** For day-to-day coding, the CodeLlama 13B model offers an excellent balance between quality and resource requirements.

## Core Parameter Optimization for Code Completion

### Context Length (`--ctx`)

For code completion, context length determines how much of your codebase the model can "see":

```bash
# For function/method level work
ollama run codellama:13b --ctx 4096

# For class/module level assistance
ollama run codellama:13b --ctx 8192

# For project-level understanding
ollama run codellama:34b --ctx 16384
```

**Think about context in terms of code structures:**
- 4096 tokens ≈ A few functions plus documentation
- 8192 tokens ≈ A complete class or small module
- 16384+ tokens ≈ Multiple files or complex systems

Having sufficient context allows the model to understand dependencies, imports, class hierarchies, and variable scopes.

### Temperature and Top-P

For code generation, these parameters control the determinism and creativity of suggestions:

```bash
# For precise, deterministic code (production use)
ollama run codellama:13b --temp 0.1 --top-p 0.8

# For balanced suggestions (typical development)
ollama run codellama:13b --temp 0.3 --top-p 0.85

# For brainstorming alternative approaches
ollama run codellama:13b --temp 0.7 --top-p 0.9
```

**Finding the right balance:**
- Lower temperature (0.1-0.3): More predictable, conventional code
- Medium temperature (0.4-0.6): Good for general development
- Higher temperature (0.7+): More varied suggestions (useful for brainstorming)

For most coding scenarios, you'll want more deterministic outputs (lower temperature) than you would for creative writing.

### Stop Sequences

For code completion, proper stop sequences help generate well-formed code blocks:

```bash
# Adding custom stop tokens for Python function completion
ollama run codellama:13b --temp 0.2 --stop "# " --stop "def " --stop "class "
```

Common stop sequences for code generation:
- `--stop "```"` - Stop at code block boundaries
- `--stop "def "` - Stop before a new function definition 
- `--stop "class "` - Stop before a new class definition
- `--stop "if __name__ == '__main__':"` - Stop at Python main entry point

This helps prevent the model from continuing to generate beyond the relevant code block.

## Programming Language-Specific Optimization

Different programming languages benefit from slightly different parameters:

### Python Optimization

```bash
# Optimized for Python development
ollama run codellama:13b --temp 0.2 --top-p 0.85 --ctx 8192
```

**Strategy:**
- Lower temperature for Pythonic conventions
- Moderate context for module-level understanding
- Include imports and function definitions in your prompts

### JavaScript/TypeScript Optimization

```bash
# Optimized for JavaScript/TypeScript
ollama run codellama:13b --temp 0.3 --top-p 0.88 --ctx 8192
```

**Strategy:**
- Slightly higher temperature for JS's flexibility
- Include type definitions in your prompt for TypeScript
- Include package.json excerpts for dependency awareness

### Java/C#/C++ Optimization

```bash
# Optimized for statically typed languages
ollama run codellama:34b --temp 0.2 --top-p 0.85 --ctx 12288
```

**Strategy:**
- Larger model for understanding type hierarchies
- Lower temperature for stricter syntax
- Larger context to capture class definitions
- Include imports/includes in your prompts

## Resource-Based Optimization for Code Completion

### Low-End Hardware (8GB RAM, Integrated Graphics)

```bash
# Code assistance on low-end hardware
ollama run codellama:7b-q4 --ctx 2048 --gpu 0 --batch 128
```

**Strategy:**
- Use the smaller 7B CodeLlama model
- Focus on function-level assistance
- Use higher quantization to save memory
- Provide more specific, narrower prompts

### Mid-Range Hardware (16GB RAM, Dedicated GPU with 6-8GB VRAM)

```bash
# Code assistance on mid-range hardware
ollama run codellama:13b --ctx 4096 --gpu 28 --batch 256
```

**Strategy:**
- The 13B model balances capability and resource usage
- Moderate context for class/module understanding
- Split model layers between GPU and CPU

### High-End Hardware (32GB+ RAM, GPU with 16GB+ VRAM)

```bash
# Code assistance on powerful hardware
ollama run codellama:34b --ctx 16384 --gpu 60 --batch 512
```

**Strategy:**
- The largest CodeLlama model for maximum capability
- Extended context for multi-file understanding
- Leverage GPU for faster response time

## Code Task-Specific Optimization

Different coding tasks benefit from different approaches:

### Function Completion

```bash
# Optimized for completing functions
ollama run codellama:13b --temp 0.2 --top-p 0.85 --ctx 4096
```

**Prompt structure:**
```
Complete the following Python function that [describe what the function does]:

```python
def calculate_optimal_price(cost_price, competitor_prices, market_demand):
    """
    Calculate the optimal price based on cost, competition, and demand.
    
    Args:
        cost_price (float): Our cost of producing the item
        competitor_prices (list): List of competitor prices
        market_demand (float): Elasticity coefficient of market demand
        
    Returns:
        float: The calculated optimal price
    """
    # Your function implementation here
```

Please ensure the function handles edge cases and is efficiently implemented.
```

### Debugging Assistance

```bash
# Optimized for debugging help
ollama run codellama:13b --temp 0.1 --top-p 0.8 --ctx 8192
```

**Prompt structure:**
```
I'm getting the following error when running my code:

```
TypeError: cannot unpack non-iterable int object at line 42
```

Here's the relevant section of my code:

```python
def process_data(items):
    results = []
    for item in items:
        item_id, count = item
        # Process each item...
        results.append(processed_item)
    return results

# Line 42:
item_id, count = items[0]
```

Please help me understand what's causing this error and how to fix it.
```

### Code Refactoring

```bash
# Optimized for refactoring suggestions
ollama run codellama:34b --temp 0.3 --top-p 0.85 --ctx 12288
```

**Prompt structure:**
```
Please refactor this JavaScript code to improve readability, efficiency, and follow best practices:

```javascript
function getData() {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'https://api.example.com/data');
        xhr.onload = function() {
            if (xhr.status === 200) {
                var data = JSON.parse(xhr.responseText);
                var result = [];
                for (var i = 0; i < data.length; i++) {
                    if (data[i].active === true) {
                        result.push({
                            id: data[i].id,
                            name: data[i].name
                        });
                    }
                }
                resolve(result);
            } else {
                reject(Error(xhr.statusText));
            }
        };
        xhr.onerror = function() {
            reject(Error("Network Error"));
        };
        xhr.send();
    });
}
```

Explain your improvements and why they matter.
```

## Creating a Code-Optimized Modelfile

For coding tasks you'll repeat often, create a custom Modelfile with optimized parameters:

```
FROM codellama:13b

# Performance parameters for code generation
PARAMETER num_ctx 8192
PARAMETER num_batch 256
PARAMETER num_gpu 40

# Generation parameters for precise code
PARAMETER temperature 0.2
PARAMETER top_p 0.85
PARAMETER repeat_penalty 1.1
PARAMETER stop "```"
PARAMETER stop "def "
PARAMETER stop "class "

# System prompt for code assistant
SYSTEM """
You are an expert programming assistant with deep knowledge of software development best practices.
When writing code:
1. Prioritize correctness and efficiency
2. Follow the language-specific conventions and best practices
3. Include helpful comments explaining complex logic
4. Consider edge cases and input validation
5. Focus on readability and maintainability
6. When appropriate, explain your implementation choices
"""
```

Save this as `Modelfile` and create your code-optimized model:

```bash
ollama create code-assistant -f Modelfile
ollama run code-assistant
```

## Prompt Engineering for Code Tasks

The way you structure your coding prompts has a significant impact on the quality of the generated code:

### Effective Code Prompt Structure

```
Language: [programming language]
Task: [describe what you're trying to accomplish]

Context:
```[language]
[existing code or relevant code snippets]
```

Requirements:
- [specific requirement 1]
- [specific requirement 2]
- [specific requirement 3]

Additional notes:
- [any frameworks or libraries to use]
- [coding style preferences]
- [performance considerations]
```

### Example: Effective Python Flask Endpoint Prompt

```
Language: Python
Task: Create a Flask API endpoint that retrieves user data from a database

Context:
```python
# Existing code
from flask import Flask, jsonify
import sqlite3

app = Flask(__name__)

def get_db_connection():
    conn = sqlite3.connect('database.db')
    conn.row_factory = sqlite3.Row
    return conn
```

Requirements:
- Create a GET endpoint at /users/<user_id>
- Return user details as JSON
- Handle case when user doesn't exist
- Include proper error handling

Additional notes:
- Follow RESTful API principles
- Include appropriate HTTP status codes
- Ensure connection is properly closed
```

## Measuring Code Completion Performance

When optimizing for code tasks, focus on these metrics:

1. **Accuracy:** Does the code work as expected without bugs?
2. **Relevance:** Does the code actually solve the specified problem?
3. **Response time:** How quickly can you get usable code suggestions?
4. **Consistency:** Does the model produce reliable results across sessions?

You can evaluate response time using Ollama's benchmarking:

```bash
ollama run codellama:13b "Write a Python function to find the nth Fibonacci number using memoization" -f
```

## Example Code Completion Configurations

### Daily Python Development

```bash
ollama run codellama:13b --ctx 8192 --gpu 28 --batch 256 --temp 0.2 --top-p 0.85
```

**Explanation:**
- 13B model offers good balance of capability and efficiency
- 8192 context handles typical module-level code
- Low temperature for conventional, reliable Python code
- Works well on mid-range development hardware

### Web Development (JavaScript/TypeScript)

```bash
ollama run codellama:7b --ctx 4096 --gpu 16 --batch 256 --temp 0.3 --top-p 0.9
```

**Explanation:**
- 7B model is responsive enough for interactive web development
- 4096 context for component/module level understanding
- Slightly higher temperature for the creative aspects of front-end work
- Efficient enough to run alongside heavy development tools

### Enterprise Back-End Development

```bash
ollama run codellama:34b --ctx 16384 --gpu 60 --batch 512 --temp 0.1 --top-p 0.8
```

**Explanation:**
- Larger model for complex enterprise patterns
- Extended context to understand service relationships
- Very low temperature for consistent, conventional code
- Optimized for larger workstation or development server

## Advanced Techniques for Code Completion

### Multi-File Context Management

When working with a complex project spanning multiple files:

1. Add minimal but sufficient imports and references
2. Include key type definitions or interfaces
3. Add brief comments explaining what's missing

Example prompt:

```
I'm working on a Java Spring Boot application with these relevant files:

File: UserController.java
```java
package com.example.api.controller;

import com.example.api.model.User;
import com.example.api.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    // Existing endpoints...
}
```

File: UserService.java (partial)
```java
package com.example.api.service;

import com.example.api.model.User;
import java.util.List;

public interface UserService {
    List<User> getAllUsers();
    User getUserById(Long id);
    // Other methods...
}
```

File: User.java (summary)
```java
// This is a JPA entity with id, username, email, and createdAt fields
```

Task: Add a new endpoint in UserController to search users by email domain with pagination support.
```

### IDE Integration Optimization

If you're integrating Ollama with your IDE (like VS Code):

```bash
# Optimized for VS Code integration
ollama run codellama:7b --ctx 4096 --gpu 16 --batch 128 --temp 0.1 --keepalive 300000
```

**Strategy:**
- Use a smaller, faster model for interactive suggestions
- Set a long keepalive to maintain quick responses
- Use lower temperature for more predictable completions
- Reduce batch size for faster first token generation

## Troubleshooting Code Completion Issues

### Issue: Generated Code Has Syntax Errors

**Solution:** Lower temperature and ensure you're using a code-specific model
```bash
ollama run codellama:13b --temp 0.1
```

### Issue: Code Doesn't Match Project Context

**Solution:** Increase context window and provide more project-specific information
```bash
ollama run codellama:13b --ctx 8192
```

### Issue: Slow Responses During Coding Sessions

**Solution:** Enable keepalive and reduce model size or batch size
```bash
ollama run codellama:7b --keepalive 300000 --batch 128
```

### Issue: Inconsistent Code Style

**Solution:** Provide examples of your preferred style in prompts and use lower temperature
```bash
ollama run codellama:13b --temp 0.1
```

## Language-Specific Modelfiles

For developers who work primarily in one language, specialized modelfiles can be highly effective:

### Python Developer Modelfile

```
FROM codellama:13b

PARAMETER temperature 0.2
PARAMETER top_p 0.85
PARAMETER num_ctx 8192
PARAMETER stop "```"
PARAMETER stop "def "
PARAMETER stop "class "

SYSTEM """
You are a Python development expert focusing on clean, Pythonic code.
When writing Python:
1. Follow PEP 8 style guidelines
2. Use appropriate Python idioms and built-in functions
3. Write comprehensive docstrings in Google style
4. Implement proper type hints (PEP 484)
5. Consider both readability and performance
6. Use list/dict comprehensions where appropriate
7. Properly handle exceptions with specific exception types
"""
```

### JavaScript Developer Modelfile

```
FROM codellama:13b

PARAMETER temperature 0.3
PARAMETER top_p 0.88
PARAMETER num_ctx 8192
PARAMETER stop "```"
PARAMETER stop "function "
PARAMETER stop "class "

SYSTEM """
You are a JavaScript development expert with deep knowledge of modern JS.
When writing JavaScript:
1. Use ES6+ features appropriately
2. Prefer const/let over var
3. Use async/await for asynchronous operations
4. Follow proper error handling patterns
5. Write clean, modular code with named exports
6. Use descriptive variable and function names
7. Consider browser compatibility where relevant
"""
```

## Next Steps

Now that you've optimized your Ollama setup for code completion, consider exploring:

- [Custom Modelfiles with Optimized Parameters](../custom-modelfiles.md) for creating specialized coding assistants
- [GPU Optimization Guide](../gpu-optimization.md) for faster code generation
- [Parameter Sweep Testing](../parameter-sweep.md) to find the perfect settings for your development workflow

Remember that code optimization is an iterative process. Pay attention to the quality and correctness of the generated code, and continuously refine your approach based on your specific development needs and coding style.
