---
name: codebase-indexer
description: "Generates a codebase map for domestic AI models to compensate for lack of native codebase indexing. Scans project structure, identifies key modules, entry points, and dependency relationships. Use when starting a new project, when the user says 'explore/了解/看看/结构', or when the model needs to understand a large codebase quickly. Outputs a structured PROJECT_CONTEXT.md with directory tree, module descriptions, entry points, and dependency graph."
license: Apache-2.0
---

# Codebase Indexer — 代码库地图生成器

国产模型没有 Codex 的原生代码库索引能力。本 skill 通过扫描项目文件生成结构化地图，让模型快速理解项目结构。

## 触发条件

- 首次进入项目目录
- 用户说"探索/了解/看看/看看结构/有什么文件"
- 用户说"这个项目是怎么组织的"
- PROJECT_CONTEXT.md 不存在或超过 7 天未更新

## 执行流程

### 1. 扫描项目结构
```
使用 LS 扫描项目根目录（排除 node_modules, .git, __pycache__, .venv）
使用 Glob 按文件类型分类：*.py, *.gd, *.ts, *.js, *.go, *.cpp, *.h
使用 Grep 查找入口点：main, app, index, __main__, _ready, _process
```

### 2. 识别关键模块
```
对每个一级目录，读 1-2 个代表性文件的前 30 行
识别：模块职责、主要类/函数、导入依赖
```

### 3. 分析依赖关系
```
使用 Grep 查找 import/require/use 语句
构建模块间依赖关系图
标注循环依赖
```

### 4. 生成 PROJECT_CONTEXT.md
按以下模板输出，写入项目根目录：

```markdown
# PROJECT_CONTEXT.md

> 自动生成于 [日期]，由 codebase-indexer 生成
> 下次更新建议：[7天后日期]

## 目录结构
[项目根目录]
├── src/          # [职责描述]
│   ├── models/   # [职责]
│   ├── services/ # [职责]
│   └── utils/    # [职责]
├── tests/        # [测试目录]
└── [其他目录]

## 关键模块
| 模块 | 路径 | 职责 | 入口文件 |
|------|------|------|----------|
| [模块名] | [路径] | [一句话职责] | [入口文件] |

## 入口点
- [入口类型]：[文件路径] → [函数/类名]

## 依赖关系
[A 模块] → [B 模块]：[依赖原因]
[A 模块] → [C 模块]：[依赖原因]

## 技术栈
- 语言：[检测到的语言]
- 框架：[检测到的框架]
- 构建工具：[检测到的工具]

## 决策日志
（后续由 model-upgrade 规则自动追加）

## 错误模式
（后续由 model-upgrade 规则自动追加）
```

## 输出要求

1. **目录结构**：只列到 2 级深度，每个目录附一句话职责
2. **关键模块**：不超过 8 个，优先列核心业务模块
3. **入口点**：列出所有可执行/可启动的入口
4. **依赖关系**：只列模块间依赖，不列第三方库依赖
5. **必须写入** `PROJECT_CONTEXT.md`，不只在对话中展示

## 语言检测规则

| 文件特征 | 判定语言 | 入口检测 |
|----------|---------|----------|
| *.py, requirements.txt, pyproject.toml | Python | `if __name__ == "__main__"`, `def main()` |
| *.gd, project.godot | GDScript | `func _ready()`, `func _process()` |
| *.ts, *.tsx, package.json | TypeScript | `export default`, `main.ts` |
| *.js, *.jsx, package.json | JavaScript | `module.exports`, `index.js` |
| *.go, go.mod | Go | `func main()`, `package main` |
| *.cpp, *.h, CMakeLists.txt | C/C++ | `int main(`, `WinMain(` |