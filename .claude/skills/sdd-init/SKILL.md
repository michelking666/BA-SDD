---
name: sdd:init
description: SDD 阶段0：初始化框架目录结构与种子文档，在空项目或新项目中一键搭建 SDD 工作流骨架
disable-model-invocation: true
---

# SDD Init — 初始化 SDD 框架

## 触发方式
用户输入 `/sdd:init`

## 用途
在一个新项目（或尚未初始化的目录）中，一键生成 SDD 框架所需的全部目录结构和种子文档。
执行后即可直接进入 `/sdd:new` → `/sdd:propose` → ... 的变更工作流。

## 设计原则
- **幂等**：已存在的目录和文件一律跳过，绝不覆盖用户已有内容。
- **种子文档来自模板**：规范类文档从本 skill 的 `templates/` 目录复制，保证内容与框架版本一致。
- **不创建示例变更**：只搭骨架，不生成示例数据。

## 执行步骤

### 1. 确认初始化位置
以当前工作目录为 SDD 根目录。先列出当前目录内容，向用户确认：
- 若目录已包含 `openspec/`、`knowledge/`、`llmwiki/` 等结构，提示"检测到已存在的 SDD 结构，将只补齐缺失部分，不覆盖现有文件"。
- 若是空目录或全新项目，直接继续。

### 2. 创建目录树
执行（已存在的目录 `mkdir -p` 不报错，天然幂等）：

```bash
mkdir -p \
  llmwiki/specs-released \
  llmwiki/archive \
  llmwiki/prd/active \
  llmwiki/prd/output \
  openspec/changes/active \
  openspec/changes/approved \
  openspec/changes/archive \
  knowledge/rules \
  knowledge/memory/experience \
  knowledge/history-code \
  knowledge/history-test-cases \
  src/frontend \
  src/backend \
  src/testcase
```

### 3. 复制种子文档（仅当目标不存在时）
模板位于本 skill 目录的 `templates/` 子目录。对每个目标文件，先判断是否已存在，存在则跳过。

逐项处理（`$SKILL` 指向本 skill 所在目录）：

| 模板文件 | 目标路径 |
|----------|----------|
| `templates/CLAUDE.md` | `CLAUDE.md` |
| `templates/architecture-spec.md` | `knowledge/rules/architecture-spec.md` |
| `templates/requirements-spec.md` | `knowledge/rules/requirements-spec.md` |
| `templates/coding-standards.md` | `knowledge/rules/coding-standards.md` |
| `templates/naming-conventions.md` | `knowledge/rules/naming-conventions.md` |
| `templates/pitfalls.md` | `knowledge/memory/experience/pitfalls.md` |
| `templates/patterns.md` | `knowledge/memory/experience/patterns.md` |
| `templates/README.md` | `llmwiki/README.md`（wiki schema） |
| `templates/index.md` | `llmwiki/index.md`（内容目录） |
| `templates/log.md` | `llmwiki/log.md`（演进时间线） |
| `templates/overview.md` | `llmwiki/overview.md`（系统全景综合页） |
| `templates/spec-page-template.md` | `llmwiki/specs-released/_spec-page-template.md`（规格页模板，供归档时参照） |

可用如下脚本（`cp -n` = 不覆盖已存在文件）：

```bash
SKILL=".claude/skills/sdd-init/templates"
cp -n "$SKILL/CLAUDE.md"             CLAUDE.md
cp -n "$SKILL/architecture-spec.md"  knowledge/rules/architecture-spec.md
cp -n "$SKILL/requirements-spec.md"  knowledge/rules/requirements-spec.md
cp -n "$SKILL/coding-standards.md"   knowledge/rules/coding-standards.md
cp -n "$SKILL/naming-conventions.md" knowledge/rules/naming-conventions.md
cp -n "$SKILL/pitfalls.md"           knowledge/memory/experience/pitfalls.md
cp -n "$SKILL/patterns.md"           knowledge/memory/experience/patterns.md
cp -n "$SKILL/README.md"             llmwiki/README.md
cp -n "$SKILL/index.md"              llmwiki/index.md
cp -n "$SKILL/log.md"                llmwiki/log.md
cp -n "$SKILL/overview.md"           llmwiki/overview.md
cp -n "$SKILL/spec-page-template.md" llmwiki/specs-released/_spec-page-template.md
```

> 注意：`$SKILL` 路径基于"当前工作目录就是 SDD 根目录"。若本 skill 安装在用户级 `~/.claude/skills/`，请改用该绝对路径。执行前先确认 `templates/` 实际位置。

### 4. 生成 `.gitkeep` 占位（可选）
为保证空目录可被纳入版本控制，在以下空目录放置 `.gitkeep`（仅当目录为空时）：
`llmwiki/specs-released`、`llmwiki/archive`、`llmwiki/prd/output`、
`openspec/changes/active`、`openspec/changes/approved`、`openspec/changes/archive`、
`knowledge/history-code`、`knowledge/history-test-cases`、
`src/frontend`、`src/backend`、`src/testcase`。

### 5. 校验与输出
执行后用 `find . -type d` 复核目录树是否齐全，并向用户报告：
- 本次**新建**的目录与文件清单
- 本次**跳过**（已存在）的文件清单
- 下一步提示：运行 `/sdd:new` 创建第一个变更

## 完成标志
目录树齐全、8 个种子文档就位，CLAUDE.md 存在。用户可立即进入变更工作流。
