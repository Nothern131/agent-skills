# Concurrency Safety Rules

10 rules that detect concurrency bugs and thread-safety violations across all languages.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| CON001 | Critical | Shared mutable state without synchronization |
| CON002 | High | Race condition — check-then-act pattern without lock |
| CON003 | High | Deadlock risk — acquiring multiple locks in inconsistent order |
| CON004 | High | Async callback capturing stale reference |
| CON005 | Medium | Atomic operation split into non-atomic read-modify-write |
| CON006 | Medium | Thread-unsafe collection used in concurrent context |
| CON007 | Medium | Missing volatile/atomic for flag shared between threads |
| CON008 | High | GDScript deferred call to freed object |
| CON009 | Medium | Lock held during blocking I/O operation |
| CON010 | Medium | Goroutine/coroutine leak — never-terminating concurrent task |

## Detection Patterns

### CON001 — Shared mutable state without synchronization

**What to look for:** Global or instance variables accessed from multiple threads/coroutines without locks or atomic operations.

**Patterns:**
```python
counter = 0  # Shared across threads

def increment():
    global counter
    counter += 1  # Not atomic: read-modify-write without lock
```
```javascript
let sharedState = {};  // Shared across async contexts

async function updateState(key, value) {
    sharedState[key] = value;  // No synchronization
}
```
```cpp
int counter = 0;  // Shared across threads

void increment() {
    counter++;  // Not atomic
}
```

**Fix:**
```python
import threading

counter = 0
counter_lock = threading.Lock()

def increment():
    global counter
    with counter_lock:
        counter += 1
```
```javascript
// Use atomic operations or single-writer pattern
const state = new Map();

async function updateState(key, value) {
    // Single-threaded event loop ensures atomicity for simple ops
    // For complex read-modify-write, use a mutex or queue
    state.set(key, value);
}
```
```cpp
std::atomic<int> counter{0};

void increment() {
    counter.fetch_add(1, std::memory_order_relaxed);
}
```

### CON002 — Race condition — check-then-act without lock

**What to look for:** Code that checks a condition and then acts on it, where the condition can change between check and act.

**Patterns:**
```python
if not file_exists(path):  # Check
    create_file(path)       # Act — another thread might create it first
```
```javascript
if (!cache.has(key)) {     // Check
    await fetchAndCache(key);  // Act — another request might fetch first
}
```
```cpp
if (ptr == nullptr) {      // Check
    ptr = new Object();     // Act — another thread might allocate first
}
```

**Fix:**
```python
with file_lock:
    if not file_exists(path):
        create_file(path)
```
```javascript
const pending = new Map();

async function getOrFetch(key) {
    if (cache.has(key)) return cache.get(key);
    if (pending.has(key)) return pending.get(key);

    const promise = fetchAndCache(key);
    pending.set(key, promise);
    try {
        return await promise;
    } finally {
        pending.delete(key);
    }
}
```
```cpp
std::mutex mtx;
std::lock_guard<std::mutex> lock(mtx);
if (ptr == nullptr) {
    ptr = new Object();
}
```

### CON003 — Deadlock risk — inconsistent lock order

**What to look for:** Multiple locks acquired in different orders in different code paths.

**Patterns:**
```python
# Path 1: lock A then B
with lock_a:
    with lock_b:
        process()

# Path 2: lock B then A (DEADLOCK!)
with lock_b:
    with lock_a:
        process_other()
```
```cpp
// Path 1
std::lock_guard<std::mutex> lk1(mutex_a);
std::lock_guard<std::mutex> lk2(mutex_b);

// Path 2 (DEADLOCK!)
std::lock_guard<std::mutex> lk1(mutex_b);
std::lock_guard<std::mutex> lk2(mutex_a);
```

**Fix:**
```python
# Always acquire locks in the same global order
with lock_a:        # Always A first
    with lock_b:    # Then B
        process()
```
```cpp
// Use std::lock for deadlock-free multi-lock acquisition
std::unique_lock<std::mutex> lk1(mutex_a, std::defer_lock);
std::unique_lock<std::mutex> lk2(mutex_b, std::defer_lock);
std::lock(lk1, lk2);  // Deadlock-free acquisition
```

### CON004 — Async callback capturing stale reference

**What to look for:** Closures or callbacks that capture references to variables that may change before the callback executes.

**Patterns:**
```javascript
for (var i = 0; i < 10; i++) {
    setTimeout(() => console.log(i), 100);  // All print 10
}
```
```gdscript
var target_node = get_node("Player")
get_tree().create_timer(2.0).timeout.connect(func():
    target_node.position = Vector2.ZERO  # target_node may be freed
)
```

**Fix:**
```javascript
for (let i = 0; i < 10; i++) {  // let creates block scope
    setTimeout(() => console.log(i), 100);  // Each captures own i
}
```
```gdscript
var target_node = get_node("Player")
get_tree().create_timer(2.0).timeout.connect(func():
    if not is_instance_valid(target_node):
        return
    target_node.position = Vector2.ZERO
)
```

### CON005 — Atomic operation split into non-atomic read-modify-write

**What to look for:** Operations that should be atomic but are implemented as separate read, modify, and write steps.

**Patterns:**
```python
balance = account.balance  # Read
balance -= amount           # Modify
account.balance = balance   # Write — not atomic
```
```cpp
int val = shared_counter.load();  // Read
val += 1;                          // Modify
shared_counter.store(val);         // Write — not atomic
```

**Fix:**
```python
with account.lock:
    account.balance -= amount  # Atomic under lock
```
```cpp
shared_counter.fetch_add(1, std::memory_order_relaxed);  // Single atomic op
```

### CON006 — Thread-unsafe collection used in concurrent context

**What to look for:** Standard collections (dict, list, HashMap, std::vector) accessed from multiple threads without external synchronization.

**Patterns:**
```python
results = []  # Shared list

def worker(item):
    results.append(process(item))  # Not thread-safe
```
```cpp
std::vector<int> results;  # Shared vector

void worker(int item) {
    results.push_back(process(item));  // Not thread-safe
}
```

**Fix:**
```python
import threading
results = []
results_lock = threading.Lock()

def worker(item):
    result = process(item)
    with results_lock:
        results.append(result)
```
```cpp
std::mutex results_mtx;
std::vector<int> results;

void worker(int item) {
    int result = process(item);
    std::lock_guard<std::mutex> lock(results_mtx);
    results.push_back(result);
}
```

### CON007 — Missing volatile/atomic for shared flag

**What to look for:** Boolean flags shared between threads without atomic or volatile semantics, causing the compiler to cache the value.

**Patterns:**
```cpp
bool running = true;  // Compiler may cache in register

void worker() {
    while (running) {  // May never see running = false
        do_work();
    }
}
```
```python
running = True  # GIL protects in CPython, but not guaranteed

def worker():
    while running:  # May not see updates from other threads
        do_work()
```

**Fix:**
```cpp
std::atomic<bool> running{true};

void worker() {
    while (running.load(std::memory_order_acquire)) {
        do_work();
    }
}
```
```python
import threading
running = threading.Event()

def worker():
    while running.is_set():
        do_work()
```

### CON008 — GDScript deferred call to freed object

**What to look for:** call_deferred or Timer callbacks that reference nodes which may have been freed before the callback fires.

**Patterns:**
```gdscript
func _on_button_pressed():
    var enemy = get_node("Enemy")
    enemy.queue_free()
    # Later deferred call still references enemy
    get_tree().create_timer(1.0).timeout.connect(func():
        enemy.play_death_sound()  # Enemy already freed!
    )
```

**Fix:**
```gdscript
func _on_button_pressed():
    var enemy = get_node("Enemy")
    enemy.queue_free()
    # Play sound before freeing, or use a separate audio node
    AudioManager.play_sound("death")
    # Or check validity in callback:
    # if is_instance_valid(enemy): enemy.play_death_sound()
```

### CON009 — Lock held during blocking I/O

**What to look for:** Holding a lock while performing network calls, file I/O, or other blocking operations, which reduces concurrency.

**Patterns:**
```python
with data_lock:
    data = prepare_data()
    response = requests.post(url, json=data)  # Blocking I/O under lock
    process_response(response)
```
```cpp
std::lock_guard<std::mutex> lock(data_mtx);
auto data = prepare_data();
auto response = http_post(url, data);  // Blocking I/O under lock
process_response(response);
```

**Fix:**
```python
with data_lock:
    data = prepare_data()
# Lock released before I/O
response = requests.post(url, json=data)
process_response(response)
```
```cpp
Data data;
{
    std::lock_guard<std::mutex> lock(data_mtx);
    data = prepare_data();
}  // Lock released before I/O
auto response = http_post(url, data);
process_response(response);
```

### CON010 — Goroutine/coroutine leak

**What to look for:** Launched goroutines, coroutines, or threads that have no guaranteed termination condition.

**Patterns:**
```go
go func() {
    for {
        doWork()  // No exit condition, no context cancellation
    }
}()
```
```python
async def background_task():
    while True:
        await process()  # No cancellation mechanism
```

**Fix:**
```go
ctx, cancel := context.WithCancel(parentCtx)
defer cancel()

go func() {
    for {
        select {
        case <-ctx.Done():
            return
        default:
            doWork()
        }
    }
}()
```
```python
async def background_task(cancellation_event):
    while not cancellation_event.is_set():
        await process()
```
