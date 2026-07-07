# Codebase Indexer — Agent Skill for Project Structure Analysis

[![Skill](https://img.shields.io/badge/Agent_Skill-codebase_indexer-blue)](https://github.com/anthropics/skills)

Codebase Indexer compensates for domestic AI models' lack of native codebase indexing. It scans project structure, identifies key modules, entry points, and dependency relationships, outputting a structured `PROJECT_CONTEXT.md` for the model to reference.

## Features

- **Auto-scan**: Reads directory structure, classifies files by language
- **Module detection**: Identifies key modules and their responsibilities
- **Entry point discovery**: Finds all executable/launchable entry points
- **Dependency graph**: Maps inter-module dependencies, detects circular dependencies
- **Tech stack detection**: Identifies language, framework, and build tools
- **PROJECT_CONTEXT.md generation**: Outputs a structured reference file

## Supported Languages

Python, GDScript, TypeScript/JavaScript, Go, C/C++

## Usage

This skill is auto-triggered by the model-upgrade rules (v3) when:
- First entering a project
- User says "explore/了解/看看/结构"
- PROJECT_CONTEXT.md is missing or outdated (> 7 days)

## Output

A `PROJECT_CONTEXT.md` file in the project root containing:
- Directory structure with responsibilities
- Key module table
- Entry point list
- Dependency relationship graph
- Tech stack summary
- Decision log and error pattern sections (for model-upgrade rules)