# Deployment Information - Day 12 Production Agent

## Public URL

**Live Service:** https://day12-agent.onrender.com

**Platform:** Render (Web Service)  
**Status:** Running ✅

---

## Test Commands

### 1. Health Check
```bash
curl https://day12-agent.onrender.com/health
```

**Expected response:**
```json
{
  "status": "ok",
  "version": "1.0.0",
  "environment": "production",
  "uptime_seconds": 123.4,
  "total_requests": 5,
  "checks": {"llm": "mock"},
  "timestamp": "2026-04-17T08:27:05.594965+00:00"
}
```

### 2. Without Authentication (should return 401)
```bash
curl https://day12-agent.onrender.com/ask -X POST \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
```

**Expected response:**
```json
{"detail": "Invalid or missing API key. Include header: X-API-Key: <key>"}
```

### 3. With API Key (should return 200)
```bash
curl https://day12-agent.onrender.com/ask -X POST \
  -H "X-API-Key: dev-key-change-me-in-production" \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello from Render"}'
```

**Expected response:**
```json
{
  "question": "Hello from Render",
  "answer": "Tôi là AI agent được deploy lên cloud...",
  "model": "gpt-4o-mini",
  "timestamp": "2026-04-17T08:27:18.915668+00:00"
}
```

### 4. Rate Limiting Test (should return 429 after 20 requests)
```bash
for i in {1..25}; do
  curl https://day12-agent.onrender.com/ask -X POST \
    -H "X-API-Key: dev-key-change-me-in-production" \
    -H "Content-Type: application/json" \
    -d '{"question": "Test '$i'"}'
  echo ""
done
```

Requests 21-25 should return:
```json
{"detail": "Rate limit exceeded: 20 req/min"}
```

---

## Environment Variables Set

✅ **AGENT_API_KEY** = dev-key-change-me-in-production  
✅ **JWT_SECRET** = [set on Render]  
✅ **ENVIRONMENT** = production  
✅ **APP_NAME** = Production AI Agent  
✅ **RATE_LIMIT_PER_MINUTE** = 20  
✅ **DAILY_BUDGET_USD** = 5.0  

---

## Endpoints Available

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/` | GET | ❌ | Service info |
| `/health` | GET | ❌ | Liveness probe |
| `/ready` | GET | ❌ | Readiness probe |
| `/ask` | POST | ✅ | Ask question to agent |
| `/metrics` | GET | ✅ | Usage metrics (protected) |

---

## Features Verified ✅

- [x] Multi-stage Docker build (image < 200MB)
- [x] Environment variables (no hardcoded secrets)
- [x] API key authentication (X-API-Key header)
- [x] Rate limiting (20 requests/minute)
- [x] Cost guard (daily budget tracking)
- [x] Health check endpoint
- [x] Readiness probe
- [x] Graceful shutdown (SIGTERM handling)
- [x] Structured JSON logging
- [x] Security headers (X-Content-Type-Options, X-Frame-Options)
- [x] CORS enabled (localhost:3000)
- [x] Public URL accessible
- [x] Error handling

---

## Logs

View live logs on Render dashboard: https://dashboard.render.com/

---

## Next Steps for Production

1. **Change AGENT_API_KEY** to something strong
2. **Add real LLM** - set OPENAI_API_KEY to use actual ChatGPT
3. **Enable monitoring** - Sentry, DataDog for error tracking
4. **Add database** - PostgreSQL for persistent conversation history
5. **Use Redis** - For rate limiting and session state instead of in-memory
6. **Set up CI/CD** - Auto-deploy on GitHub push
7. **Add custom domain** - Use your own domain instead of onrender.com
8. **Enable SSL** - Automatic with Render

---

**Deployment completed:** 2026-04-17  
**Lab Status:** ✅ Complete
