# Case Study: Docker Container Cannot Access Host Network

## Context
While testing an automation microservice locally, the Docker container could not communicate with a web service running on the Windows host machine.  
Even though the host service was reachable via browser (`http://localhost:8080`), the container consistently failed to connect.

> **Error Message:**  
> `curl: (7) Failed to connect to host.docker.internal port 8080: Connection refused`

---

## Problem
Containers on Windows and Linux run in a virtualized network namespace.  
By default, Docker isolates container traffic from the host system.  
If the container is not properly configured to access the host network, requests to `localhost` or `127.0.0.1` will fail.

---

## Diagnosis
1. Confirmed that the web service was running on the Windows host:
   ```bash
   netstat -an | find "8080"
   ```
   Output showed:  
   `LISTENING 127.0.0.1:8080`

2. Checked container network settings:
   ```bash
   docker inspect <container_id> --format='{{.HostConfig.NetworkMode}}'
   ```
   Result: `bridge` (default isolated network).

3. Attempted connection from container shell:
   ```bash
   docker exec -it <container_id> ping host.docker.internal
   ```
   Ping failed → host alias not resolving correctly in WSL environment.

---

## Solution
### Step 1 – Use Docker’s special DNS hostname
Docker provides a built-in DNS entry that points to the host:
```bash
curl http://host.docker.internal:8080
```
If this does not work on Linux, manually add it to `/etc/hosts` inside the container:
```bash
172.17.0.1 host.docker.internal
```

### Step 2 – Use `--network host` mode (Linux only)
When running the container:
```bash
docker run --network host my-container
```
> ⚠️ On Windows and macOS this flag is not available, only on Linux.

### Step 3 – Forward ports from host to container
If `--network host` is not supported:
```bash
docker run -p 8080:8080 my-container
```
This exposes the same port on the container and the host.

### Step 4 – Verify connection
Inside the container:
```bash
curl http://host.docker.internal:8080
```
The request should now return a valid response from the host application.

---

## Result
After adjusting the Docker network configuration and correctly referencing the host through `host.docker.internal`, the container successfully connected to the local web service.  
This allowed automated workflows and testing environments to communicate seamlessly between host and container.

---

## Lessons Learned
- Containers cannot access host `localhost` directly; use `host.docker.internal` instead.  
- `--network host` works only on Linux-based Docker environments.  
- Port forwarding remains the safest and most portable solution across operating systems.  
- Network isolation is intentional — only expose ports that are required for automation or testing.

---

## Keywords
`Docker` · `Networking` · `Bridge Mode` · `Host Network` · `Container Communication` · `Port Forwarding` · `Automation` · `DevOps` · `Troubleshooting`
