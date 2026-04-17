# Day 12 Lab - Mission Answers

**Student:** Quang Qui Tran  
**Date:** 2026-04-17

---

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found

1. **API key hardcode** (line 17) - Secret key visible in code, will be exposed if pushed to GitHub
2. **Database URL hardcode** (line 18) - Password hardcoded, security risk
3. **Print() instead of logging** (line 33-34) - Unstructured logs, hard to parse by log aggregators
4. **No health check endpoint** - Platform (Kubernetes/Railway) can't detect crashes
5. **Debug=True + reload=True** - Slower, memory leak, insecure for production
6. **Port hardcoded to 8000** - On cloud platforms, PORT is injected via env var

### Exercise 1.2: Basic version runs ✅
```
curl http://localhost:8000/ask?question=Hello
→ {"answer": "Mock response"}
```

### Exercise 1.3: Comparison table

| Feature | Develop | Production | Why Important? |
|---------|---------|------------|----------------|
| **Config** | Hardcode in code | Environment variables (os.getenv) | Easy to change per env, no secrets in code |
| **Health check** | None | GET /health endpoint | Platform uses to auto-restart if fails |
| **Logging** | print() unstructured | JSON structured logs | Easier to parse by log aggregators (Datadog, CloudWatch) |
| **Shutdown** | Abrupt Ctrl+C | Graceful SIGTERM handler | Complete requests before shutdown, no data loss |
| **Debug mode** | DEBUG=True | DEBUG=False | Dev is slow + memory leaks, prod is fast + safe |
| **Security headers** | None | X-Content-Type-Options, X-Frame-Options | Prevent XSS, clickjacking attacks |

---

## Part 2: Docker Containerization

### Exercise 2.1: Dockerfile questions

1. **Base image:** `python:3.11-slim` - Linux OS with Python 3.11 pre-installed
2. **Working directory:** `/app` - Container's internal folder where code runs
3. **COPY requirements.txt first:** Dependencies rarely change, so Docker caches this layer (faster rebuilds)
4. **CMD vs ENTRYPOINT:**
   - CMD = default arguments (can be overridden)
   - ENTRYPOINT = the program that always runs (cannot override)

### Exercise 2.3: Image size comparison

- **Develop:** 413 MB (full python:3.11 + all tools)
- **Production:** 56.7 MB (slim + multi-stage build)
- **Improvement:** 86% smaller ⚡

**Why?**
- Stage 1 (builder): python:3.11-slim + gcc + build tools → compile packages → creates .whl files
- Stage 2 (runtime): python:3.11-slim + copy .whl files → no gcc needed → much lighter

---

## Part 4: API Security

### Exercise 4.1: API Key Authentication

**Test results:**
```bash
# Without key → 401 Unauthorized ✅
curl http://localhost:8000/ask -X POST
→ {"detail":"Invalid or missing API key..."}

# With key → 200 OK ✅
curl http://localhost:8000/ask -X POST \
  -H "X-API-Key: dev-key-change-me-in-production"
→ {"question":"...","answer":"...","model":"gpt-4o-mini"}
```

**How it works:**
```python
def verify_api_key(api_key: str = Header(...)):
    if api_key != settings.AGENT_API_KEY:
        raise HTTPException(401, "Invalid key")
    return api_key
```

### Exercise 4.4: Cost Guard Implementation

**Logic:**
- Each user has monthly budget ($5-$10)
- Track spending in memory (daily_cost variable)
- Reset at midnight
- If spending > budget → return 402 Payment Required

**Code:**
```python
def check_and_record_cost(input_tokens: int, output_tokens: int):
    global _daily_cost, _cost_reset_day
    
    # Reset if new day
    if today != _cost_reset_day:
        _daily_cost = 0.0
    
    # Check budget
    if _daily_cost >= settings.daily_budget_usd:
        raise HTTPException(503, "Daily budget exhausted")
    
    # Estimate cost: ~$0.00015 per 1k input tokens, $0.0006 per 1k output
    cost = (input_tokens/1000)*0.00015 + (output_tokens/1000)*0.0006
    _daily_cost += cost
```

---

## Part 5: Scaling & Reliability

### Exercise 5.1: Health checks

**Implemented in main.py:**
```python
@app.get("/health")
def health():
    """Liveness probe - is container still alive?"""
    return {"status": "ok"}

@app.get("/ready")  
def ready():
    """Readiness probe - can accept traffic?"""
    if not _is_ready:
        raise HTTPException(503, "Not ready")
    return {"ready": True}
```

### Exercise 5.2: Graceful shutdown

Signal handler catches SIGTERM from container orchestrator, finishes current requests before exiting.

### Exercise 5.3: Stateless design

**Anti-pattern:**
```python
conversation_history = {}  # In memory

@app.post("/ask")
def ask(user_id: str):
    history = conversation_history.get(user_id, [])  # Fails on second instance!
```

**Correct:**
```python
@app.post("/ask")
def ask(user_id: str):
    history = redis.lrange(f"history:{user_id}", 0, -1)  # Works on any instance!
```

Why? When scaling to 3 instances, each has its own memory. Redis is shared.

---

## Summary

✅ Understand dev vs production gaps  
✅ Multi-stage Docker reduces image size 86%  
✅ API key authentication prevents unauthorized access  
✅ Rate limiting + cost guard protect budget  
✅ Health checks enable auto-restart  
✅ Graceful shutdown prevents data loss  
✅ Stateless design enables horizontal scaling  
