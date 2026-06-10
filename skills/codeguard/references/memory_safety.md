# Memory Safety Rules

10 rules that detect memory safety violations in C, C++, and systems-level code.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| MEM001 | Critical | Buffer overflow — array/buffer access without bounds check |
| MEM002 | Critical | Use-after-free — accessing memory after free/delete |
| MEM003 | Critical | Memory leak — allocated memory never freed |
| MEM004 | High | Null pointer dereference — pointer used without null check |
| MEM005 | High | Double free — same memory freed more than once |
| MEM006 | High | Uninitialized variable — variable used before assignment |
| MEM007 | Medium | Dangling pointer — pointer to stack/local variable returned |
| MEM008 | Medium | Integer overflow in allocation size calculation |
| MEM009 | Medium | Missing RAII guard for resource acquisition (C++) |
| MEM010 | High | Unsafe type cast — reinterpret_cast/C-style cast violating strict aliasing |

## Detection Patterns

### MEM001 — Buffer overflow

**What to look for:** Array indexing, memcpy, strcpy, sprintf, or buffer writes without bounds verification.

**Patterns:**
```c
// Violation: no bounds check
void copy_name(char *dest, const char *src) {
    strcpy(dest, src);  // No length check
}
```
```c
// Violation: array index not validated
int get_element(int *arr, int index) {
    return arr[index];  // No bounds check
}
```

**Fix:**
```c
void copy_name(char *dest, size_t dest_size, const char *src) {
    if (strlen(src) >= dest_size) {
        logger_error("Buffer overflow prevented: src length %zu >= dest size %zu",
                     strlen(src), dest_size);
        dest[0] = '\0';
        return;
    }
    strncpy(dest, src, dest_size - 1);
    dest[dest_size - 1] = '\0';
}
```
```c
int get_element(int *arr, int arr_len, int index) {
    if (index < 0 || index >= arr_len) {
        logger_error("Index out of bounds: %d (len=%d)", index, arr_len);
        return -1;
    }
    return arr[index];
}
```

### MEM002 — Use-after-free

**What to look for:** Pointers or references used after the memory they point to has been freed or deleted.

**Patterns:**
```c
free(buffer);
process(buffer);  // Use after free
```
```cpp
delete obj;
obj->method();  // Use after delete
```

**Fix:**
```c
free(buffer);
buffer = NULL;  // Nullify immediately after free
// process(buffer) would now safely crash (detectable) instead of silent corruption
```
```cpp
delete obj;
obj = nullptr;
```

### MEM003 — Memory leak

**What to look for:** malloc/calloc/new without corresponding free/delete, especially in error paths.

**Patterns:**
```c
char *buffer = malloc(1024);
if (!buffer) return -1;
FILE *f = fopen(path, "r");
if (!f) return -1;  // Leak: buffer not freed on error path
```
```cpp
Widget *w = new Widget();
if (!init_success) return false;  // Leak: w not deleted
```

**Fix:**
```c
char *buffer = malloc(1024);
if (!buffer) return -1;
FILE *f = fopen(path, "r");
if (!f) {
    free(buffer);  // Free on error path
    return -1;
}
// ... use both ...
free(buffer);
fclose(f);
```
```cpp
auto w = std::make_unique<Widget>();  // RAII: auto-freed
if (!init_success) return false;  // No leak
```

### MEM004 — Null pointer dereference

**What to look for:** Pointer dereference without prior null check, especially for return values of malloc, fopen, or function parameters.

**Patterns:**
```c
void process(User *user) {
    printf("%s\n", user->name);  // No null check
}
```
```c
char *data = malloc(size);
data[0] = 'x';  // malloc can return NULL
```

**Fix:**
```c
void process(User *user) {
    if (!user) {
        logger_error("Null user pointer in process()");
        return;
    }
    printf("%s\n", user->name);
}
```
```c
char *data = malloc(size);
if (!data) {
    logger_error("Allocation failed for size %zu", size);
    return -1;
}
data[0] = 'x';
```

### MEM005 — Double free

**What to look for:** The same pointer passed to free/delete more than once, often in complex error handling paths.

**Patterns:**
```c
void cleanup(char *buf1, char *buf2) {
    free(buf1);
    free(buf2);
    free(buf1);  // Double free
}
```

**Fix:**
```c
void cleanup(char *buf1, char *buf2) {
    free(buf1);
    buf1 = NULL;
    free(buf2);
    buf2 = NULL;
    // Even if called again, free(NULL) is safe
}
```

### MEM006 — Uninitialized variable

**What to look for:** Local variables used before being assigned a value.

**Patterns:**
```c
int result;  // Uninitialized
if (condition) {
    result = compute();
}
return result;  // May return garbage if condition is false
```
```cpp
int count;  // Uninitialized
for (auto &item : items) {
    count++;  // Incrementing garbage
}
```

**Fix:**
```c
int result = 0;  // Initialize with default
if (condition) {
    result = compute();
}
return result;
```
```cpp
int count = 0;  // Initialize
for (auto &item : items) {
    count++;
}
```

### MEM007 — Dangling pointer

**What to look for:** Functions returning pointers to local/stack variables, or storing pointers to objects that have gone out of scope.

**Patterns:**
```c
int *get_value() {
    int value = 42;
    return &value;  // Dangling: stack variable
}
```
```cpp
std::string_view get_name() {
    std::string name = compute_name();
    return name;  // Dangling: name destroyed after scope exit
}
```

**Fix:**
```c
int *get_value() {
    int *value = malloc(sizeof(int));
    if (!value) return NULL;
    *value = 42;
    return value;  // Caller must free
}
```
```cpp
std::string get_name() {  // Return by value
    std::string name = compute_name();
    return name;  // Safe: moved/copied
}
```

### MEM008 — Integer overflow in allocation size

**What to look for:** Size calculations for malloc/new that can overflow, especially when multiplying dimensions.

**Patterns:**
```c
int *arr = malloc(count * sizeof(int));  // count * sizeof(int) can overflow
```
```c
size_t total = width * height * channels;  // Multiplication can overflow
pixel_t *img = malloc(total);
```

**Fix:**
```c
if (count > SIZE_MAX / sizeof(int)) {
    logger_error("Allocation size overflow: count=%zu", count);
    return NULL;
}
int *arr = malloc(count * sizeof(int));
```
```c
size_t total;
if (__builtin_mul_overflow(width, height, &total) ||
    __builtin_mul_overflow(total, channels, &total)) {
    logger_error("Image size overflow: %zux%zux%zu", width, height, channels);
    return NULL;
}
pixel_t *img = malloc(total);
```

### MEM009 — Missing RAII guard for resource acquisition

**What to look for:** C++ code acquiring resources (locks, file handles, sockets) without using RAII wrappers.

**Patterns:**
```cpp
mutex.lock();
do_work();
mutex.unlock();  // If do_work() throws, lock is never released
```
```cpp
FILE *f = fopen(path, "r");
process(f);
fclose(f);  // If process() throws, file leaks
```

**Fix:**
```cpp
std::lock_guard<std::mutex> lock(mutex);  // RAII: auto-unlocks
do_work();
```
```cpp
auto f = std::unique_ptr<FILE, decltype(&fclose)>(fopen(path, "r"), &fclose);
if (!f) return error("Cannot open file");
process(f.get());
// Auto-closed when f goes out of scope
```

### MEM010 — Unsafe type cast

**What to look for:** reinterpret_cast or C-style casts that violate strict aliasing rules or cast between unrelated types.

**Patterns:**
```cpp
float f = 3.14f;
int *p = (int *)&f;  // Strict aliasing violation
int bits = *p;
```
```cpp
Base *base = get_base();
Derived *derived = (Derived *)base;  // No type check
derived->derived_method();  // Undefined if base is not actually Derived
```

**Fix:**
```cpp
float f = 3.14f;
int bits;
static_assert(sizeof(f) == sizeof(bits), "Size mismatch");
memcpy(&bits, &f, sizeof(f));  // Safe type punning via memcpy
```
```cpp
Base *base = get_base();
Derived *derived = dynamic_cast<Derived *>(base);  // Type-safe cast
if (!derived) {
    logger_error("Invalid cast from Base to Derived");
    return error_result;
}
derived->derived_method();
```
