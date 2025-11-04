# Case Study: Python ‘ModuleNotFoundError’ in Virtual Environment

## Context
While developing AI and automation scripts, Python modules such as `numpy`, `pandas`, and `openai` could not be imported, even though they appeared to be installed.  
The issue occurred inside a virtual environment (`venv`) used for running local test scripts on Windows 11.

> **Error Message:**  
> `ModuleNotFoundError: No module named 'numpy'`

---

## Problem
The Python interpreter was unable to locate installed packages within the virtual environment.  
This problem often arises due to one or more of the following reasons:
- The virtual environment is not activated before running the script.
- Packages were installed globally instead of inside the `venv`.
- Multiple Python versions are installed and cause interpreter mismatch.

---

## Diagnosis
1. Checked active environment using:
   ```bash
   where python
   where pip
   ```
   Output showed the global Python installation instead of the virtual environment path (`C:\Users\<username>\project\venv\Scripts\python.exe`).

2. Verified installed packages:
   ```bash
   pip list
   ```
   The required modules were missing from the environment.

3. Confirmed that the script was executed using the global interpreter instead of the virtual environment one.

---

## Solution
### Step 1 – Activate the correct virtual environment
For **Windows (PowerShell):**
```powershell
cd C:\Users\<username>\project
venv\Scripts\activate
```

For **Linux / WSL:**
```bash
source venv/bin/activate
```

### Step 2 – Install required packages inside the environment
```bash
pip install numpy pandas openai
```

### Step 3 – Verify installation
```bash
pip list
python -m pip show numpy
```

### Step 4 – Ensure IDE uses the same interpreter
In **VS Code → Command Palette (Ctrl + Shift + P)** → `Python: Select Interpreter` → Choose:
```
C:\Users\<username>\project\venv\Scripts\python.exe
```

### Step 5 – (Optional) Rebuild the environment if broken
If activation still fails, delete and recreate the virtual environment:
```bash
rmdir /s /q venv
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

---

## Result
After activating the correct environment and reinstalling dependencies, all imports worked as expected.  
AI automation scripts executed successfully with full module support inside the isolated `venv` environment.

---

## Lessons Learned
- Always activate the virtual environment before installing packages.  
- Use `where python` or `which python` to confirm interpreter paths.  
- Keep global and project environments separated to avoid dependency conflicts.  
- Define a `requirements.txt` file for reproducible environments across systems.

---

## Keywords
`Python` · `Virtual Environment` · `venv` · `ModuleNotFoundError` · `AI Development` · `Environment Management` · `Dependency Handling` · `Troubleshooting`
