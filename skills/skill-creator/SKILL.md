---
name: skill-creator
description: >
  创建新 Skill 的元技能。当用户说"创建skill/做技能/新技能/写一个skill"，
  或 Agent 发现某类任务反复出现且没有现成 Skill 覆盖时自动触发。
  教 Agent 如何按 TRAE Agent Skills 规范创建完整的 Skill 文件，
  包含 YAML 前置元数据、触发条件、执行流程、规则约束、参考文档。
  适用于任何需要封装为可复用 Skill 的工作流。
license: Apache-2.0
---

# Skill Creator — 元技能：创建 Skill 的技能

让 Agent 自己创建 Skill。当某类任务反复出现、没有现成 Skill 覆盖、或用户明确要求时，Agent 可以用本 Skill 的规范创建一个新的 Skill。

## 触发条件

- 用户说"创建skill/做技能/写一个skill/新技能/封装成skill"
- Agent 发现某类任务模式反复出现 3 次以上，且没有现成 Skill 覆盖
- 用户说"把刚才做的事做成一个skill"
- 用户说"能不能让 AI 自动做这个"（判断是否需要封装为 Skill）

## Skill 文件结构

```
skills/<skill-name>/
├── SKILL.md          # 必需：Skill 定义文件
├── README.md         # 可选：使用说明
├── LICENSE.txt       # 可选：许可证
└── references/       # 可选：参考文档
    ├── guide.md
    └── examples.md
```

**命名规范**：
- skill-name 用 kebab-case（小写+连字符）：`codebase-indexer`, `ponytail-ladder`
- 名称描述能力，不描述场景：`test-automation` 而非 `when-user-says-test`
- 避免与已有 Skill 重名（先 `ls skills/` 检查）

## YAML 前置元数据（必需）

```yaml
---
name: <skill-name>
description: >
  <3-5 句话描述 Skill 做什么、何时触发、解决什么问题>
  <包含触发关键词，让 Agent 能自动匹配>
argument-hint: "[参数提示]"  # 可选，有参数时必填
license: Apache-2.0  # 或其他
---
```

**description 编写规则**：
1. 第一句说清楚 Skill 做什么
2. 第二句说清楚何时触发（含关键词）
3. 第三句说清楚输入/输出
4. 如果有参数，说明 argument-hint
5. 用中文编写（因为面向 TRAE CN + 国产模型）

**示例（好的 description）**：
```yaml
description: >
  强制 AI 在写任何代码前执行 7 阶决策梯，杜绝过度工程化。
  代码量减少 54%，Token 减少 22%，成本降低 20%。
  适用于任何编码任务。用户说"过度工程化/简化/YAGNI/偷懒模式"时自动触发。
```

**示例（不好的 description）**：
```yaml
description: "A skill for creating skills"  # 太短，没有触发条件
```

## SKILL.md 正文结构

### 1. 标题和一句话定位
```markdown
# Skill Name — 中文副标题

一句话说清楚这个 Skill 解决什么问题。
```

### 2. 触发条件（必需）
```markdown
## 触发条件

- 用户说"关键词1/关键词2/关键词3"
- 系统状态检测（如：PROJECT_CONTEXT.md 不存在）
- 某类任务模式出现
```

### 3. 执行流程（核心，必需）
```markdown
## 执行流程

### 1. 步骤一：做什么
具体操作指令，用代码块展示命令

### 2. 步骤二：做什么
具体操作指令

### 3. 步骤三：验证
如何确认步骤完成
```

**执行流程编写规则**：
- 每个步骤包含：做什么 + 怎么做 + 如何验证
- 使用代码块展示具体命令（`LS`, `Grep`, `Read`, `RunCommand` 等）
- 步骤之间用 `→` 标注数据流
- 错误处理：每个步骤说明"如果失败怎么办"

### 4. 规则约束（可选）
```markdown
## 规则

- 禁止：xxx
- 必须：xxx
- 优先：xxx
```

### 5. 边界（必需）
```markdown
## 边界

- 本 Skill 管什么
- 本 Skill 不管什么（路由到其他 Skill）
- 什么情况下应该停止
```

### 6. 输出格式（推荐）
```markdown
## 输出格式

[定义 Skill 执行完毕后应该输出什么格式的结果]
```

### 7. 参考（可选）
```markdown
## 参考

- 外部链接
- 相关 Skill
- 参考资料
```

## 创建 Skill 的完整流程

### Step 1：需求分析

回答以下问题：
1. 这个 Skill 解决什么**重复出现**的问题？
2. 触发条件是什么？（用户说什么？系统检测到什么？）
3. 输入是什么？输出是什么？
4. 有没有参数？（如强度级别 lite/full/ultra）
5. 有没有现成的 Skill 可以覆盖？如果不确定，先 `ls skills/` 检查

**判断标准**：以下情况**不需要**创建 Skill
- 一次性任务（不会重复出现）
- 已有 Skill 可以覆盖（先检查 `skills/` 目录）
- 任务太简单（一句话能描述的操作）
- 任务是"写代码"而非"封装工作流"

### Step 2：设计 Skill 结构

```
1. 确定 skill-name（kebab-case）
2. 写 YAML 前置元数据
3. 写标题和一句话定位
4. 写触发条件
5. 写执行流程（核心！）
6. 写规则约束
7. 写边界
8. 写输出格式
9. 写参考（可选）
```

### Step 3：创建文件

```bash
mkdir "skills/<skill-name>"
# 创建 SKILL.md
# 创建 README.md（可选）
# 创建 LICENSE.txt（可选）
# 创建 references/ 目录（可选）
```

### Step 4：验证检查清单

创建完成后，逐项检查：

- [ ] YAML 前置元数据包含 name、description、license
- [ ] description 包含触发关键词
- [ ] 标题有中文副标题
- [ ] 触发条件明确列出用户关键词
- [ ] 执行流程每个步骤有：做什么 + 怎么做 + 验证方法
- [ ] 边界明确（管什么、不管什么）
- [ ] 输出格式定义清晰
- [ ] 文件路径正确（`skills/<skill-name>/SKILL.md`）
- [ ] 命名符合 kebab-case
- [ ] 不与已有 Skill 重名

### Step 5：注册到 2.11 触发表

创建 Skill 后，**必须**更新 `e:\.trae\rules\project_rules.md` 的 2.11 触发表：
1. 在 2.11 表格中新增一行
2. 在第九章触发判断总表中新增一行
3. 在第八章详细参考中新增一行
4. 更新记忆口诀（如需要）

---

## 已有 Skill 参考（学习格式）

| Skill | 特点 | 学习点 |
|-------|------|--------|
| `codebase-indexer` | 项目扫描+地图生成 | 触发条件基于系统状态检测 |
| `test-automation` | 框架检测+运行+解析 | 多框架适配，Closed Loop 集成 |
| `codeguard` | 8维度80规则 | 复杂规则系统，references/ 子目录 |
| `ponytail-ladder` | 7阶决策梯 | argument-hint 参数，3级强度 |
| `clarifying-questions` | 结构化反问 | 任务类型识别+模糊检测 |

---

## 常见反模式（禁止）

1. **Skill 太宽泛**：一个 Skill 做所有事 → 拆成多个
2. **Skill 太窄**：只覆盖一个特定场景 → 合并到已有 Skill
3. **触发条件太模糊**："用户需要时触发" → 写具体关键词
4. **执行流程像散文**：大段描述而非步骤化指令 → 用代码块+编号步骤
5. **没有边界**：Skill 不知道什么时候该停 → 写清楚"不管什么"
6. **忘记注册**：创建了 Skill 但不更新 2.11 表 → 创建后必须注册
7. **重复造轮子**：已有 Skill 能覆盖还创建新的 → 先检查 `skills/` 目录

---

## 示例：用本 Skill 创建一个新 Skill

假设用户说："把代码审查流程做成一个 skill"

### Step 1：需求分析
- 问题：每次写完代码需要手动审查
- 触发：用户说"审查/检查/review" 或代码生成后
- 输入：代码文件路径
- 输出：审查报告（评分+问题列表）
- 已有：`codeguard` 已覆盖安全扫描，但不覆盖代码风格审查

### Step 2：设计
```
name: code-review
description: 代码风格和最佳实践审查。检查命名规范、代码结构、注释质量。
             用户说"审查/检查/review/代码审查"时触发。
             输出评分和修复建议。
```

### Step 3：创建文件
```bash
mkdir "skills/code-review"
# 写 SKILL.md
```

### Step 4：验证
检查清单全部通过

### Step 5：注册
更新 `project_rules.md` 的 2.11 表

---

## 边界

- 本 Skill 管：Skill 的创建流程、格式规范、注册流程
- 本 Skill 不管：Skill 的具体内容（那由 Agent 根据任务自己设计）
- 本 Skill 不管：更新已有 Skill（那是 `Edit` 工具的事）
- 本 Skill 不管：删除 Skill（那是 `DeleteFile` 工具的事）

## 与 TRAE 内置 skill-creator 的关系

TRAE 内置了 `skill-creator` 作为系统级 Skill 发现机制。本 Skill 是**本地版本**，包含：
- 我们这个项目的具体 Skill 格式规范（YAML 前置元数据、中文描述）
- 2.11 触发表注册流程
- 基于已有 Skill 的实战参考
- 反模式清单

当 Agent 需要创建 Skill 时，优先使用本 Skill 的规范，因为它针对 TRAE CN + 国产模型 + 本项目规则体系做了适配。