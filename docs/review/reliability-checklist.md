# Reliability and Resilience Checklist

This checklist ensures code changes implement proper error handling, resilience patterns, and fault tolerance to maintain system reliability.

---

## 1. Timeouts and Retries {#timeouts-retries}

### Timeout Configuration
- [ ] All external HTTP calls have explicit timeouts
- [ ] Database queries have timeout configurations
- [ ] Message queue operations have timeouts
- [ ] Timeout values are reasonable for the operation type

**Timeout Guidelines:**
- API calls: 5-30 seconds (depending on operation)
- Database queries: 5-10 seconds for OLTP
- File uploads: Based on expected file size
- Streaming operations: Heartbeat/keepalive configured

**Example:**
```typescript
// ❌ BAD - No timeout
const response = await axios.get('https://api.example.com/data');

// ✅ GOOD - Explicit timeout with abort controller
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await axios.get('https://api.example.com/data', {
    signal: controller.signal,
    timeout: 5000
  });
  return response.data;
} finally {
  clearTimeout(timeoutId);
}
```

### Retry Logic with Exponential Backoff {#exponential-backoff}
- [ ] Transient failures retried with exponential backoff
- [ ] Maximum retry attempts configured (typically 3-5)
- [ ] Jitter added to prevent thundering herd
- [ ] Idempotent operations marked for safe retries
- [ ] Non-retriable errors identified and handled separately

**Example:**
```python
import time
import random
from typing import Callable, Any

def retry_with_backoff(
    func: Callable,
    max_attempts: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0
) -> Any:
    """Retry function with exponential backoff and jitter"""
    for attempt in range(max_attempts):
        try:
            return func()
        except TransientError as e:
            if attempt == max_attempts - 1:
                raise
            
            # Exponential backoff with jitter
            delay = min(base_delay * (2 ** attempt), max_delay)
            jitter = random.uniform(0, delay * 0.1)
            time.sleep(delay + jitter)
            
            logger.warning(f"Retry attempt {attempt + 1}/{max_attempts} after {delay}s")
    
# Usage
result = retry_with_backoff(lambda: api_client.fetch_data())
```

---

## 2. Circuit Breakers {#circuit-breakers}

### Circuit Breaker Pattern
- [ ] Circuit breakers implemented for critical external dependencies
- [ ] Failure threshold configured appropriately
- [ ] Half-open state testing implemented
- [ ] Circuit breaker state monitored and alerted

**Circuit Breaker States:**
- **Closed**: Normal operation, requests pass through
- **Open**: Failures exceeded threshold, requests fail fast
- **Half-Open**: Testing if service recovered, limited requests allowed

**Example:**
```java
// ✅ GOOD - Circuit breaker with Resilience4j
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public PaymentResponse processPayment(PaymentRequest request) {
    return paymentServiceClient.process(request);
}

public PaymentResponse paymentFallback(PaymentRequest request, Exception ex) {
    logger.error("Payment service unavailable, using fallback", ex);
    // Return cached response or queue for later processing
    return queuePaymentForLater(request);
}
```

### Fallback Strategies
- [ ] Fallback behavior defined for circuit breaker trips
- [ ] Degraded mode implemented when dependencies unavailable
- [ ] Default values or cached data used when appropriate
- [ ] User-friendly error messages for service unavailability

---

## 3. Idempotency {#idempotency}

### Idempotent Operations
- [ ] POST/PUT/PATCH operations with side effects are idempotent
- [ ] Idempotency keys used for critical operations (payments, orders)
- [ ] Duplicate request detection implemented
- [ ] Database constraints prevent duplicate entries

**Example:**
```typescript
// ✅ GOOD - Idempotent payment processing
router.post('/api/payments', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  
  if (!idempotencyKey) {
    return res.status(400).json({ error: 'Idempotency-Key header required' });
  }
  
  // Check if request was already processed
  const existing = await PaymentLog.findOne({ idempotencyKey });
  if (existing) {
    return res.status(200).json(existing.response);
  }
  
  // Process payment
  const result = await processPayment(req.body);
  
  // Store result with idempotency key
  await PaymentLog.create({
    idempotencyKey,
    request: req.body,
    response: result,
    createdAt: new Date()
  });
  
  res.status(200).json(result);
});
```

### State Management
- [ ] Operations can be safely retried without side effects
- [ ] State transitions are atomic
- [ ] Partial failures handled gracefully
- [ ] Compensating transactions defined for failures

---

## 4. Error Handling {#error-handling}

### Comprehensive Error Handling
- [ ] All error paths explicitly handled (no silent failures)
- [ ] Errors logged with sufficient context
- [ ] Errors categorized (transient vs permanent, retriable vs non-retriable)
- [ ] Stack traces captured for debugging

**Example:**
```javascript
// ❌ BAD - Poor error handling
async function getUserData(userId) {
  try {
    return await api.getUser(userId);
  } catch (e) {
    console.log('Error');
    return null;
  }
}

// ✅ GOOD - Comprehensive error handling
async function getUserData(userId) {
  try {
    return await api.getUser(userId);
  } catch (error) {
    // Log with context
    logger.error('Failed to fetch user data', {
      userId,
      error: error.message,
      stack: error.stack,
      endpoint: error.config?.url
    });
    
    // Categorize error
    if (error.response?.status === 404) {
      throw new UserNotFoundError(userId);
    } else if (error.response?.status === 429) {
      throw new RateLimitError('API rate limit exceeded');
    } else if (error.code === 'ECONNABORTED') {
      throw new TimeoutError('Request timeout');
    }
    
    throw new ExternalServiceError('Failed to fetch user data', error);
  }
}
```

### Error Responses
- [ ] Error responses include actionable information
- [ ] Sensitive information not leaked in error messages
- [ ] Error codes standardized (HTTP status codes, custom error codes)
- [ ] Correlation IDs included for request tracing

---

## 5. Graceful Degradation

### Service Dependencies
- [ ] Non-critical dependencies don't block core functionality
- [ ] Fallback mechanisms for optional features
- [ ] Feature flags for toggling functionality
- [ ] Partial responses acceptable when some data unavailable

**Example:**
```python
# ✅ GOOD - Graceful degradation
async def get_product_details(product_id: str):
    # Core data (required)
    product = await db.get_product(product_id)
    
    # Enrichment data (optional)
    recommendations = []
    reviews_summary = None
    
    try:
        # Non-critical: product recommendations
        recommendations = await recommendations_service.get(product_id, timeout=2)
    except Exception as e:
        logger.warning(f"Recommendations unavailable: {e}")
    
    try:
        # Non-critical: reviews summary
        reviews_summary = await reviews_service.get_summary(product_id, timeout=2)
    except Exception as e:
        logger.warning(f"Reviews unavailable: {e}")
    
    return {
        "product": product,
        "recommendations": recommendations,
        "reviews": reviews_summary
    }
```

---

## 6. Rate Limiting {#rate-limiting}

### API Rate Limiting
- [ ] Rate limiting implemented on public endpoints
- [ ] Rate limits appropriate for use case (per user, per IP, global)
- [ ] Rate limit headers included in responses
- [ ] Graceful handling when limits exceeded

**Example:**
```javascript
// ✅ GOOD - Rate limiting with express-rate-limit
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later',
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Rate limit exceeded',
      retryAfter: req.rateLimit.resetTime
    });
  }
});

router.post('/api/login', apiLimiter, loginHandler);
```

### Resource Protection
- [ ] Request size limits enforced
- [ ] Connection limits configured
- [ ] Query complexity limits for GraphQL APIs
- [ ] Concurrent request limits per user/session

---

## 7. Data Consistency and Transactions

### Transaction Management
- [ ] Database transactions used for multi-step operations
- [ ] Transaction isolation level appropriate
- [ ] Transactions kept short to avoid locks
- [ ] Rollback logic implemented for failures

### Eventual Consistency
- [ ] Eventual consistency acceptable cases documented
- [ ] Conflict resolution strategies defined
- [ ] Data reconciliation processes in place

---

## 8. Health Checks and Monitoring

### Health Endpoints
- [ ] Health check endpoint implemented (`/health`, `/healthz`)
- [ ] Readiness probe checks dependencies
- [ ] Liveness probe detects deadlocks
- [ ] Startup probe for slow-starting applications

**Example:**
```typescript
// ✅ GOOD - Comprehensive health checks
router.get('/health/live', (req, res) => {
  // Liveness: Is the app running?
  res.status(200).json({ status: 'alive' });
});

router.get('/health/ready', async (req, res) => {
  // Readiness: Can the app serve traffic?
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalApi: await checkExternalApi()
  };
  
  const allHealthy = Object.values(checks).every(check => check.healthy);
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ready' : 'not ready',
    checks
  });
});
```

---

## Review Checklist Summary

Use this quick checklist during PR reviews:

- [ ] **Timeouts**: All external calls have timeouts, values reasonable
- [ ] **Retries**: Exponential backoff implemented, max attempts defined
- [ ] **Circuit Breakers**: Implemented for critical dependencies, fallbacks defined
- [ ] **Idempotency**: Operations with side effects are idempotent
- [ ] **Error Handling**: All paths handled, errors logged with context
- [ ] **Degradation**: Non-critical failures don't block core functionality
- [ ] **Rate Limiting**: Implemented on public endpoints
- [ ] **Health Checks**: Liveness and readiness probes implemented

---

## References

- [Microsoft Azure Reliability Patterns](https://docs.microsoft.com/en-us/azure/architecture/framework/resiliency/reliability-patterns)
- [AWS Well-Architected Framework - Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/)
- [Resilience4j Documentation](https://resilience4j.readme.io/)
- [Release It! (Book) - Design and Deploy Production-Ready Software](https://pragprog.com/titles/mnee2/release-it-second-edition/)
