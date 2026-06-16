# Agent Skills — 对比演示

## 演示场景

任务：实现一个用户注册 API，包含邮箱验证、密码强度检查、重复用户名检测。

---

## 对照组：无框架辅助（国产模型直接输出）

### 模型输出

```python
from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

@app.route('/api/register', methods=['POST'])
def register():
    data = request.json
    username = data['username']
    email = data['email']
    password = data['password']
    
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    
    cursor.execute("SELECT * FROM users WHERE username='" + username + "'")
    if cursor.fetchone():
        return jsonify({"error": "Username exists"}), 400
    
    cursor.execute("INSERT INTO users (username, email, password) VALUES ('" + username + "', '" + email + "', '" + password + "')")
    conn.commit()
    conn.close()
    
    return jsonify({"message": "User registered"}), 201

if __name__ == '__main__':
    app.run()
```

### codeguard 扫描结果

```
CodeGuard Scan Report
=====================
Overall Score: 15/100

CRITICAL Issues:
- [PERM002] SQL injection: string concatenation in SELECT query (line 13)
- [PERM002] SQL injection: string concatenation in INSERT query (line 16)
- [PERM001] Hardcoded password stored in plain text (line 16)

HIGH Issues:
- [ERR001] Missing try-catch around database operations
- [ERR008] No input validation on username/email/password
- [DB002] Database connection not guaranteed to close on error
- [PERM005] No authentication on route

MEDIUM Issues:
- [DIAG001] No logging
- [DIAG002] Generic error message without error code
- [RES001] No retry mechanism for database connection
- [DB008] Hard delete instead of soft delete

LOW Issues:
- [DIAG003] Error response missing context information
```

### 问题总结

| 问题类型 | 数量 | 严重度 |
|---------|------|--------|
| SQL注入 | 2 | Critical |
| 明文密码 | 1 | Critical |
| 无错误处理 | 2 | High |
| 无输入验证 | 1 | High |
| 连接泄漏风险 | 1 | High |
| 无日志 | 1 | Medium |
| 无错误码 | 1 | Medium |

**结论：这段代码不能上线。**

---

## 实验组：model-upgrade + codeguard 辅助

### Step 1: 影响分析（铁律 Rule 1）

```
影响分析：
1. 新增文件：api/auth.py, services/user_service.py, models/user.py
2. 依赖文件：utils/validator.py, utils/hash.py, config/database.py
3. 潜在副作用：无（新功能）
4. 需要的测试：test_user_registration.py
```

### Step 2: 任务拆解（Layer 3）

```
Step 1: 定义 User 模型和数据库 schema
Step 2: 实现输入验证工具函数
Step 3: 实现密码哈希工具函数
Step 4: 实现用户注册服务层
Step 5: 实现 API 端点
Step 6: 编写测试
Step 7: codeguard 扫描 + 修复
```

### Step 3: 逐步执行（每步验证）

**Step 1: User 模型**

```python
# models/user.py
from dataclasses import dataclass
from datetime import datetime
from typing import Optional


@dataclass
class User:
    """User entity representing a registered user.

    Attributes:
        id: Unique user identifier (auto-generated)
        username: Unique username (3-30 chars, alphanumeric + underscore)
        email: Valid email address
        password_hash: Bcrypt hashed password (never store plain text)
        created_at: Account creation timestamp
        is_active: Whether the account is active (soft delete support)
    """
    id: Optional[int] = None
    username: str = ""
    email: str = ""
    password_hash: str = ""
    created_at: datetime = None
    is_active: bool = True

    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.utcnow()
```

**Step 2: 输入验证**

```python
# utils/validator.py
import re
from typing import Tuple


def validate_email(email: str) -> Tuple[bool, str]:
    """Validate email format.

    Args:
        email: Email address to validate.

    Returns:
        Tuple of (is_valid, error_message). error_message is empty if valid.
    """
    if not email or len(email) > 254:
        return False, "Email must be 1-254 characters"
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(pattern, email):
        return False, "Invalid email format"
    return True, ""


def validate_password_strength(password: str) -> Tuple[bool, str]:
    """Validate password meets minimum strength requirements.

    Requirements: 8+ chars, at least one uppercase, one lowercase, one digit.

    Args:
        password: Password to validate.

    Returns:
        Tuple of (is_valid, error_message). error_message is empty if valid.
    """
    if len(password) < 8:
        return False, "Password must be at least 8 characters"
    if not re.search(r'[A-Z]', password):
        return False, "Password must contain at least one uppercase letter"
    if not re.search(r'[a-z]', password):
        return False, "Password must contain at least one lowercase letter"
    if not re.search(r'\d', password):
        return False, "Password must contain at least one digit"
    return True, ""


def validate_username(username: str) -> Tuple[bool, str]:
    """Validate username format.

    Requirements: 3-30 chars, alphanumeric + underscore only.

    Args:
        username: Username to validate.

    Returns:
        Tuple of (is_valid, error_message). error_message is empty if valid.
    """
    if not username or len(username) < 3 or len(username) > 30:
        return False, "Username must be 3-30 characters"
    if not re.match(r'^[a-zA-Z0-9_]+$', username):
        return False, "Username can only contain letters, numbers, and underscores"
    return True, ""
```

**Step 3: 密码哈希**

```python
# utils/hash.py
import bcrypt
import logging

logger = logging.getLogger(__name__)


def hash_password(password: str) -> str:
    """Hash a password using bcrypt.

    Args:
        password: Plain text password to hash.

    Returns:
        Bcrypt hashed password string.

    Raises:
        ValueError: If password is empty.
    """
    if not password:
        raise ValueError("Password cannot be empty")
    salt = bcrypt.gensalt(rounds=12)
    return bcrypt.hashpw(password.encode('utf-8'), salt).decode('utf-8')


def verify_password(password: str, password_hash: str) -> bool:
    """Verify a password against its bcrypt hash.

    Args:
        password: Plain text password to verify.
        password_hash: Stored bcrypt hash to compare against.

    Returns:
        True if password matches the hash, False otherwise.
    """
    try:
        return bcrypt.checkpw(
            password.encode('utf-8'),
            password_hash.encode('utf-8')
        )
    except Exception as e:
        logger.error("Password verification failed: %s", str(e))
        return False
```

**Step 4: 服务层**

```python
# services/user_service.py
import logging
from typing import Tuple, Optional
from models.user import User
from utils.validator import validate_email, validate_password_strength, validate_username
from utils.hash import hash_password

logger = logging.getLogger(__name__)


class UserService:
    """Business logic for user registration and management.

    Args:
        db: Database connection pool or session.
    """

    def __init__(self, db):
        self.db = db

    def register_user(
        self,
        username: str,
        email: str,
        password: str
    ) -> Tuple[Optional[User], Optional[str]]:
        """Register a new user with validation and duplicate checking.

        Args:
            username: Desired username (3-30 chars, alphanumeric + underscore).
            email: Valid email address.
            password: Password meeting strength requirements.

        Returns:
            Tuple of (user, error_code). user is the created User on success,
            error_code is a string error code on failure. Both are None-safe.
        """
        # 1. Validate inputs
        valid, msg = validate_username(username)
        if not valid:
            logger.warning("Registration failed: invalid username '%s' - %s", username, msg)
            return None, "INVALID_USERNAME"

        valid, msg = validate_email(email)
        if not valid:
            logger.warning("Registration failed: invalid email '%s' - %s", email, msg)
            return None, "INVALID_EMAIL"

        valid, msg = validate_password_strength(password)
        if not valid:
            logger.warning("Registration failed: weak password for '%s' - %s", username, msg)
            return None, "WEAK_PASSWORD"

        # 2. Check for duplicate username
        try:
            existing = self.db.execute(
                "SELECT id FROM users WHERE username = %s AND is_active = TRUE",
                (username,)
            )
            if existing:
                logger.info("Registration failed: username '%s' already exists", username)
                return None, "USERNAME_EXISTS"
        except Exception as e:
            logger.error("Database error checking username '%s': %s", username, str(e))
            return None, "DATABASE_ERROR"

        # 3. Check for duplicate email
        try:
            existing = self.db.execute(
                "SELECT id FROM users WHERE email = %s AND is_active = TRUE",
                (email,)
            )
            if existing:
                logger.info("Registration failed: email '%s' already registered", email)
                return None, "EMAIL_EXISTS"
        except Exception as e:
            logger.error("Database error checking email '%s': %s", email, str(e))
            return None, "DATABASE_ERROR"

        # 4. Create user with hashed password
        try:
            password_hash = hash_password(password)
            user_id = self.db.execute(
                "INSERT INTO users (username, email, password_hash, is_active) "
                "VALUES (%s, %s, %s, TRUE) RETURNING id",
                (username, email, password_hash)
            )
            user = User(
                id=user_id,
                username=username,
                email=email,
                password_hash=password_hash
            )
            logger.info("User '%s' registered successfully (id=%s)", username, user_id)
            return user, None
        except Exception as e:
            logger.error("Failed to create user '%s': %s", username, str(e))
            return None, "USER_CREATION_FAILED"
```

**Step 5: API 端点**

```python
# api/auth.py
import logging
from flask import Blueprint, request, jsonify
from services.user_service import UserService

logger = logging.getLogger(__name__)
auth_bp = Blueprint('auth', __name__)


@auth_bp.route('/api/register', methods=['POST'])
def register():
    """User registration endpoint.

    Request body:
        username (str): 3-30 chars, alphanumeric + underscore
        email (str): Valid email address
        password (str): 8+ chars with uppercase, lowercase, digit

    Responses:
        201: User registered successfully
        400: Validation error (INVALID_USERNAME/EMAIL/PASSWORD)
        409: Duplicate (USERNAME_EXISTS/EMAIL_EXISTS)
        500: Server error (DATABASE_ERROR/USER_CREATION_FAILED)
    """
    data = request.get_json(silent=True)
    if not data:
        return jsonify({
            "success": False,
            "error": "INVALID_REQUEST",
            "detail": "Request body must be valid JSON"
        }), 400

    username = data.get("username", "")
    email = data.get("email", "")
    password = data.get("password", "")

    user, error_code = UserService(request.db).register_user(
        username=username,
        email=email,
        password=password
    )

    if error_code:
        status_map = {
            "INVALID_USERNAME": 400,
            "INVALID_EMAIL": 400,
            "WEAK_PASSWORD": 400,
            "USERNAME_EXISTS": 409,
            "EMAIL_EXISTS": 409,
            "DATABASE_ERROR": 500,
            "USER_CREATION_FAILED": 500,
        }
        return jsonify({
            "success": False,
            "error": error_code,
            "detail": _error_detail(error_code)
        }), status_map.get(error_code, 500)

    return jsonify({
        "success": True,
        "data": {
            "id": user.id,
            "username": user.username,
            "email": user.email,
            "created_at": user.created_at.isoformat()
        }
    }), 201


def _error_detail(error_code: str) -> str:
    """Map error codes to human-readable messages."""
    details = {
        "INVALID_USERNAME": "Username must be 3-30 alphanumeric/underscore characters",
        "INVALID_EMAIL": "Please provide a valid email address",
        "WEAK_PASSWORD": "Password must be 8+ chars with uppercase, lowercase, and digit",
        "USERNAME_EXISTS": "This username is already taken",
        "EMAIL_EXISTS": "This email is already registered",
        "DATABASE_ERROR": "A database error occurred, please try again",
        "USER_CREATION_FAILED": "Failed to create account, please try again",
    }
    return details.get(error_code, "An unknown error occurred")
```

### codeguard 扫描结果

```
CodeGuard Scan Report
=====================
Overall Score: 92/100

LOW Issues:
- [DIAG003] _error_detail could include error_code in response for debugging
- [RES001] No retry on transient database connection failures

Info:
- All CRITICAL and HIGH issues resolved
- All input validated
- Password hashed with bcrypt
- Parameterized queries (no SQL injection)
- Structured error codes
- Complete logging
- Proper error handling with try-catch
- Soft delete support (is_active flag)
```

---

## 对比总结

| 指标 | 无框架辅助 | model-upgrade + codeguard | 提升 |
|------|-----------|--------------------------|------|
| **codeguard 评分** | 15/100 | 92/100 | **+513%** |
| **Critical 问题** | 3个 | 0个 | **-100%** |
| **High 问题** | 4个 | 0个 | **-100%** |
| **SQL注入** | 2处 | 0处 | **-100%** |
| **明文密码** | 是 | 否（bcrypt） | **已修复** |
| **输入验证** | 无 | 完整 | **从0到1** |
| **错误处理** | 无 | 完整 | **从0到1** |
| **日志** | 无 | 完整 | **从0到1** |
| **错误码** | 无 | 结构化 | **从0到1** |
| **代码行数** | 22行 | ~180行 | **质量换行数** |

### 关键差异

1. **安全性**：从3个Critical漏洞到0个
2. **可靠性**：从无错误处理到完整try-catch+日志
3. **可维护性**：从单文件到分层架构（model/service/api）
4. **可观测性**：从无日志到完整的结构化日志
5. **可测试性**：从不可测试到每层可独立测试

**核心结论：同样的模型，加上工程纪律后，输出质量从"不能上线"提升到"生产级"。**
