# Case Study: n8n Google API Rate Limit / Retry Logic Implementation

## Context
An n8n automation workflow periodically fetched data from Google APIs (Sheets & Gmail).  
Under higher load, multiple nodes started failing with **HTTP 429 (Too Many Requests)** and intermittent **5xx** responses, causing workflow interruptions and partial data syncs.

> **Observed Errors:**  
> - `HTTPError: 429 Resource has been exhausted (e.g. quota exceeded).`  
> - `Error: 503 back-end error`  
> - `Error: socket hang up`

---

## Problem
The workflow executed bursts of requests (parallel node execution) without respecting Google’s dynamic quotas.  
Without a retry strategy and rate control, transient failures halted the entire workflow, creating gaps in downstream processes.

---

## Diagnosis
1. Enabled verbose logging on failing nodes (Google Sheets / Gmail):  
   - Captured response headers: `x-ratelimit-limit`, `x-ratelimit-remaining`, `retry-after` (when present).  
2. Identified parallel execution spikes in n8n (multiple branches hitting the same API).  
3. Confirmed that default n8n retries were insufficient for quota bursts and needed **jittered exponential backoff** plus **request pacing**.

---

## Solution
### Step 1 – Add Exponential Backoff with Jitter (Function Node)
Use a Function node to calculate backoff delays based on attempt count:
```javascript
// Input: items[0].json.attempt or default 1
const attempt = (items[0].json.attempt ?? 1);
const base = 500; // 0.5s
const max = 30000; // 30s
const jitter = Math.floor(Math.random() * 300); // 0-300ms
let delay = Math.min(max, Math.round(base * Math.pow(2, attempt - 1)) + jitter);
return [{ json: { delay, attempt } }];
```

### Step 2 – Implement Delay Before Each Google Node
Insert an **n8n Delay node** before the Google node and set it to **“Use expression”** → `{{ $json.delay }}`.  
On failure, increment attempt and loop back to the Function node:

```javascript
// In a Set/Function node after a failure
return [{ json: { attempt: ($json.attempt ?? 1) + 1 } }];
```

### Step 3 – Centralize Request Pace (Queue)
Introduce a **central queue** pattern with a **Rate Limit** (e.g., 5 req/s) using n8n’s **Interval + Split In Batches**:
- **Interval** node triggers every 1s.  
- **Split In Batches** sends small chunks (e.g., 5 items) to Google nodes.  
- Downstream paths wait for completion before releasing the next batch.

### Step 4 – Respect `retry-after` Header When Present
In an **HTTP Request** node (or a Function item wrapper), parse response headers and override delay:
```javascript
const retryAfter = $json.headers?.['retry-after'];
if (retryAfter) {
  // retry-after can be seconds or HTTP-date; handle seconds here
  const delay = (parseInt(retryAfter, 10) || 1) * 1000;
  return [{ json: { delay, attempt: ($json.attempt ?? 1) + 1 } }];
}
```

### Step 5 – Idempotency & State
Ensure duplicates don’t cause corruption when retries occur:
- Use **Upsert** operations (Sheets: update by row key / index).  
- Store a **dedup token** (message-id / checksum) and skip if already processed.  
- Persist last successful page/row in a **Static Data** or **Data Store** node.

### Step 6 – Observability
- Add a **Merge node (By Pass Through)** to a **Notification** path for failures after N attempts (e.g., N=5).  
- Send a Slack/Email alert with context: endpoint, attempt count, last status code, next retry delay.

---

## Result
With **jittered exponential backoff**, **centralized rate limiting**, and **retry-after** handling, workflow success rate increased from ~78% to **~99.5%** during peak windows.  
Transient Google API throttling no longer caused full workflow failures; partial retries recovered automatically with idempotent writes.

---

## Lessons Learned
- Google enforces dynamic quotas; plan for **429** and **5xx** as normal transient states.  
- Combine **exponential backoff + jitter** to avoid synchronized retries (“thundering herd”).  
- Use **Split In Batches** or a queue-like pattern to smooth traffic.  
- Design **idempotent** operations so retries are safe.  
- Surface **observability** (alerts, metrics) to detect degradation early.

---

## Keywords
`n8n` · `Google API` · `Rate Limit` · `429` · `Retry-After` · `Exponential Backoff` · `Jitter` · `Split In Batches` · `Idempotency` · `Automation Reliability`
