# Case Study: GPU Acceleration Disabled in Stable Diffusion / ComfyUI

## Context
During AI image generation tests using **ComfyUI** and **Stable Diffusion**, rendering performance was extremely slow.  
Despite having a laptop with a dedicated GPU (NVIDIA RTX series), both applications defaulted to CPU rendering, causing generation times of over 3 minutes per image.

> **Observed Behavior:**  
> - ComfyUI console showed no CUDA device detected.  
> - Stable Diffusion launched but displayed: `Torch not compiled with CUDA enabled`.

---

## Problem
GPU acceleration was not being used because **PyTorch** was installed without CUDA support, or the system's NVIDIA drivers were outdated.  
This issue frequently occurs on new setups where Python environments are created before the GPU toolkit (CUDA + cuDNN) is properly configured.

---

## Diagnosis
1. Checked available devices in Python:
   ```python
   import torch
   print(torch.cuda.is_available())
   print(torch.cuda.get_device_name(0))
   ```
   Output:
   ```
   False
   CUDA error: no device found
   ```

2. Verified installed PyTorch build:
   ```bash
   pip show torch
   ```
   Showed CPU-only version.

3. Checked NVIDIA drivers and CUDA toolkit:
   ```bash
   nvidia-smi
   ```
   Output showed an outdated driver incompatible with the installed PyTorch version.

---

## Solution
### Step 1 – Update GPU drivers
Download and install the latest **NVIDIA Game Ready or Studio Driver** from:  
[https://www.nvidia.com/Download/index.aspx](https://www.nvidia.com/Download/index.aspx)

### Step 2 – Install CUDA and cuDNN
Install the toolkit that matches the PyTorch version. Example for CUDA 12.1:
```bash
winget install Nvidia.CUDA
```

### Step 3 – Reinstall PyTorch with CUDA support
Uninstall old CPU-only versions:
```bash
pip uninstall torch torchvision torchaudio -y
```
Reinstall with GPU support:
```bash
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
```

### Step 4 – Verify GPU activation
```python
import torch
print(torch.cuda.is_available())  # should return True
print(torch.cuda.get_device_name(0))
```
Expected output:
```
True
NVIDIA GeForce RTX 4060 Laptop GPU
```

### Step 5 – Reconfigure ComfyUI / Stable Diffusion
In ComfyUI’s `settings.json`, verify that `"use_cuda": true`.  
Restart the app and test image generation again.

---

## Result
After reinstalling the correct CUDA-enabled PyTorch build, GPU acceleration became active.  
Render times dropped from ~3 minutes to under 15 seconds per image.  
Both ComfyUI and Stable Diffusion now utilize the NVIDIA GPU correctly.

---

## Lessons Learned
- PyTorch must be installed with the correct CUDA version matching the driver and toolkit.  
- Always verify CUDA detection with `torch.cuda.is_available()`.  
- Keeping drivers and toolkits up to date prevents performance bottlenecks.  
- GPU diagnostics should be part of every AI system setup checklist.

---

## Keywords
`ComfyUI` · `Stable Diffusion` · `CUDA` · `PyTorch` · `GPU Acceleration` · `AI Image Generation` · `NVIDIA` · `Deep Learning` · `Troubleshooting`
