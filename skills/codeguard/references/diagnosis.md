# Diagnostics Rules

10 rules that detect missing observability and diagnostic capabilities.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| DIAG001 | High | Code missing logging |
| DIAG002 | Medium | Error missing error code |
| DIAG003 | Medium | Log missing context information |
| DIAG004 | Medium | API request missing trace ID |
| DIAG005 | Low | Service missing health check endpoint |
| DIAG006 | Medium | Critical business path missing metrics |
| DIAG007 | Low | Error log not including exception object |
| DIAG008 | Low | Missing debug mode control |
| DIAG009 | Medium | Severe error missing alert mechanism |
| DIAG010 | Medium | Sensitive operation missing audit log |

## Detection Patterns

### DIAG001 — Code missing logging

**What to look for:** Functions or modules with no logging statements, especially in critical paths.

**Patterns:**
```javascript
async function processPayment(amount, cardInfo) {
  const charge = await stripe.charges.create({ amount, source: cardInfo.token });
  await db.query('INSERT INTO payments ...', [charge.id, amount]);
  return charge;
}
// No logging at all
```

**Fix:**
```javascript
async function processPayment(amount, cardInfo) {
  logger.info('Processing payment', { amount, cardLast4: cardInfo.last4 });
  try {
    const charge = await stripe.charges.create({ amount, source: cardInfo.token });
    logger.info('Payment successful', { chargeId: charge.id, amount });
    await db.query('INSERT INTO payments ...', [charge.id, amount]);
    return charge;
  } catch (err) {
    logger.error('Payment failed', { amount, error: err });
    throw err;
  }
}
```

### DIAG002 — Error missing error code

**What to look for:** Errors thrown with only a message string, making them hard to identify programmatically.

**Patterns:**
```javascript
throw new Error('User not found');
throw new Error('Payment failed');
```
```python
raise ValueError("User not found")
raise RuntimeError("Payment failed")
```

**Fix:**
```javascript
throw new AppError('USER_NOT_FOUND', { userId });
throw new AppError('PAYMENT_FAILED', { amount, reason: err.message });
```
```python
raise AppError("USER_NOT_FOUND", user_id=user_id)
raise AppError("PAYMENT_FAILED", amount=amount, reason=str(err))
```

### DIAG003 — Log missing context information

**What to look for:** Log statements with only a message, no structured context (user ID, request ID, resource ID).

**Patterns:**
```javascript
logger.info('User logged in');
logger.error('Database query failed');
```

**Fix:**
```javascript
logger.info('User logged in', { userId: user.id, ip: req.ip, method: 'password' });
logger.error('Database query failed', { query: 'INSERT orders', error: err, retryCount });
```

### DIAG004 — API request missing trace ID

**What to look for:** HTTP handlers that don't generate or propagate a trace/correlation ID for request tracking.

**Patterns:**
```javascript
app.get('/api/users/:id', (req, res) => {
  // No trace ID in logs or response headers
  const user = getUser(req.params.id);
  res.json(user);
});
```

**Fix:**
```javascript
app.use((req, res, next) => {
  req.traceId = req.headers['x-trace-id'] || crypto.randomUUID();
  res.setHeader('X-Trace-Id', req.traceId);
  next();
});

app.get('/api/users/:id', (req, res) => {
  logger.info('Fetching user', { traceId: req.traceId, userId: req.params.id });
  const user = getUser(req.params.id);
  res.json(user);
});
```

### DIAG005 — Service missing health check endpoint

**What to look for:** Backend services without a /health or /readiness endpoint.

**Patterns:**
```javascript
// No health check route defined
app.listen(3000);
```

**Fix:**
```javascript
app.get('/health', (req, res) => {
  res.json({ status: 'ok', uptime: process.uptime(), timestamp: new Date().toISOString() });
});

app.get('/readiness', async (req, res) => {
  try {
    await db.query('SELECT 1');
    res.json({ status: 'ready' });
  } catch (err) {
    res.status(503).json({ status: 'not_ready', reason: err.message });
  }
});
```

### DIAG006 — Critical business path missing metrics

**What to look for:** Key business operations (payments, signups, order processing) without metrics/counters.

**Patterns:**
```javascript
async function processOrder(order) {
  await validateOrder(order);
  await chargeCustomer(order);
  await shipOrder(order);
}
// No metrics collected
```

**Fix:**
```javascript
async function processOrder(order) {
  const startTime = Date.now();
  try {
    await validateOrder(order);
    await chargeCustomer(order);
    await shipOrder(order);
    metrics.increment('orders.processed.success');
    metrics.histogram('orders.processing_time', Date.now() - startTime);
  } catch (err) {
    metrics.increment('orders.processed.failure', { reason: err.code });
    throw err;
  }
}
```

### DIAG007 — Error log not including exception object

**What to look for:** Error log statements that log a message but not the actual exception object, losing stack trace information.

**Patterns:**
```javascript
catch (err) {
  logger.error('Operation failed: ' + err.message);  // No stack trace
}
```

**Fix:**
```javascript
catch (err) {
  logger.error('Operation failed', { error: err, context: { orderId } });
  // Structured logging preserves the full error object including stack trace
}
```

### DIAG008 — Missing debug mode control

**What to look for:** Verbose logging or debug features that are always on, with no toggle.

**Patterns:**
```javascript
console.log('Request body:', req.body);  // Always logs
app.use(morgan('dev'));  // Always verbose
```

**Fix:**
```javascript
if (config.debug) {
  logger.debug('Request body', { body: req.body });
}
// Use log levels: debug in dev, info in production
logger.setLevel(config.env === 'production' ? 'info' : 'debug');
```

### DIAG009 — Severe error missing alert mechanism

**What to look for:** Critical errors (payment failure, data loss, service down) logged but not triggering alerts.

**Patterns:**
```javascript
catch (err) {
  logger.error('Payment gateway unreachable', { error: err });
  // No alert sent
}
```

**Fix:**
```javascript
catch (err) {
  logger.error('Payment gateway unreachable', { error: err });
  alerting.send({
    level: 'critical',
    title: 'Payment Gateway Down',
    description: err.message,
    runbook: 'https://wiki/runbooks/payment-gateway-down',
  });
}
```

### DIAG010 — Sensitive operation missing audit log

**What to look for:** Data modifications, permission changes, or admin actions without an audit trail.

**Patterns:**
```javascript
async function deleteUser(userId) {
  await db.query('DELETE FROM users WHERE id = ?', [userId]);
  // No audit record
}
```

**Fix:**
```javascript
async function deleteUser(userId, adminId) {
  await db.query('DELETE FROM users WHERE id = ?', [userId]);
  await auditLog.record({
    action: 'USER_DELETED',
    actor: adminId,
    target: userId,
    timestamp: new Date(),
    metadata: { ip: req.ip },
  });
}
```
