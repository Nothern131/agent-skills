# Resource Management Rules

10 rules that detect resource leaks, mismanagement, and lifecycle violations across all languages.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| RES_M001 | Critical | File handle leak — opened file never closed |
| RES_M002 | High | Network connection leak — socket/connection not released |
| RES_M003 | High | Database connection not returned to pool |
| RES_M004 | Medium | Temporary file not cleaned up |
| RES_M005 | Medium | GDScript node created but never added to scene tree or freed |
| RES_M006 | High | Unbounded cache growth — no eviction policy |
| RES_M007 | Medium | Missing cleanup on process shutdown (graceful exit) |
| RES_M008 | Medium | Configuration loaded without fallback for missing values |
| RES_M009 | High | Large dataset loaded entirely into memory |
| RES_M010 | Medium | Timer/interval not cleared on component destroy |

## Detection Patterns

### RES_M001 — File handle leak

**What to look for:** Files opened but not guaranteed to be closed, especially in error paths.

**Patterns:**
```python
f = open('data.txt', 'r')
content = f.read()
# If exception occurs before close, handle leaks
f.close()
```
```c
FILE *f = fopen(path, "r");
if (!f) return -1;
char buf[256];
fgets(buf, sizeof(buf), f);
// No fclose if error occurs after fopen
return 0;
```
```javascript
const fd = fs.openSync('/path/to/file', 'r');
const data = fs.readFileSync(fd);
// No close if exception occurs
fs.closeSync(fd);
```

**Fix:**
```python
with open('data.txt', 'r') as f:  # Context manager guarantees close
    content = f.read()
```
```c
FILE *f = fopen(path, "r");
if (!f) return -1;
char buf[256];
fgets(buf, sizeof(buf), f);
fclose(f);  // Always close
// Better: use a cleanup attribute or goto cleanup pattern
```
```javascript
const readData = () => {
    const fd = fs.openSync('/path/to/file', 'r');
    try {
        return fs.readFileSync(fd);
    } finally {
        fs.closeSync(fd);
    }
};
```

### RES_M002 — Network connection leak

**What to look for:** Sockets, HTTP connections, or WebSocket connections opened but not properly closed.

**Patterns:**
```python
conn = http.client.HTTPSConnection("api.example.com")
conn.request("GET", "/data")
response = conn.getresponse()
# No conn.close() in error path
```
```go
conn, err := net.Dial("tcp", "host:8080")
if err != nil {
    return err
}
// If subsequent operations fail, conn is never closed
```

**Fix:**
```python
try:
    conn = http.client.HTTPSConnection("api.example.com")
    conn.request("GET", "/data")
    response = conn.getresponse()
    return response.read()
finally:
    conn.close()
```
```go
conn, err := net.Dial("tcp", "host:8080")
if err != nil {
    return err
}
defer conn.Close()  // Guaranteed cleanup
```

### RES_M003 — Database connection not returned to pool

**What to look for:** Database connections acquired from a pool but not released back, especially in error paths.

**Patterns:**
```javascript
const conn = await pool.getConnection();
const result = await conn.query('SELECT ...');
// If query throws, connection is never released
return result;
```
```python
conn = pool.getconn()
cursor = conn.cursor()
cursor.execute("SELECT ...")
# No pool.putconn() if exception occurs
return cursor.fetchall()
```

**Fix:**
```javascript
let conn;
try {
    conn = await pool.getConnection();
    return await conn.query('SELECT ...');
} finally {
    if (conn) conn.release();  // Always return to pool
}
```
```python
conn = pool.getconn()
try:
    cursor = conn.cursor()
    cursor.execute("SELECT ...")
    return cursor.fetchall()
finally:
    pool.putconn(conn)
```

### RES_M004 — Temporary file not cleaned up

**What to look for:** Temporary files created but not deleted after use.

**Patterns:**
```python
import tempfile
f = tempfile.NamedTemporaryFile(delete=False)
f.write(data)
f.close()
# File persists after process exits
process_file(f.name)
```
```c
char path[] = "/tmp/data_XXXXXX";
int fd = mkstemp(path);
write(fd, data, len);
close(fd);
// File not unlinked — persists after use
```

**Fix:**
```python
import tempfile
with tempfile.NamedTemporaryFile(delete=True) as f:  # Auto-deleted on close
    f.write(data)
    f.flush()
    process_file(f.name)
```
```c
char path[] = "/tmp/data_XXXXXX";
int fd = mkstemp(path);
if (fd < 0) return -1;
write(fd, data, len);
close(fd);
process_file(path);
unlink(path);  // Delete after use
```

### RES_M005 — GDScript node created but never added to scene tree or freed

**What to look for:** Nodes instantiated via .instantiate() or .new() that are neither added as children nor explicitly freed.

**Patterns:**
```gdscript
var effect = preload("effect.tscn").instantiate()
effect.position = pos
# Not added to tree, not freed — memory leak
```
```gdscript
var timer = Timer.new()
timer.wait_time = 2.0
# Not add_child(timer), not timer.free()
```

**Fix:**
```gdscript
var effect = preload("effect.tscn").instantiate()
effect.position = pos
add_child(effect)
# Or if temporary:
# effect.queue_free() after use
```
```gdscript
var timer = Timer.new()
timer.wait_time = 2.0
add_child(timer)
timer.one_shot = true
timer.start()
timer.timeout.connect(func(): timer.queue_free())
```

### RES_M006 — Unbounded cache growth

**What to look for:** In-memory caches with no size limit or eviction policy.

**Patterns:**
```python
cache = {}

def get_data(key):
    if key not in cache:
        cache[key] = fetch_from_db(key)  # Grows without bound
    return cache[key]
```
```javascript
const cache = new Map();

function getData(key) {
    if (!cache.has(key)) {
        cache.set(key, fetchFromDb(key));  // No eviction
    }
    return cache.get(key);
}
```

**Fix:**
```python
from functools import lru_cache

@lru_cache(maxsize=1024)  # Bounded with LRU eviction
def get_data(key):
    return fetch_from_db(key)
```
```javascript
class LRUCache {
    constructor(maxSize = 1024) {
        this.maxSize = maxSize;
        this.cache = new Map();
    }
    get(key) {
        if (!this.cache.has(key)) return undefined;
        const value = this.cache.get(key);
        this.cache.delete(key);
        this.cache.set(key, value);  // Move to end (most recent)
        return value;
    }
    set(key, value) {
        if (this.cache.has(key)) this.cache.delete(key);
        this.cache.set(key, value);
        if (this.cache.size > this.maxSize) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);  // Evict oldest
        }
    }
}
```

### RES_M007 — Missing cleanup on process shutdown

**What to look for:** Long-running processes without signal handlers or shutdown hooks to clean up resources.

**Patterns:**
```python
# Server starts but has no shutdown handler
server = create_server()
server.run_forever()
# On SIGTERM: connections dropped, temp files left, data may be corrupted
```
```javascript
// No graceful shutdown
http.createServer(app).listen(3000);
// On SIGTERM: in-flight requests dropped
```

**Fix:**
```python
import signal

server = create_server()

def shutdown(signum, frame):
    logger.info("Shutting down gracefully...")
    server.stop()
    cleanup_temp_files()
    flush_buffers()
    sys.exit(0)

signal.signal(signal.SIGTERM, shutdown)
signal.signal(signal.SIGINT, shutdown)
server.run_forever()
```
```javascript
const server = http.createServer(app);
server.listen(3000);

function gracefulShutdown() {
    logger.info('Shutting down gracefully...');
    server.close(() => {
        cleanupResources();
        process.exit(0);
    });
    setTimeout(() => process.exit(1), 10000);  // Force exit after 10s
}

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);
```

### RES_M008 — Configuration loaded without fallback

**What to look for:** Configuration values read without default values or validation, causing crashes when config is missing.

**Patterns:**
```python
db_host = config['database']['host']  # KeyError if missing
port = int(config['port'])  # ValueError if not a number
```
```javascript
const dbHost = config.database.host;  // TypeError if database undefined
const port = parseInt(config.port);   // NaN if missing
```

**Fix:**
```python
db_host = config.get('database', {}).get('host', 'localhost')
port = int(config.get('port', 5432))
```
```javascript
const dbHost = config?.database?.host ?? 'localhost';
const port = parseInt(config?.port ?? '5432', 10);
if (isNaN(port)) throw new ConfigError('Invalid port value');
```

### RES_M009 — Large dataset loaded entirely into memory

**What to look for:** Reading entire files or query results into memory without streaming or pagination.

**Patterns:**
```python
rows = cursor.execute("SELECT * FROM large_table").fetchall()  # All in memory
```
```javascript
const data = fs.readFileSync('huge_file.json');  # All in memory
const parsed = JSON.parse(data);
```
```python
with open('large.csv') as f:
    all_lines = f.readlines()  # All in memory
```

**Fix:**
```python
cursor.execute("SELECT * FROM large_table")
while True:
    rows = cursor.fetchmany(1000)  # Paginated
    if not rows:
        break
    process_batch(rows)
```
```javascript
// Use streaming JSON parser
import { createReadStream } from 'fs';
import { pipeline } from 'stream';
import JSONStream from 'JSONStream';

pipeline(
    createReadStream('huge_file.json'),
    JSONStream.parse('items.*'),
    processItem,
    callback
);
```
```python
with open('large.csv') as f:
    for line in f:  # Stream line by line
        process_line(line)
```

### RES_M010 — Timer/interval not cleared on component destroy

**What to look for:** setInterval/setTimeout or recurring timers that are not cleared when the component is destroyed.

**Patterns:**
```javascript
class Dashboard {
    constructor() {
        this.timer = setInterval(() => this.refresh(), 5000);
    }
    destroy() {
        // Timer not cleared — keeps firing after component is destroyed
    }
}
```
```gdscript
func _ready():
    var timer = Timer.new()
    timer.wait_time = 1.0
    add_child(timer)
    timer.start()
    # If node is freed, timer may still fire

func _exit_tree():
    # Timer not stopped
    pass
```

**Fix:**
```javascript
class Dashboard {
    constructor() {
        this.timer = setInterval(() => this.refresh(), 5000);
    }
    destroy() {
        clearInterval(this.timer);
        this.timer = null;
    }
}
```
```gdscript
var _refresh_timer: Timer

func _ready():
    _refresh_timer = Timer.new()
    _refresh_timer.wait_time = 1.0
    add_child(_refresh_timer)
    _refresh_timer.start()

func _exit_tree():
    if _refresh_timer:
        _refresh_timer.stop()
        _refresh_timer.queue_free()
```
