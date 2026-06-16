# Agent Skills — 部署与使用教程

## 快速开始

### 前置条件

- 任意 AI 编码助手（Claude Code、Trae、Cursor、Windsurf、Cline 等）
- Git（用于克隆仓库）

### 安装步骤

#### 方式一：克隆仓库（推荐）

```bash
git clone https://github.com/Nothern131/agent-skills.git
```

#### 方式二：下载 ZIP

访问 https://github.com/Nothern131/agent-skills ，点击 "Code" → "Download ZIP"

---

## 各工具安装指南

### Claude Code

```bash
# 方式1：添加为插件
/plugin marketplace add Nothern131/agent-skills

# 方式2：手动安装
cp -r agent-skills/skills/* ~/.claude/skills/
```

### Trae

```powershell
# Windows
xcopy /E /I agent-skills\skills\* %USERPROFILE%\.trae\skills\

# macOS/Linux
cp -r agent-skills/skills/* ~/.trae/skills/
```

### Cursor

1. 打开 Cursor Settings → Features → Rules
2. 将 `skills/codeguard/SKILL.md` 内容添加为 Project Rule
3. 将 `skills/model-upgrade/SKILL.md` 内容添加为 Project Rule
4. 或将内容放入 `.cursor/rules/` 目录

### Windsurf (Codeium)

1. 打开 Windsurf Settings → AI → Rules
2. 将 Skill 内容添加为自定义规则
3. 或合并到 `.windsurfrules` 文件

### Cline (VS Code Extension)

1. 打开 Cline 设置 → Custom Instructions
2. 粘贴 SKILL.md 中的指令内容
3. 或添加到项目根目录的 `.clinerules` 文件

### GitHub Copilot

1. 在仓库中创建 `.github/copilot-instructions.md`
2. 粘贴 Skill 内容

### Aider

```bash
# 作为自定义指令文件
aider --message-file skills/codeguard/SKILL.md

# 或添加到配置
echo "read: skills/codeguard/SKILL.md" >> .aider.conf.yml
```

---

## 使用教程

### 教程1：使用 codeguard 进行代码审查

**场景：** 你刚写完一段代码，想检查质量。

**操作：**

```
你：请用 codeguard 审查 src/auth/login.py 的代码质量
```

**AI 会自动：**
1. 加载 codeguard 的 8 个维度规则
2. 读取你的代码
3. 逐维度扫描
4. 输出 0-100 评分报告
5. 按严重度排序列出问题
6. 为每个问题提供修复建议

**输出示例：**
```
CodeGuard Scan Report
=====================
Overall Score: 72/100

CRITICAL:
- [PERM002] login.py:15 — SQL injection: string concatenation in query

HIGH:
- [ERR001] login.py:8 — Missing try-catch around database call
- [ERR008] login.py:10 — No input validation on username parameter

MEDIUM:
- [DIAG001] login.py — No logging in authentication flow

Top Priority Fixes:
1. [CRITICAL] Replace string concatenation with parameterized query
2. [HIGH] Add try-catch with proper error logging
3. [HIGH] Add input validation for username and password
```

### 教程2：使用 model-upgrade 完成复杂任务

**场景：** 你需要重构一个用户认证模块。

**操作：**

```
你：按照 model-upgrade 框架重构用户认证模块
```

**AI 会自动：**
1. 加载 PROJECT_CONTEXT.md（或生成）
2. 执行影响分析
3. 拆解为小步骤
4. 逐步执行，每步验证
5. 完成后总结变更

**输出示例：**
```
影响分析：
1. 修改文件：api/auth.py, services/user_service.py
2. 依赖文件：utils/validator.py, models/user.py
3. 潜在副作用：现有登录流程可能受影响
4. 需要测试：test_auth.py, test_user_service.py

任务拆解：
Step 1: 重构 User 模型（添加字段验证）
Step 2: 重构 UserService（提取验证逻辑）
Step 3: 重构 API 端点（统一错误格式）
Step 4: 运行测试
Step 5: codeguard 扫描

执行 Step 1...
[验证通过] ✓

执行 Step 2...
[验证通过] ✓

...

变更总结：
- 修改文件：3个
- 新增测试：5个
- codeguard 评分：从 45 提升到 88
```

### 教程3：使用 clarifying-questions 明确需求

**场景：** 你有一个模糊的想法，还没想清楚具体要做什么。

**操作：**

```
你：我想做一个用户管理功能
```

**AI 会自动：**
1. 识别需求中的模糊点
2. 生成针对性的澄清问题
3. 根据你的回答明确需求

**输出示例：**
```
在开始之前，我需要确认几个关键点：

1. 用户管理包含哪些操作？
   - 仅 CRUD？还是包含角色/权限管理？
   
2. 用户量级预期？
   - 100人？1000人？10万人？
   
3. 认证方式？
   - 用户名密码？OAuth2？手机验证码？
   
4. 是否需要管理后台？
   - 还是只需要 API？
```

---

## model-upgrade 项目规则部署

### 初始化新项目

```bash
# 1. 复制规则模板到项目根目录
cp agent-skills/skills/model-upgrade/templates/rules.md .trae/rules.md

# 2. 复制上下文模板
cp agent-skills/skills/model-upgrade/templates/PROJECT_CONTEXT.md ./PROJECT_CONTEXT.md

# 3. 在 AI 助手中开始会话
# 说："初始化项目上下文，为当前代码库生成 PROJECT_CONTEXT.md"
```

### 规则文件位置

| 工具 | 规则文件位置 |
|------|------------|
| Trae | `.trae/rules.md` |
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursor/rules/` |
| Windsurf | `.windsurfrules` |
| Cline | `.clinerules` |

---

## Godot 编辑器插件安装

### 安装步骤

```bash
# 1. 克隆仓库
git clone https://github.com/Nothern131/godot-editor-plugins.git

# 2. 复制需要的插件到你的 Godot 项目
# 例如安装 camera_fx 插件：
cp -r godot-editor-plugins/plugins/camera_fx/ /你的Godot项目/addons/camera_fx/

# 3. 在 Godot 编辑器中启用
# 项目 → 项目设置 → 插件 → 启用对应插件
```

### 可用插件

| 插件 | 功能 | 适用场景 |
|------|------|---------|
| auto_collision_occlusion | 自动生成碰撞和遮挡多边形 | TileMap/Sprite 场景 |
| camera_fx | 摄像机震动、冻结、缩放特效 | 2D 游戏动作场景 |
| debug_overlay | 实时调试信息叠加层 | 开发调试 |
| dialogue_editor | 可视化对话编辑器 | 对话系统 |
| map_expander | 地图自动扩展 | 大地图生成 |
| plugin_manager | 插件创建和管理 | 插件开发 |
| scene_templates | 18种场景模板一键生成 | 快速创建场景 |
| spritesheet_animator | 精灵图切割和动画生成 | 角色动画 |
| vfx_generator | 粒子特效生成器 | 视觉特效 |
