# Case Study: N8N Workflow Credential Isolation Issue

## Context
During the setup of local n8n automation workflows, multiple users on the same instance noticed that credentials were visible across all accounts.  
This created a potential security risk, as one user could edit or use another user's API credentials.

> **Observed Behavior:**  
> - Credentials for connected Google, Slack, and Notion accounts were shared globally.  
> - Even when using separate n8n user accounts, the same credential set was displayed.  

---

## Problem
The n8n instance was configured to use a single local SQLite database and shared environment variables.  
This caused credentials to be stored in a common context rather than per-user scope.  
In enterprise or collaborative setups, credential isolation is essential to maintain data privacy and prevent accidental data leaks.

---

## Diagnosis
1. Checked environment configuration in `.env` file:  
   ```bash
   N8N_ENCRYPTION_KEY=defaultkey
   N8N_USER_FOLDER=/home/n8n/.n8n
   DB_TYPE=sqlite
   ```
   All users were connecting to the same SQLite database.

2. Verified credential sharing behavior:  
   In the n8n UI, both users could view the same Google API connection.

3. Analyzed logs:  
   ```bash
   docker logs n8n
   ```
   Confirmed credentials were stored centrally without encryption isolation.

---

## Solution
### Step 1 – Switch from SQLite to Postgres for user isolation
Modify the `.env` configuration:
```bash
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n_user
DB_POSTGRESDB_PASSWORD=securepassword
```

### Step 2 – Use environment variable separation per user
Create distinct `.env` files or Docker Compose profiles:
```bash
version: '3'
services:
  n8n_user1:
    image: n8nio/n8n
    env_file: ./envs/user1.env
  n8n_user2:
    image: n8nio/n8n
    env_file: ./envs/user2.env
```

Each environment file contains a unique encryption key:
```bash
N8N_ENCRYPTION_KEY=unique_key_user1
```

### Step 3 – Mount isolated data folders
Ensure each user instance has a separate storage directory:
```bash
volumes:
  - ./data/user1:/home/n8n/.n8n
```

### Step 4 – Restart all containers
```bash
docker compose down
docker compose up -d
```

---

## Result
Each n8n user now has a fully isolated environment, with unique encryption keys and separated credential storage.  
Credentials are no longer shared or visible between accounts.  
This configuration aligns with enterprise-grade deployment standards.

---

## Lessons Learned
- SQLite is ideal for single-user setups but insecure for multi-user environments.  
- Credential isolation requires separate databases or unique encryption keys.  
- Docker Compose can manage multiple isolated instances efficiently.  
- Always verify credential visibility before scaling automation systems in teams.

---

## Keywords
`n8n` · `Credentials` · `Security` · `Automation` · `Docker` · `PostgreSQL` · `Multi-User` · `Environment Variables` · `Troubleshooting`
