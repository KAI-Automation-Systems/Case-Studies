# Case Study: High CPU Usage from WSL Background Processes

## Context
During regular development sessions, the Windows laptop became noticeably slow and fans ramped up.  
Task Manager showed **`vmmem.exe`** and related WSL processes consuming high CPU and memory, even when no visible terminal was running.

> **Observed Behavior:**  
> - `vmmem.exe` uses 30–90% CPU intermittently.  
> - Memory usage grows over time due to idle WSL distros and Docker WSL backend.  

---

## Problem
WSL 2 runs lightweight VMs that can remain active in the background (especially when **Docker Desktop WSL integration** is enabled).  
Idle processes, lingering Node/Python servers, and container daemons keep the WSL VM hot, causing sustained CPU usage.

---

## Diagnosis
1. Listed running WSL distros and their state:
   ```powershell
   wsl --status
   wsl --list --verbose
   ```
   Output indicated multiple running distros (`Ubuntu`, `docker-desktop`, `docker-desktop-data`).

2. Checked Docker Desktop resources:
   - Docker Desktop → **Settings → Resources → WSL Integration** showed multiple distros enabled.  
   - Background containers were running.

3. Verified lingering processes inside Ubuntu:
   ```bash
   ps aux | grep -E "(node|python|gunicorn|uvicorn)"
   ```
   Found an unattended Node server and a Python process left by a previous session.

---

## Solution
### Step 1 – Gracefully shut down all WSL distros
```powershell
wsl --shutdown
```
This immediately releases CPU and memory used by WSL VMs.

### Step 2 – Limit WSL resource usage via `.wslconfig`
Create or edit the file at:
```
%UserProfile%\.wslconfig
```
Add conservative limits (tune to your hardware):
```ini
[wsl2]
memory=6GB        # Limits VM memory
processors=4      # Limits CPU cores
swap=8GB          # Optional, can reduce pressure on RAM
localhostForwarding=true
```
> Restart WSL after changes:
```powershell
wsl --shutdown
```

### Step 3 – Tame Docker Desktop background usage
- Docker Desktop → **Settings → Resources → WSL Integration**: **enable only the distro you actually use**.  
- Disable “Start Docker Desktop when you log in” if not needed.  
- Stop unused containers and images:
  ```bash
  docker ps
  docker stop <container_id>
  ```

### Step 4 – Add a one-click cleanup script (optional)
Create a PowerShell script `wsl_cleanup.ps1`:
```powershell
wsl --shutdown
Stop-Process -Name "Docker Desktop" -ErrorAction SilentlyContinue
```
Run it when you finish a session.

### Step 5 – Manual process check inside WSL
```bash
top -o %CPU
# or
htop
```
Stop lingering services:
```bash
sudo systemctl stop docker || true
pkill -f "node|python" || true
```

---

## Result
After applying resource limits, disabling unnecessary WSL integrations, and shutting down idle distros, **CPU usage dropped to normal levels** and battery life improved.  
The workstation remains responsive, and WSL only consumes resources during active development.

---

## Lessons Learned
- WSL VMs persist in the background; use `wsl --shutdown` to reclaim resources.  
- Configure `.wslconfig` to set hard caps for CPU and memory.  
- Keep Docker Desktop from auto-starting if you don’t need containers every session.  
- Periodically audit background services inside WSL to prevent runaway usage.

---

## Keywords
`WSL2` · `vmmem.exe` · `Windows 11` · `Performance` · `Docker Desktop` · `.wslconfig` · `Resource Limits` · `Dev Environment` · `Troubleshooting`
