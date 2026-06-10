# SDD 软件设计文档管理框架

> 基于 Claude Code Skills 的结构化需求管理工作流

## 工作流概述

本框架用于项目迭代过程中的需求管理，采用6阶段变更工作流 + 探索模式。

| 命令 | 阶段 | 说明 |
|------|------|------|
| `/sdd:init` | 阶段0 | 初始化框架 — 生成目录结构与种子文档 |
| `/sdd:new` | 阶段1 | 创建变更 — 定义变更范围和目标 |
| `/sdd:propose` | 阶段2 | 提案评审 — 生成原型、提案、设计、任务、计划 |
| `/sdd:apply-change` | 阶段3 | 实施变更 — 前端优先，严格匹配原型 |
| `/sdd:generate-tests` | 阶段4 | 生成测试 — 基于需求规范生成测试用例 |
| `/sdd:verify-change` | 阶段5 | 验证变更 — 验证代码与原型一致性 |
| `/sdd:archive-change` | 阶段6 | 归档变更 — 归档并同步到 llmwiki |
| `/sdd:explore` | — | 探索模式 — 分析想法可行性，引导进入变更流程 |
| `/sdd:wiki-lint` | — | wiki 体检 — 检查 llmwiki 矛盾/陈旧/孤立页，保持知识库健康 |
| `/sdd:prd-generator` | 辅助 | 生成PRD — 基于规格生成Word格式PRD |
| `/sdd:prd-validate` | 辅助 | 验证PRD — 验证PRD与需求规格的契合度 |
| `/sdd:create-approval-record` | 辅助 | 审批记录 — 创建或更新审批记录 |

## 阶段衔接规则

- 前一阶段未完成，禁止触发下一阶段
- 阶段2审批通过后，文件从 `active/` 复制到 `approved/`
- 阶段6归档完成后，文件移至 `archive/`，清理 active/approved

## 职责划分：llmwiki vs knowledge

两者边界清晰，不重叠：

- **llmwiki = 需求事实库（WHAT）**：回答"系统是什么"。存放全量最新规格、历史演变快照、对外交付的 PRD。由 `/sdd:propose`、`/sdd:explore` 读取以了解现状。
- **knowledge = 工程方法与记忆库（HOW）**：回答"该怎么做、做过怎样"。存放编码/架构规范、踩坑与成功模式、历史代码与测试存档。

一句话：**规格去 llmwiki，规矩和记忆去 knowledge。**

> llmwiki 是**持续累积、自我维护的产物**，不是查询时临时拼凑的 RAG。围绕三个操作运转：Ingest 摄入（`/sdd:archive-change` 归档时把规格编译进 wiki）、Query 查询（`/sdd:propose`/`/sdd:explore` 先读 index 定位再钻入，结果可回填）、Lint 体检（`/sdd:wiki-lint` 保持健康）。规程详见 `llmwiki/README.md`。

## 目录结构说明

```
SDD/
├── llmwiki/                    # 需求事实库 (WHAT，累积式 wiki)
│   ├── README.md               # wiki schema：结构/约定/维护规程
│   ├── index.md                # 内容目录（导航入口）
│   ├── log.md                  # 演进时间线 (摄入/查询/体检)
│   ├── overview.md             # 系统功能全景综合页
│   ├── specs-released/         # 全量最新规格 (每模块一页)
│   ├── archive/<变更名>/       # 规格历史快照 (归档时生成)
│   └── prd/                    # PRD文档管理
├── openspec/changes/           # 变更生命周期管理
│   ├── active/                 # 进行中变更 (含原始需求草稿)
│   ├── approved/               # 已批准变更
│   └── archive/                # 已归档变更
├── knowledge/                  # 工程方法与记忆库 (HOW)
│   ├── rules/                  # 编码和架构规范
│   ├── memory/experience/      # 经验记忆 (踩坑/成功模式)
│   ├── history-code/           # 历史代码存档
│   └── history-test-cases/     # 历史测试存档
└── src/                        # 源代码
```

## 文档规范

- 所有文档使用中文撰写
- 文件格式以 `.md` 为主，需要时通过 `/sdd:prd-generator` 导出 `.docx`
- 变更命名使用 kebab-case（如 `user-management-v2`）
- 归档命名格式：`YYYYMMDD-<变更名>`

## 反馈机制（阶段3-5）

在代码实施过程中，如用户要求修改：
1. 所有修改归属当前变更（禁止创建新变更）
2. 修改后确认是否与 prototypes.html 一致
3. 一致后反向同步 spec 文档
4. 完成同步后方可进入归档
