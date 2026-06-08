# Error Handling Rules

10 rules that detect missing or improper error handling in code.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| ERR001 | Critical | Missing try-catch error handling |
| ERR002 | High | Empty catch/except block swallows errors |
| ERR003 | High | Promise/await missing error handling |
| ERR004 | Medium | Caught error neither rethrown nor logged |
| ERR005 | Medium | Throwing non-Error/Exception object |
| ERR006 | Medium | Resource operation missing finally cleanup |
| ERR007 | Low | catch block only console.log without handling |
| ERR008 | Medium | Function parameters missing input validation |
| ERR009 | Medium | Bare except catching all exceptions (Python) |
| ERR010 | High | Critical operation missing fallback mechanism |

## Detection Patterns

### ERR001 — Missing try-catch error handling

**What to look for:** I/O operations, network calls, file access, or database queries that are not wrapped in error handling.

**Patterns:**
```javascript
// Violation: no try-catch around file read
const data = fs.readFileSync('/path/to/file');
```
```python
# Violation: no try-except around network call
response = requests.get('https://api.example.com/data')
```

**Fix:**
```javascript
try {
  const data = fs.readFileSync('/path/to/file');
} catch (err) {
  logger.error('Failed to read file', { path: '/path/to/file', error: err });
  throw new AppError('FILE_READ_FAILED', err);
}
```
```python
try:
    response = requests.get('https://api.example.com/data')
except requests.RequestException as e:
    logger.error(f"API call failed: {e}")
    raise AppError("API_CALL_FAILED", e)
```

### ERR002 — Empty catch/except block

**What to look for:** catch or except blocks with empty bodies or only comments.

**Patterns:**
```javascript
try {
  doSomething();
} catch (e) {
  // silent
}
```
```python
try:
    do_something()
except Exception:
    pass
```

**Fix:**
```javascript
try {
  doSomething();
} catch (e) {
  logger.warn('Non-critical operation failed', { error: e.message });
  // Explicitly document why we're swallowing this error
}
```
```python
except Exception as e:
    logger.warning(f"Non-critical operation failed: {e}")
    # Explicitly document why we're swallowing this error
```

### ERR003 — Promise/await missing error handling

**What to look for:** await expressions without surrounding try-catch, or .then() chains without .catch().

**Patterns:**
```javascript
const result = await fetchData();  // No try-catch
```
```javascript
fetch('/api/data').then(res => res.json());  // No .catch()
```

**Fix:**
```javascript
try {
  const result = await fetchData();
} catch (err) {
  logger.error('Data fetch failed', { error: err });
  throw new ServiceError('FETCH_FAILED', err);
}
```
```javascript
fetch('/api/data')
  .then(res => res.json())
  .catch(err => {
    logger.error('Fetch failed', { error: err });
  });
```

### ERR004 — Caught error neither rethrown nor logged

**What to look for:** catch blocks that catch an error but don't log it, rethrow it, or handle it meaningfully.

**Patterns:**
```javascript
catch (e) {
  setErrorState(true);  // Sets state but doesn't log or rethrow
}
```

**Fix:**
```javascript
catch (e) {
  logger.error('Operation failed', { error: e });
  setErrorState(true);
}
```

### ERR005 — Throwing non-Error object

**What to look for:** throw statements with strings, numbers, or plain objects instead of Error instances.

**Patterns:**
```javascript
throw 'Something went wrong';
throw { code: 500, message: 'fail' };
```
```python
raise "something went wrong"  # Not an Exception subclass
```

**Fix:**
```javascript
throw new Error('Something went wrong');
throw new AppError('OPERATION_FAILED', { code: 500 });
```

### ERR006 — Resource operation missing finally cleanup

**What to look for:** file handles, database connections, or network sockets opened but not guaranteed to close.

**Patterns:**
```javascript
const conn = await pool.getConnection();
const result = await conn.query('SELECT ...');
// No finally to release connection
```
```python
f = open('data.txt', 'r')
content = f.read()
# No finally/close
```

**Fix:**
```javascript
let conn;
try {
  conn = await pool.getConnection();
  return await conn.query('SELECT ...');
} finally {
  if (conn) conn.release();
}
```
```python
with open('data.txt', 'r') as f:
    content = f.read()
```

### ERR007 — catch block only console.log

**What to look for:** catch blocks that only call console.log/console.error without any recovery or propagation.

**Patterns:**
```javascript
catch (e) {
  console.log(e);
}
```

**Fix:**
```javascript
catch (e) {
  logger.error('Operation failed', { error: e, context: { userId } });
  notifyMonitoring(e);
  throw new AppError('OPERATION_FAILED', e);
}
```

### ERR008 — Function parameters missing input validation

**What to look for:** public-facing functions that accept parameters without validating type, range, or format.

**Patterns:**
```javascript
function createUser(name, email, age) {
  db.insert({ name, email, age });  // No validation
}
```

**Fix:**
```javascript
function createUser(name, email, age) {
  if (!name || typeof name !== 'string') throw new ValidationError('Invalid name');
  if (!isValidEmail(email)) throw new ValidationError('Invalid email');
  if (age < 0 || age > 150) throw new ValidationError('Invalid age');
  db.insert({ name, email, age });
}
```

### ERR009 — Bare except catching all exceptions

**What to look for:** Python `except:` or `except Exception:` that catches everything including SystemExit and KeyboardInterrupt.

**Patterns:**
```python
try:
    do_something()
except:  # Catches everything including SystemExit
    pass
```

**Fix:**
```python
try:
    do_something()
except (ValueError, IOError) as e:
    logger.warning(f"Expected failure: {e}")
```

### ERR010 — Critical operation missing fallback

**What to look for:** payment processing, data persistence, or other critical operations with no alternative path on failure.

**Patterns:**
```javascript
await paymentService.charge(amount);  // No fallback if service is down
```

**Fix:**
```javascript
try {
  await paymentService.charge(amount);
} catch (err) {
  logger.error('Primary payment failed, trying fallback', { error: err });
  await paymentQueue.enqueue({ amount, retryAt: Date.now() + 60000 });
  notifyUser('Payment processing, will retry shortly');
}
```
