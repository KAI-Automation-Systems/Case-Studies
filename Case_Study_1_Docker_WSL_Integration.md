# Case Study: Docker Desktop WSL Integration Failure (ARM64 vs AMD64)

## Context
During the setup of Docker Desktop on Windows 11 devices, multiple integration issues occurred when using the WSL 2 backend. Both ARM64 and AMD64 systems showed similar error behavior when Docker tried to start the `docker-desktop-user-distro` proxy.

> **Error Message:**  
> WSL integration with distro 'Ubuntu' unexpectedly stopped.  
> running wsl distro proxy in Ubuntu distro: running proxy: docker-desktop-user-distro proxy has not started after 4 minutes, killing it.

---

## Problem
Docker Desktop failed to start due to a mismatch between the host CPU architecture and the Docker binaries.  
On AMD64 systems, the ARM build caused startup failure. On ARM64 systems (Snapdragon X Elite), the AMD64 build was incompatible with the ARM-based WSL kernel.

---

## Diagnosis
A detailed system analysis revealed the following root cause:

| System | CPU Architecture | Windows Version | Root Cause |
|---------|-----------------|-----------------|-------------|
| Acer Swift Go 14 AI | AMD64 | Windows 11 Pro x64 | Installed ARM64 Docker build caused WSL proxy crash |
| Asus Vivobook (Snapdragon X Elite) | ARM64 | Windows 11 on ARM | Installed AMD64 Docker build could not connect to ARM-based WSL kernel |

---

## Solution
### Step 1 – Verify WSL configuration
```powershell
wsl --status
wsl --version
wsl --update
wsl --shutdown
```

### Step 2 – Enable required Windows features
1. Open **Windows Features** (`appwiz.cpl` → “Turn Windows features on or off”)  
2. Enable the following:  
   - ✅ Windows Subsystem for Linux  
   - ✅ Virtual Machine Platform  
3. Restart the system

### Step 3 – Remove broken Docker distros
```powershell
wsl --unregister docker-desktop
wsl --unregister docker-desktop-data
```

### Step 4 – Install the correct Docker build
- For **AMD64 systems:** [Docker Desktop AMD64](https://docs.docker.com/desktop/install/windows-install/)  
- For **ARM64 systems:** [Docker Desktop for Windows on ARM (Preview)](https://docs.docker.com/desktop/windows/arm/)

### Step 5 – Verify installation
```bash
docker version
docker run hello-world
```

---

## Result
After reinstalling the correct architecture build, Docker Desktop successfully connected to WSL 2 and the daemon started without delay.  
This resolved the proxy timeout issue and enabled full container operations on both architectures.

---

## Lessons Learned
- WSL 2 always runs with the kernel architecture of the host system.  
- Mixing ARM64 and AMD64 binaries leads to `user-distro proxy` crashes.  
- Hyper-V is **not required** for WSL 2 backend operation.  
- Correct architecture alignment is essential in multi-platform automation environments.

---

## Keywords
`Docker Desktop` · `WSL2` · `ARM64` · `AMD64` · `Windows 11` · `DevOps` · `System Architecture` · `Automation` · `Troubleshooting`
