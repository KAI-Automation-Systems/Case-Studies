# Case Study: Git Commit Identity Mismatch (‘unknown author’)

## Context
While pushing updates to a GitHub repository, commit history displayed incorrect or missing author information.  
Instead of showing the expected username and email, GitHub labeled commits as “unknown” or showed a generic account ID.

> **Observed Behavior:**  
> - Commits in GitHub UI displayed as: `unknown <unknown@unknown>`  
> - Local `git log` showed mismatched or empty user identity.  

---

## Problem
Git uses local configuration settings to identify the author of each commit.  
If these settings are missing, incorrect, or not matching a verified GitHub account, commits appear anonymous.  
This is common when using multiple systems or when Git was installed without user configuration.

---

## Diagnosis
1. Checked current Git identity:
   ```bash
   git config --global user.name
   git config --global user.email
   ```
   Output was empty, confirming no identity set.

2. Verified commit author on a specific commit:
   ```bash
   git log -1 --pretty=format:'%an <%ae>'
   ```
   Showed: `unknown <unknown@unknown>`

3. Confirmed mismatch with GitHub account email.  
   The local email did not match the primary email in the GitHub profile.

---

## Solution
### Step 1 – Configure global Git identity
```bash
git config --global user.name "KAI-Automation-Systems"
git config --global user.email "kevinmast.km@gmail.com"
```

### Step 2 – Verify configuration
```bash
git config --list
```
Ensure the identity is listed correctly.

### Step 3 – Re-sign old commits (optional)
If you want previous commits to display correctly:
```bash
git commit --amend --reset-author
git push --force
```

### Step 4 – Verify GitHub connection
Check SSH or HTTPS authentication:
```bash
git remote -v
```
If using SSH, ensure your key is added to GitHub:
```bash
ssh -T git@github.com
```
Expected message:  
`Hi <username>! You've successfully authenticated.`

---

## Result
All new commits now display correctly on GitHub with the proper author name and email.  
Older commits can be updated manually if required.  
The Git configuration now matches the global identity used for KAI Automation Systems projects.

---

## Lessons Learned
- Always verify `git config` after a fresh installation or on new machines.  
- Git author identity must match your verified GitHub email address.  
- Use SSH keys for secure authentication across multiple repositories.  
- Consistent Git configuration ensures clean project history for recruiters and collaborators.

---

## Keywords
`Git` · `GitHub` · `Commit Author` · `SSH` · `Version Control` · `Configuration` · `Identity` · `Automation Engineering` · `Troubleshooting`
