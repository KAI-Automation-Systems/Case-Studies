# Case Study: Node.js & NPM Path Error on Windows

## Context
While setting up a local automation environment for n8n and JavaScript-based AI tools, the Node.js installation appeared successful, but both `node` and `npm` commands were not recognized in the terminal.  
This prevented the execution of automation scripts and n8n CLI commands on Windows 11.

> **Error Message:**  
> `'node' is not recognized as an internal or external command, operable program or batch file.`

---

## Problem
Even after installing Node.js via the official installer, the system could not locate the executables.  
This typically occurs when the **PATH environment variable** is not updated correctly or when multiple Node installations (ARM64/x64) conflict on the same system.

---

## Diagnosis
1. Verified Node installation folder:  
   `C:\Program Files\nodejs\` existed and contained `node.exe` and `npm.cmd`.
2. Checked system PATH variable:  
   The Node.js path was missing from the user and system environment variables.
3. Confirmed architecture conflict:  
   Windows ARM64 had mistakenly installed the x64 emulated version, causing inconsistent paths in WSL and Git Bash.

---

## Solution
### Step 1 – Remove incorrect Node installations
Open **Settings → Apps → Installed Apps** and uninstall all Node.js versions.  
Then manually delete any leftover folders from:
```
C:\Program Files\nodejs\
C:\Users\<username>\AppData\Roaming\npm
```

### Step 2 – Install correct version (x64 or ARM64)
Download the proper build from the official site:  
- For **x64 systems (Intel/AMD):** [Node.js 22.x LTS x64](https://nodejs.org/en/download)  
- For **ARM64 systems:** [Node.js 22.x ARM64 build](https://nodejs.org/en/download)

### Step 3 – Verify PATH environment variable
After installation, check PATH variables:
1. Open **System Properties → Environment Variables**.  
2. Under **User variables**, verify:
   ```
   C:\Program Files\nodejs\
   C:\Users\<username>\AppData\Roaming\npm
   ```
3. Restart your terminal or PC.

### Step 4 – Test installation
```bash
node -v
npm -v
```
If both return version numbers, the issue is resolved.

---

## Result
After cleaning up conflicting installations and reinstalling the correct architecture build, both `node` and `npm` executed successfully in Git Bash, PowerShell, and WSL.  
n8n CLI commands such as `n8n start` ran without any dependency errors.

---

## Lessons Learned
- Always match Node.js architecture (x64 or ARM64) to your Windows system.  
- Ensure PATH variables are automatically or manually set after installation.  
- Multiple installations can silently conflict — check environment variables if CLI commands fail.  
- Automating environment setup via PowerShell scripts can prevent future configuration drift.

---

## Keywords
`Node.js` · `npm` · `PATH Variable` · `Windows 11` · `n8n` · `Automation` · `CLI` · `Environment Configuration` · `Troubleshooting`
