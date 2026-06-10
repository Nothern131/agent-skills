# Security / Permissions Rules

10 rules that detect security vulnerabilities and permission issues across all languages (Python, JavaScript/TypeScript, C/C++, GDScript, Go).

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| PERM001 | Critical | Hardcoded passwords/keys/tokens |
| PERM002 | Critical | SQL injection risk |
| PERM003 | Critical | Command injection risk |
| PERM004 | High | Path traversal risk |
| PERM005 | High | Route missing authentication check |
| PERM006 | High | Insecure random number in security context |
| PERM007 | High | Admin operation missing permission control |
| PERM008 | Critical | Using eval() to execute dynamic code |
| PERM009 | Medium | POST request missing CSRF protection |
| PERM010 | Medium | Debug information exposed in production |

## Detection Patterns

### PERM001 — Hardcoded passwords/keys/tokens

**What to look for:** Literal strings containing passwords, API keys, secrets, or tokens directly in source code.

**Patterns:**
```javascript
const API_KEY = 'sk-abc123def456';
const dbPassword = 'mysecretpassword';
password = 'admin123';
```
```python
API_KEY = "sk-abc123def456"
DB_PASSWORD = "mysecretpassword"
```

**Fix:**
```javascript
const API_KEY = process.env.API_KEY;
const dbPassword = process.env.DB_PASSWORD;
```
```python
import os
API_KEY = os.environ["API_KEY"]
DB_PASSWORD = os.environ["DB_PASSWORD"]
```
```c
// C: Read from environment or config file
const char *api_key = getenv("API_KEY");
if (!api_key) {
    fprintf(stderr, "API_KEY not set\n");
    return EXIT_FAILURE;
}
```
```cpp
// C++: Use config abstraction
auto apiKey = Config::get("API_KEY");  // Reads from env/config
```

### PERM002 — SQL injection risk

**What to look for:** SQL queries built by string concatenation or f-strings with user input.

**Patterns:**
```javascript
const query = `SELECT * FROM users WHERE id = ${userId}`;
db.query(`SELECT * FROM users WHERE name = '${name}'`);
```
```python
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute("SELECT * FROM users WHERE name = '" + name + "'")
```

**Fix:**
```javascript
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
```
```python
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

### PERM003 — Command injection risk

**What to look for:** Shell commands constructed with user input, especially via exec, system, or subprocess with shell=True.

**Patterns:**
```javascript
exec(`rm -rf ${userInput}`);
child_process.exec(`ls ${dirName}`);
```
```python
os.system(f"rm -rf {user_input}")
subprocess.run(f"ls {dir_name}", shell=True)
```

**Fix:**
```javascript
const { execFile } = require('child_process');
execFile('rm', ['-rf', sanitizePath(userInput)]);
```
```python
subprocess.run(["rm", "-rf", sanitize_path(user_input)], shell=False)
```
```c
// C: Use execvp with argument array, never system()
char *args[] = {"rm", "-rf", sanitized_path, NULL};
pid_t pid = fork();
if (pid == 0) {
    execvp("rm", args);
    _exit(EXIT_FAILURE);
}
```

### PERM004 — Path traversal risk

**What to look for:** File paths constructed from user input without sanitization, allowing `../` sequences.

**Patterns:**
```javascript
const filePath = path.join('/uploads', req.params.filename);
fs.readFile(filePath);
```
```python
file_path = os.path.join("/uploads", request.args.get("filename"))
```

**Fix:**
```javascript
const filename = path.basename(req.params.filename);
const filePath = path.join('/uploads', filename);
if (!filePath.startsWith('/uploads/')) throw new Error('Invalid path');
```
```python
filename = os.path.basename(request.args.get("filename"))
file_path = os.path.join("/uploads", filename)
if not file_path.startswith("/uploads/"):
    raise ValueError("Invalid path")
```

### PERM005 — Route missing authentication check

**What to look for:** API routes that perform sensitive operations but have no auth middleware or token validation.

**Patterns:**
```javascript
app.delete('/api/users/:id', (req, res) => {
  // No auth check
  deleteUser(req.params.id);
});
```

**Fix:**
```javascript
app.delete('/api/users/:id', authenticate, authorize('admin'), (req, res) => {
  deleteUser(req.params.id);
});
```

### PERM006 — Insecure random number in security context

**What to look for:** Math.random(), random.random(), or similar non-cryptographic RNG used for tokens, passwords, or session IDs.

**Patterns:**
```javascript
const token = Math.random().toString(36).substring(2);
```
```python
import random
token = ''.join(random.choices(string.ascii_letters, k=32))
```

**Fix:**
```javascript
const token = crypto.randomBytes(32).toString('hex');
```
```python
import secrets
token = secrets.token_urlsafe(32)
```

### PERM007 — Admin operation missing permission control

**What to look for:** Endpoints that perform administrative actions (delete all, modify config, manage users) without role/permission checks.

**Patterns:**
```javascript
app.post('/api/admin/config', (req, res) => {
  updateConfig(req.body);  // No role check
});
```

**Fix:**
```javascript
app.post('/api/admin/config', authenticate, requireRole('admin'), (req, res) => {
  updateConfig(req.body);
});
```

### PERM008 — Using eval() to execute dynamic code

**What to look for:** eval(), Function(), exec(), or similar calls with user-controllable input.

**Patterns:**
```javascript
eval(userInput);
new Function('return ' + codeString)();
```
```python
exec(user_input)
eval(user_input)
```

**Fix:**
```javascript
// Use a safe parser/interpreter for the specific DSL
const result = safeJsonParser.parse(userInput);
```
```python
# Use ast.literal_eval for safe evaluation of literals
import ast
result = ast.literal_eval(user_input)
```

### PERM009 — POST request missing CSRF protection

**What to look for:** State-changing POST/PUT/DELETE endpoints that don't validate CSRF tokens.

**Patterns:**
```javascript
app.post('/api/transfer', (req, res) => {
  transferMoney(req.body);  // No CSRF token check
});
```

**Fix:**
```javascript
app.post('/api/transfer', csrfProtection, (req, res) => {
  transferMoney(req.body);
});
```

### PERM010 — Debug information exposed in production

**What to look for:** Stack traces, error details, or debug mode enabled in production configuration.

**Patterns:**
```javascript
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.stack });  // Exposes stack trace
});
```
```python
app.config["DEBUG"] = True  # In production
```

**Fix:**
```javascript
app.use((err, req, res, next) => {
  logger.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});
```
```python
app.config["DEBUG"] = os.environ.get("FLASK_DEBUG", "false").lower() == "true"
```
