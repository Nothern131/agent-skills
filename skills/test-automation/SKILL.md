---
name: test-automation
description: "Auto-detects test framework, runs tests, parses results, and feeds back to the Closed Loop for autonomous quality verification. Compensates for domestic models' lack of native test execution. Use after code generation, after Closed Loop fixes, or when the user says 'test/验证/跑测试'. Works with pytest, unittest, Jest, Go test, and GDScript GUT."
license: Apache-2.0
---

# Test Automation — 自动化测试执行器

国产模型在 TRAE 中没有 Codex 的原生测试执行能力。本 skill 自动检测测试框架、运行测试、解析结果，将反馈注入 Closed Loop。

## 触发条件

- 代码生成完成后（Closed Loop 验证阶段）
- Closed Loop 修复后（重新验证）
- 用户说"测试/验证/跑测试/检查一下/能跑吗"
- 交付前（证据完成）

## 执行流程

### 1. 检测测试框架
```
检查项目根目录的配置文件：
  Python: pytest → pytest.ini / pyproject.toml / setup.cfg
          unittest → 无配置文件，检查 tests/ 目录下是否有 test_*.py
  JS/TS:  Jest → jest.config.* / package.json 含 jest
          Mocha → .mocharc.*
  Go:     go test → go.mod 存在
  GDScript: GUT → addons/gut/ 目录存在
  C/C++:  CTest → CMakeLists.txt 含 enable_testing()
          Catch2 → 检查 #include <catch2/ 引用
```

### 2. 运行测试
```
Python:  pytest tests/ -v --tb=short 2>&1 | head -100
         （如果 pytest 不存在，用 python -m unittest discover tests/）
JS/TS:   npx jest --verbose 2>&1 | head -100
Go:      go test ./... -v 2>&1 | head -100
GDScript: 通过 Godot 编辑器运行 GUT 测试
C/C++:   ctest --test-dir build --output-on-failure 2>&1 | head -100
```

### 3. 解析结果
```
提取关键信息：
  - 总测试数：[从输出中提取 total/N 个测试]
  - 通过数：[从输出中提取 passed/ok]
  - 失败数：[从输出中提取 failed/F]
  - 错误数：[从输出中提取 errors/E]
  - 失败测试名称：[每个失败的测试名]
  - 错误信息：[每个失败的错误摘要]
```

### 4. 输出格式

```
【测试结果】
- 框架：[检测到的框架]
- 总数：[N] | 通过：[P] | 失败：[F] | 错误：[E]
- 通过率：[P/N * 100]%

【失败详情】
- [测试名1]：[错误摘要]
- [测试名2]：[错误摘要]

【Closed Loop 反馈】
- 是否达标：是/否（通过率 ≥ 90% 且无 Critical 失败）
- 是否需要修复：[是/否]
- 修复优先级：[哪些失败需要优先修复]
```

## 框架未检测到时

如果无法检测到测试框架，输出：
```
【测试结果】
- 框架：未检测到
- 建议：项目当前缺少测试框架。建议配置：
  - Python：pip install pytest
  - JS/TS：npm install --save-dev jest
  - Go：go test 已内置
  - GDScript：安装 GUT 插件

是否跳过测试验证，继续交付？[是/否]
```

## 集成到 Closed Loop

```
测试结果解析后 → 判断是否达标
  ├→ 达标（通过率 ≥ 90%，无 Critical 失败）→ 继续 Open Loop（提下一步）
  └→ 不达标 → 触发 Closed Loop
       ├→ 分析失败原因
       ├→ 修复代码
       ├→ 重新运行测试（本 skill）
       └→ 最多 3 次循环，仍不达标→停止并报告
```

## 边界情况

- **测试框架未安装**：提示用户安装，不阻塞流程
- **测试超时（> 60 秒）**：输出已完成部分，标注"测试运行中，可能有超时问题"
- **无测试文件**：标注"项目无测试文件"，建议补写
- **测试输出过长（> 2000 行）**：只取前 100 行和后 20 行，中间省略