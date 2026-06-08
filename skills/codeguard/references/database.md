# Database Protection Rules

10 rules that detect database safety issues and data integrity risks.

## Rule Table

| ID | Severity | Detection |
|----|----------|-----------|
| DB001 | Critical | Multi-step write missing transaction |
| DB002 | High | Database connection not explicitly closed |
| DB003 | Medium | Using direct connection instead of connection pool |
| DB004 | High | Write operation missing data validation |
| DB005 | Low | Query missing index hint |
| DB006 | High | N+1 query problem |
| DB007 | High | Destructive operation missing backup prompt |
| DB008 | Medium | Using hard delete instead of soft delete |
| DB009 | Medium | Database migration missing rollback plan |
| DB010 | High | Sensitive data stored unencrypted |

## Detection Patterns

### DB001 — Multi-step write missing transaction

**What to look for:** Multiple insert/update/delete operations that should be atomic but are not wrapped in a transaction.

**Patterns:**
```javascript
await db.query('INSERT INTO orders ...', [orderData]);
await db.query('INSERT INTO order_items ...', [items]);  // If this fails, order exists without items
await db.query('UPDATE inventory ...', [quantities]);    // Inconsistent state
```
```python
cursor.execute("INSERT INTO orders ...")
cursor.execute("INSERT INTO order_items ...")  # No transaction
```

**Fix:**
```javascript
const tx = await db.beginTransaction();
try {
  await tx.query('INSERT INTO orders ...', [orderData]);
  await tx.query('INSERT INTO order_items ...', [items]);
  await tx.query('UPDATE inventory ...', [quantities]);
  await tx.commit();
} catch (err) {
  await tx.rollback();
  throw err;
}
```
```python
with db.transaction():
    cursor.execute("INSERT INTO orders ...")
    cursor.execute("INSERT INTO order_items ...")
```

### DB002 — Database connection not explicitly closed

**What to look for:** Connections opened but not guaranteed to close on all code paths.

**Patterns:**
```javascript
const conn = await pool.getConnection();
const result = await conn.query('SELECT ...');
return result;  // Connection never released
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

### DB003 — Using direct connection instead of connection pool

**What to look for:** Creating new database connections for each request instead of using a connection pool.

**Patterns:**
```javascript
const conn = mysql.createConnection({ host, user, password });
```
```python
conn = psycopg2.connect(host=host, user=user, password=password)
```

**Fix:**
```javascript
const pool = mysql.createPool({ host, user, password, connectionLimit: 10 });
const conn = await pool.getConnection();
```
```python
pool = psycopg2.pool.ThreadedConnectionPool(minconn=1, maxconn=10, ...)
conn = pool.getconn()
```

### DB004 — Write operation missing data validation

**What to look for:** INSERT or UPDATE statements that accept user input without validation or sanitization.

**Patterns:**
```javascript
await db.query('INSERT INTO users SET ?', req.body);  // No validation
```
```python
cursor.execute("INSERT INTO users (name, email) VALUES (%s, %s)",
               (request.json["name"], request.json["email"]))  # No validation
```

**Fix:**
```javascript
const { name, email } = await validate(req.body, {
  name: { type: 'string', required: true, maxLength: 100 },
  email: { type: 'email', required: true },
});
await db.query('INSERT INTO users SET ?', { name, email });
```

### DB005 — Query missing index hint

**What to look for:** WHERE clauses on columns that are likely not indexed, especially in large tables.

**Patterns:**
```sql
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2024-01-01';
-- No index on (status, created_at)
```

**Fix:**
```sql
CREATE INDEX idx_orders_status_created ON orders(status, created_at);
SELECT * FROM orders USE INDEX (idx_orders_status_created) WHERE status = 'pending' AND created_at > '2024-01-01';
```

### DB006 — N+1 query problem

**What to look for:** Queries inside loops that could be replaced with a single batch query.

**Patterns:**
```javascript
const users = await db.query('SELECT id FROM users');
for (const user of users) {
  user.orders = await db.query('SELECT * FROM orders WHERE user_id = ?', [user.id]);
}
```

**Fix:**
```javascript
const users = await db.query('SELECT id FROM users');
const userIds = users.map(u => u.id);
const orders = await db.query('SELECT * FROM orders WHERE user_id IN (?)', [userIds]);
const ordersByUser = groupBy(orders, 'user_id');
users.forEach(u => u.orders = ordersByUser[u.id] || []);
```

### DB007 — Destructive operation missing backup prompt

**What to look for:** DROP TABLE, TRUNCATE, or bulk DELETE without a preceding backup or confirmation step.

**Patterns:**
```javascript
await db.query('DROP TABLE old_data');
await db.query('DELETE FROM logs WHERE created_at < ?', [cutoffDate]);
```

**Fix:**
```javascript
// Before destructive operation
if (!options.skipBackup) {
  await db.query('CREATE TABLE old_data_backup AS SELECT * FROM old_data');
  logger.info('Backup created: old_data_backup');
}
await db.query('DROP TABLE old_data');
```

### DB008 — Using hard delete instead of soft delete

**What to look for:** DELETE statements that permanently remove records instead of marking them as deleted.

**Patterns:**
```javascript
await db.query('DELETE FROM users WHERE id = ?', [userId]);
```

**Fix:**
```javascript
await db.query('UPDATE users SET deleted_at = NOW() WHERE id = ?', [userId]);
// Add scope: default scope excludes deleted_at IS NOT NULL
```

### DB009 — Database migration missing rollback plan

**What to look for:** Migration scripts with only an "up" method and no corresponding "down" method.

**Patterns:**
```javascript
exports.up = async (db) => {
  await db.query('ALTER TABLE users ADD COLUMN phone VARCHAR(20)');
};
// No exports.down
```

**Fix:**
```javascript
exports.up = async (db) => {
  await db.query('ALTER TABLE users ADD COLUMN phone VARCHAR(20)');
};

exports.down = async (db) => {
  await db.query('ALTER TABLE users DROP COLUMN phone');
};
```

### DB010 — Sensitive data stored unencrypted

**What to look for:** Columns storing PII, financial data, or credentials in plaintext.

**Patterns:**
```javascript
await db.query('INSERT INTO users (ssn, credit_card) VALUES (?, ?)', [ssn, cardNumber]);
// Stored as plaintext
```

**Fix:**
```javascript
const encryptedSsn = encrypt(ssn, ENCRYPTION_KEY);
const encryptedCard = encrypt(cardNumber, ENCRYPTION_KEY);
await db.query('INSERT INTO users (ssn_encrypted, credit_card_encrypted) VALUES (?, ?)',
  [encryptedSsn, encryptedCard]);
```
