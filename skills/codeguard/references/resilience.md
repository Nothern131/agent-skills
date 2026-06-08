# Resilience Rules

10 rules that detect missing fault tolerance and resilience patterns.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| RES001 | High | Network operation missing retry mechanism |
| RES002 | Medium | External call missing circuit breaker |
| RES003 | High | Async operation missing timeout control |
| RES004 | Medium | API endpoint missing rate limiting |
| RES005 | High | Critical function missing degradation plan |
| RES006 | High | Write operation missing idempotency protection |
| RES007 | Medium | Message queue missing dead letter queue |
| RES008 | Low | Multi-service call missing isolation |
| RES009 | Medium | Startup connection missing retry |
| RES010 | Medium | Concurrent operation missing limit |

## Detection Patterns

### RES001 — Network operation missing retry mechanism

**What to look for:** HTTP requests, API calls, or network operations that fail without retry.

**Patterns:**
```javascript
const response = await fetch('https://api.example.com/data');
const data = await response.json();
// No retry on failure
```
```python
response = requests.get("https://api.example.com/data")
data = response.json()  # No retry
```

**Fix:**
```javascript
async function fetchWithRetry(url, options = {}, retries = 3, delay = 1000) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response;
      if (response.status >= 500 && i < retries - 1) {
        await sleep(delay * Math.pow(2, i));  // Exponential backoff
        continue;
      }
      throw new HttpError(response.status, response.statusText);
    } catch (err) {
      if (i === retries - 1) throw err;
      await sleep(delay * Math.pow(2, i));
    }
  }
}
```
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
def fetch_data(url):
    response = requests.get(url)
    response.raise_for_status()
    return response.json()
```

### RES002 — External call missing circuit breaker

**What to look for:** Calls to external services without circuit breaker protection. If the external service goes down, it will cause cascading failures.

**Patterns:**
```javascript
async function getRecommendations(userId) {
  const response = await recommendationService.get(userId);
  return response.data;
}
// If recommendation service is down, every request hangs or fails
```

**Fix:**
```javascript
const breaker = new CircuitBreaker({
  timeout: 3000,
  errorThreshold: 50,
  resetTimeout: 30000,
});

async function getRecommendations(userId) {
  try {
    return await breaker.fire(() => recommendationService.get(userId));
  } catch (err) {
    if (breaker.open) {
      logger.warn('Recommendation service circuit open, using fallback');
      return getCachedRecommendations(userId);
    }
    throw err;
  }
}
```

### RES003 — Async operation missing timeout control

**What to look for:** await calls without a timeout, which can hang indefinitely.

**Patterns:**
```javascript
const result = await externalService.process(data);  // No timeout
```
```python
response = requests.get("https://slow-api.example.com/data")  # No timeout
```

**Fix:**
```javascript
async function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) => setTimeout(() => reject(new TimeoutError(ms)), ms)),
  ]);
}

const result = await withTimeout(externalService.process(data), 5000);
```
```python
response = requests.get("https://slow-api.example.com/data", timeout=5)
```

### RES004 — API endpoint missing rate limiting

**What to look for:** Public-facing API endpoints without rate limiting middleware.

**Patterns:**
```javascript
app.post('/api/login', (req, res) => {
  // No rate limiting — vulnerable to brute force
  const token = authenticate(req.body);
  res.json({ token });
});
```

**Fix:**
```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,
  message: 'Too many login attempts, please try again later',
});

app.post('/api/login', loginLimiter, (req, res) => {
  const token = authenticate(req.body);
  res.json({ token });
});
```

### RES005 — Critical function missing degradation plan

**What to look for:** Essential features with no fallback when dependencies are unavailable.

**Patterns:**
```javascript
async function getProductPage(productId) {
  const product = await productService.get(productId);
  const reviews = await reviewService.get(productId);
  const recommendations = await recService.get(productId);
  // If any service is down, the whole page fails
  return { product, reviews, recommendations };
}
```

**Fix:**
```javascript
async function getProductPage(productId) {
  const product = await productService.get(productId);  // Essential

  const [reviews, recommendations] = await Promise.allSettled([
    reviewService.get(productId),
    recService.get(productId),
  ]);

  return {
    product,
    reviews: reviews.status === 'fulfilled' ? reviews.value : [],
    recommendations: recommendations.status === 'fulfilled' ? recommendations.value : [],
    degraded: reviews.status === 'rejected' || recommendations.status === 'rejected',
  };
}
```

### RES006 — Write operation missing idempotency protection

**What to look for:** POST/PUT handlers that create or modify data without idempotency keys, risking duplicate operations on retry.

**Patterns:**
```javascript
app.post('/api/orders', (req, res) => {
  const order = createOrder(req.body);
  res.json(order);  // Retrying creates duplicate orders
});
```

**Fix:**
```javascript
app.post('/api/orders', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  if (idempotencyKey) {
    const existing = await getOrderByIdempotencyKey(idempotencyKey);
    if (existing) return res.json(existing);
  }
  const order = await createOrder({ ...req.body, idempotencyKey });
  res.json(order);
});
```

### RES007 — Message queue missing dead letter queue

**What to look for:** Message consumers that discard failed messages instead of routing them to a dead letter queue.

**Patterns:**
```javascript
consumer.on('message', async (msg) => {
  try {
    await processMessage(msg);
  } catch (err) {
    logger.error('Message processing failed', { error: err });
    // Message is lost
  }
});
```

**Fix:**
```javascript
consumer.on('message', async (msg) => {
  try {
    await processMessage(msg);
  } catch (err) {
    logger.error('Message processing failed', { error: err, messageId: msg.id });
    if (msg.retryCount >= MAX_RETRIES) {
      await deadLetterQueue.send({
        originalMessage: msg,
        error: err.message,
        failedAt: new Date(),
      });
    } else {
      await retryQueue.send({ ...msg, retryCount: msg.retryCount + 1 });
    }
  }
});
```

### RES008 — Multi-service call missing isolation

**What to look for:** Multiple external service calls in the same request without bulkhead isolation. One slow service can exhaust all resources.

**Patterns:**
```javascript
const [users, orders, inventory] = await Promise.all([
  userService.getUsers(),
  orderService.getOrders(),
  inventoryService.getStock(),
]);
// All share the same connection pool / thread pool
```

**Fix:**
```javascript
// Use separate connection pools or thread pools per service
const userPool = new ConnectionPool({ max: 10, service: 'user' });
const orderPool = new ConnectionPool({ max: 10, service: 'order' });
const inventoryPool = new ConnectionPool({ max: 5, service: 'inventory' });

const [users, orders, inventory] = await Promise.all([
  userPool.execute(() => userService.getUsers()),
  orderPool.execute(() => orderService.getOrders()),
  inventoryPool.execute(() => inventoryService.getStock()),
]);
```

### RES009 — Startup connection missing retry

**What to look for:** Application startup that connects to databases or services without retry, causing crashes if the dependency isn't ready yet.

**Patterns:**
```javascript
const db = await mongoose.connect(process.env.MONGODB_URI);
app.listen(3000);  // Crashes if MongoDB isn't ready
```

**Fix:**
```javascript
async function connectWithRetry(uri, maxRetries = 5, delay = 2000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await mongoose.connect(uri);
      logger.info('Connected to MongoDB');
      return;
    } catch (err) {
      logger.warn(`MongoDB connection attempt ${i + 1} failed`, { error: err });
      if (i === maxRetries - 1) throw err;
      await sleep(delay * (i + 1));
    }
  }
}

await connectWithRetry(process.env.MONGODB_URI);
app.listen(3000);
```

### RES010 — Concurrent operation missing limit

**What to look for:** Unbounded concurrency — processing all items in parallel without a concurrency limit.

**Patterns:**
```javascript
const results = await Promise.all(items.map(item => processItem(item)));
// If items has 10,000 entries, this spawns 10,000 concurrent operations
```

**Fix:**
```javascript
async function mapConcurrent(items, fn, concurrency = 10) {
  const results = [];
  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);
    const batchResults = await Promise.all(batch.map(fn));
    results.push(...batchResults);
  }
  return results;
}

const results = await mapConcurrent(items, processItem, 10);
```
