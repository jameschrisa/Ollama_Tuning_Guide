# Common Issues & Solutions

Even with optimal hyperparameter settings, you may encounter issues when running models through Ollama. This troubleshooting guide will help you diagnose and resolve common problems, covering everything from basic installation issues to advanced performance troubleshooting.

## Installation and Setup Issues

### Issue: Ollama Installation Fails

**Symptoms:**
- Installation script errors out
- Package manager reports conflicts or missing dependencies

**Solutions:**

1. **Verify system requirements:**
   ```bash
   # Check OS version
   lsb_release -a   # Linux
   sw_vers           # macOS
   ```

2. **Try the manual installation method:**
   ```bash
   # On Linux
   curl -fsSL https://ollama.com/install.sh -o install.sh
   chmod +x install.sh
   
   # Inspect the script
   less install.sh
   
   # Run with verbose output
   ./install.sh -v
   ```

3. **Check for conflicting installations:**
   ```bash
   # Look for existing Ollama installations
   which ollama
   ps aux | grep ollama
   ```

### Issue: "No such file or directory" when running Ollama

**Symptoms:**
- `ollama: command not found` error
- `No such file or directory` error

**Solutions:**

1. **Check and update PATH:**
   ```bash
   echo $PATH
   
   # Add to PATH if needed
   echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
   source ~/.bashrc
   ```

2. **Verify installation location:**
   ```bash
   find / -name ollama 2>/dev/null
   ```

3. **Fix permissions:**
   ```bash
   sudo chmod +x /usr/local/bin/ollama
   ```

## Model Download Issues

### Issue: Model Download Gets Stuck or Fails

**Symptoms:**
- Progress bar hangs at a certain percentage
- Connection errors during model download
- Download fails with timeout errors

**Solutions:**

1. **Check your internet connection:**
   ```bash
   ping ollama.ai
   curl -I https://ollama.ai
   ```

2. **Try with a VPN if regional restrictions might apply**

3. **Resume a partial download:**
   ```bash
   # Remove the partial download
   rm -rf ~/.ollama/models/manifests/registry.ollama.ai/library/[model]
   
   # Try again
   ollama pull [model]
   ```

4. **Download a smaller model first:**
   ```bash
   ollama pull phi:2.7b
   ```

### Issue: "CUDA error: out of memory" When Downloading Model

**Symptoms:**
- Error message about CUDA or GPU memory
- Download fails at a specific percentage

**Solutions:**

1. **Pull with CPU-only mode:**
   ```bash
   CUDA_VISIBLE_DEVICES= ollama pull llama3:70b
   ```

2. **Free up GPU memory:**
   ```bash
   # Check what's using your GPU
   nvidia-smi
   
   # Kill GPU processes if needed
   sudo kill -9 [PID]
   ```

3. **Try a different quantization level:**
   ```bash
   ollama pull llama3:70b-q4
   ```

## Model Running Issues

### Issue: "Out of Memory" Errors

**Symptoms:**
- Model crashes with OOM (Out Of Memory) errors
- System becomes extremely slow or unresponsive
- Error messages about CUDA, VRAM, or system memory

**Solutions:**

1. **Reduce context length:**
   ```bash
   ollama run llama3:70b --ctx 4096  # Try smaller context
   ```

2. **Use fewer GPU layers:**
   ```bash
   # If using --gpu 32, try reducing
   ollama run llama3:70b --gpu 16
   ```

3. **Enable memory mapping:**
   ```bash
   ollama run llama3:70b --mmap 1
   ```

4. **Use a smaller model or higher quantization:**
   ```bash
   ollama run llama3:8b  # Instead of 70B
   # Or
   ollama run llama3:70b-q4  # More aggressive quantization
   ```

5. **Add swap space (Linux):**
   ```bash
   # Create 8GB swap file
   sudo fallocate -l 8G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   
   # Make permanent
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
   ```

### Issue: Model Generates Nonsensical or Repetitive Text

**Symptoms:**
- Output contains gibberish or random characters
- Model keeps repeating the same phrases
- Output is incoherent or irrelevant to the prompt

**Solutions:**

1. **Adjust temperature and top-p settings:**
   ```bash
   # More deterministic output
   ollama run llama3:8b --temp 0.3 --top-p 0.9
   ```

2. **Increase the repetition penalty:**
   ```bash
   ollama run llama3:8b --repeat-penalty 1.2
   ```

3. **Try a different model:**
   ```bash
   ollama run mistral:7b  # Different architecture may perform better
   ```

4. **Verify the model was downloaded correctly:**
   ```bash
   # Remove and re-pull
   ollama rm llama3:8b
   ollama pull llama3:8b
   ```

5. **Create a custom Modelfile with clear system prompts:**
   ```
   FROM llama3:8b
   
   PARAMETER temperature 0.7
   PARAMETER top_p 0.9
   
   SYSTEM """
   You are a helpful AI assistant. Provide clear, concise, and accurate responses.
   """
   ```

### Issue: Extremely Slow Generation

**Symptoms:**
- Model takes minutes to generate a simple response
- Tokens per second (tok/s) is unusually low
- First token latency is very high

**Solutions:**

1. **Check for CPU thermal throttling:**
   ```bash
   # Monitor CPU temperature
   sensors  # Linux
   sudo powermetrics --samplers smc | grep -i "CPU die temperature"  # macOS
   ```

2. **Optimize thread count:**
   ```bash
   # Try different thread counts
   ollama run llama3:8b --threads 4
   ```

3. **Increase batch size:**
   ```bash
   ollama run llama3:8b --batch 512
   ```

4. **Check GPU utilization:**
   ```bash
   nvidia-smi -l 1
   ```

5. **Verify you're using GPU acceleration:**
   ```bash
   # Make sure you're using GPU
   ollama run llama3:8b --gpu 32
   ```

6. **Disable other resource-intensive processes**

## GPU-Specific Issues

### Issue: "Failed to Load CUDA" or "No CUDA-capable device is detected"

**Symptoms:**
- Error message about CUDA drivers or devices
- Model falls back to CPU only
- Very slow performance on systems with GPUs

**Solutions:**

1. **Verify CUDA installation:**
   ```bash
   nvidia-smi
   nvcc --version
   ```

2. **Install or upgrade NVIDIA drivers:**
   ```bash
   # Ubuntu
   sudo apt update
   sudo apt install nvidia-driver-535  # Use latest available
   
   # Check driver compatibility at https://developer.nvidia.com/cuda-gpus
   ```

3. **Set environment variables manually:**
   ```bash
   export CUDA_VISIBLE_DEVICES=0
   export CUDA_HOME=/usr/local/cuda
   ollama run llama3:8b --gpu 32
   ```

4. **Try a different CUDA version:**
   ```bash
   # Install CUDA Toolkit 12.2
   # Follow NVIDIA's instructions for your OS
   ```

### Issue: GPU Memory Fragmentation

**Symptoms:**
- OOM errors despite having sufficient VRAM
- Performance degradation over time
- Inconsistent behavior between runs

**Solutions:**

1. **Restart Ollama service:**
   ```bash
   # Linux
   sudo systemctl restart ollama
   
   # macOS
   pkill ollama
   ollama serve
   ```

2. **Specify exact GPU memory allocation:**
   ```bash
   # Set max_split_size_mb
   PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128 ollama run llama3:8b
   ```

3. **Enable unified memory:**
   ```bash
   # Use unified memory addressing
   CUDA_VISIBLE_DEVICES=0 PYTORCH_CUDA_ALLOC_CONF=garbage_collection_threshold:0.8,max_split_size_mb:128 ollama run llama3:8b
   ```

## Advanced Troubleshooting

### Issue: Unstable Performance (Variance Between Runs)

**Symptoms:**
- Significant differences in generation speed between identical runs
- Inconsistent memory usage
- Variable first token latency

**Solutions:**

1. **Monitor thermal performance:**
   ```bash
   # Watch temperature over time
   watch -n 1 nvidia-smi
   ```

2. **Enable deterministic generation:**
   ```bash
   # Set a fixed seed
   ollama run llama3:8b --seed 42
   ```

3. **Create a script to test consistency:**
   ```bash
   #!/bin/bash
   
   MODEL="llama3:8b"
   PROMPT="Explain the theory of relativity in detail"
   RUNS=5
   
   for i in $(seq 1 $RUNS); do
     echo "Run $i of $RUNS"
     time ollama run $MODEL "$PROMPT" -f > run_$i.txt
     grep "eval rate:" run_$i.txt
     echo "---"
     sleep 10
   done
   ```

### Issue: Memory Leaks

**Symptoms:**
- RAM usage grows over time even when idle
- System becomes progressively slower
- Eventually crashes with OOM errors

**Solutions:**

1. **Set shorter keepalive:**
   ```bash
   # Only keep model in memory for 30 seconds
   ollama run llama3:8b --keepalive 30000
   ```

2. **Run with limited memory allocation:**
   ```bash
   # Use cgroups to limit memory (Linux)
   sudo cgcreate -g memory:ollama
   sudo cgset -r memory.limit_in_bytes=16G ollama
   sudo cgexec -g memory:ollama ollama run llama3:8b
   ```

3. **Monitor and restart when needed:**
   ```bash
   # Create a watchdog script
   if [ $(ps aux | grep ollama | grep -v grep | awk '{print $6}') -gt 16000000 ]; then
     sudo systemctl restart ollama
   fi
   ```

### Issue: Models Not Visible or Missing Files

**Symptoms:**
- `ollama list` doesn't show downloaded models
- Errors about missing model files
- "Failed to initialize model" errors

**Solutions:**

1. **Check Ollama data directory:**
   ```bash
   ls -la ~/.ollama/models/
   ```

2. **Verify permissions:**
   ```bash
   sudo chown -R $(whoami) ~/.ollama/
   ```

3. **Rebuild the model registry:**
   ```bash
   # Stop Ollama
   sudo systemctl stop ollama
   
   # Move current registry
   mv ~/.ollama/models/registry ~/.ollama/models/registry.bak
   
   # Start Ollama and pull models again
   sudo systemctl start ollama
   ollama pull llama3:8b
   ```

## System Integration Issues

### Issue: REST API Connection Problems

**Symptoms:**
- Cannot connect to Ollama API
- Connection refused errors
- Timeouts when trying to use the API

**Solutions:**

1. **Verify Ollama is running:**
   ```bash
   ps aux | grep ollama
   ```

2. **Check if the API is accessible:**
   ```bash
   curl http://localhost:11434/api/tags
   ```

3. **Start with explicit bind address:**
   ```bash
   OLLAMA_HOST=0.0.0.0:11434 ollama serve
   ```

4. **Check firewall settings:**
   ```bash
   # Ubuntu/Debian
   sudo ufw status
   sudo ufw allow 11434/tcp
   
   # macOS
   sudo /usr/libexec/ApplicationFirewall/socketfilterfw --listapps
   ```

### Issue: High CPU/GPU Usage When Idle

**Symptoms:**
- CPU or GPU usage remains high even when not generating text
- Fan noise and heat even when not actively using Ollama
- System responsiveness issues

**Solutions:**

1. **Set a shorter keepalive time:**
   ```bash
   ollama run llama3:8b --keepalive 5000  # 5 seconds
   ```

2. **Explicitly unload the model when done:**
   ```bash
   # In your application code, make an API call to
   # http://localhost:11434/api/show
   # to see what's loaded, then call
   # http://localhost:11434/api/exit
   # to unload when finished
   ```

3. **Create a service with resource limits:**
   ```bash
   # Edit the Ollama systemd service
   sudo systemctl edit ollama.service
   
   # Add resource limits
   [Service]
   CPUQuota=50%
   MemoryMax=16G
   ```

## Modelfile and Custom Model Issues

### Issue: Custom Modelfile Fails to Build

**Symptoms:**
- `ollama create` command fails
- Errors about invalid Modelfile syntax
- Model creation hangs or timeouts

**Solutions:**

1. **Validate Modelfile syntax:**
   ```bash
   # Check for common issues
   grep -n "SYSTEM\\|FROM\\|PARAMETER" Modelfile
   ```

2. **Simplify and incrementally build:**
   ```
   # Start with minimal Modelfile
   FROM llama3:8b
   ```

   Then add parameters one by one, testing after each addition.

3. **Check for space/disk issues:**
   ```bash
   df -h
   ```

4. **Verbose creation to see issues:**
   ```bash
   OLLAMA_DEBUG=1 ollama create mymodel -f Modelfile
   ```

### Issue: Specific Parameter Not Taking Effect

**Symptoms:**
- Changing parameters doesn't affect behavior
- Model ignores certain settings
- Performance doesn't improve despite parameter changes

**Solutions:**

1. **Verify parameter name and format:**
   ```
   # Correct format
   PARAMETER temperature 0.7
   
   # Not
   PARAMETER temp 0.7
   ```

2. **Check for parameter conflicts:**
   ```
   # These might conflict
   PARAMETER num_ctx 4096
   PARAMETER num_ctx_parallel 4096
   ```

3. **Use command-line parameters to override:**
   ```bash
   ollama run mymodel --temp 0.7 --ctx 4096
   ```

4. **Check Ollama version supports the parameter:**
   ```bash
   ollama --version
   ```

## Performance Optimization Issues

### Issue: Performance Doesn't Improve Despite Tuning

**Symptoms:**
- Changing parameters has minimal effect on speed
- Resource utilization remains low despite adjustments
- Generation quality doesn't improve

**Solutions:**

1. **Identify the actual bottleneck:**
   ```bash
   # CPU bound?
   htop
   
   # GPU bound?
   nvidia-smi -l 1
   
   # I/O bound?
   iostat -x 1
   
   # Memory bound?
   free -h
   ```

2. **Try more dramatic parameter changes:**
   ```bash
   # Example: Cut context in half
   ollama run llama3:8b --ctx 2048  # If using 4096
   ```

3. **Verify you're measuring correctly:**
   ```bash
   # Run with benchmark flag
   ollama run llama3:8b "Write a paragraph about AI" -f
   ```

4. **Eliminate environmental factors:**
   - Close other applications
   - Disable background services
   - Turn off power saving modes

### Issue: Significantly Worse Performance Than Expected

**Symptoms:**
- Performance much worse than reported benchmarks
- Extremely slow compared to similar hardware setups
- Model barely runs despite having sufficient hardware

**Solutions:**

1. **Check for hardware issues:**
   ```bash
   # Check for thermal throttling
   sudo dmidecode -t processor
   
   # Check for ECC memory errors (if applicable)
   sudo mcelog
   ```

2. **Verify you're using the right quantization level:**
   ```bash
   # Check what model variant you're using
   ollama list
   
   # Pull a specific quantization if needed
   ollama pull llama3:8b-q4  # q4 version
   ```

3. **Rule out system issues:**
   ```bash
   # Create a clean benchmark environment
   docker run --gpus all -it --rm ubuntu:latest
   # Then install Ollama fresh in the container
   ```

4. **Try a simpler model first:**
   ```bash
   ollama run phi:2.7b "Hello, world"
   ```

## Additional Troubleshooting Tips

### Check Ollama Logs

```bash
# View logs
sudo journalctl -u ollama -f  # Linux with systemd
tail -f ~/.ollama/logs/server.log  # Older installations
```

### Run with Debug Mode

```bash
OLLAMA_DEBUG=1 ollama run llama3:8b "Test prompt"
```

### Check for Resource Contention

```bash
# Check if other processes are using the GPU
nvidia-smi

# Check for high CPU processes
top -o %CPU
```

### Test with Minimal Configuration

```bash
# Reset to defaults
ollama run llama3:8b
```

### Create a Diagnostic Report

```bash
#!/bin/bash
# ollama-diagnostic.sh

echo "Ollama Diagnostic Report"
echo "======================="
echo "Date: $(date)"
echo

echo "System Information:"
echo "------------------"
uname -a
echo

echo "CPU Information:"
lscpu
echo

echo "Memory Information:"
free -h
echo

echo "Disk Space:"
df -h
echo

echo "GPU Information:"
if command -v nvidia-smi &> /dev/null; then
    nvidia-smi
else
    echo "No NVIDIA GPU detected or nvidia-smi not installed"
fi
echo

echo "Ollama Version:"
ollama --version
echo

echo "Installed Models:"
ollama list
echo

echo "Basic Performance Test:"
ollama run llama3:8b "Write a short paragraph about ML." -f
echo

echo "======================="
echo "End of Diagnostic Report"
```

Save this script, make it executable (`chmod +x ollama-diagnostic.sh`), and run it to generate a comprehensive diagnostic report.

### Contact Community Support

If you're still having issues after trying these solutions:

1. **Check the Ollama GitHub repository:**
   - Issues: https://github.com/ollama/ollama/issues
   - Discussions: https://github.com/ollama/ollama/discussions

2. **Join the Ollama Discord community:**
   - Share your diagnostic report
   - Ask for help with your specific issue

3. **Search for similar issues on Stack Overflow**
   - Use the `[ollama]` tag

## Next Steps

Once you've resolved your issues, consider exploring:

- [Parameter Sweep Testing](parameter-sweep.md) to find the best settings for your system
- [Automated Hyperparameter Tuning](automated-tuning.md) for advanced optimization
- [Custom Modelfiles](../custom-modelfiles.md) to create tailored configurations

Remember that troubleshooting is often an iterative process. Start with the simplest potential solutions, test one change at a time, and keep track of what works and what doesn't for your specific setup.
