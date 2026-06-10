# SDD 框架 · 新手指南

> 一个**纯 Claude Code Skills** 构成的软件设计文档（SDD）管理框架。
> 本目录下每个 `sdd-*/` 文件夹就是一个 skill，通过 `/sdd:<名字>` 在 Claude Code 中调用。

## 这是什么

SDD 把"需求 → 原型 → 规格 → 实施 → 测试 → 验证 → 归档"的全过程结构化，并用一个**持续累积的知识库 llmwiki** 沉淀"系统是什么"。它解决的核心问题：需求散落、规格与代码脱节、历史决策无处可查。

整个框架不依赖任何外部程序——所有能力都封装在 skill 里，目录结构和文档由 skill 自动生成与维护。

## 5 分钟上手

```
1. /sdd:init      # 在空项目里一键生成目录骨架与种子文档
2. /sdd:new       # 创建一个变更，定义目标与范围
3. /sdd:propose   # 生成原型+规格+提案+任务+计划，等你审批
4. /sdd:apply-change   # 按原型实施（前端优先）
5. /sdd:generate-tests # 基于规格生成测试
6. /sdd:verify-change  # 验证代码与原型一致
7. /sdd:archive-change # 归档，并把规格"摄入"llmwiki
```

第一次用，先跑 `/sdd:init`。之后每个需求都是一轮 `new → … → archive-change` 的循环。

## 命令全景

| 命令 | 阶段 | 作用 |
|------|------|------|
| `/sdd:init` | 0 | 初始化框架——生成 llmwiki/openspec/knowledge/src 目录与种子文档 |
| `/sdd:new` | 1 | 创建变更——定义名称（kebab-case）、目标、范围 |
| `/sdd:propose` | 2 | 提案评审——生成可点击原型、需求规格、提案、任务分解、实施计划 |
| `/sdd:apply-change` | 3 | 实施变更——前端优先，严格匹配 prototypes.html |
| `/sdd:generate-tests` | 4 | 生成测试——基于需求规格产出测试用例 |
| `/sdd:verify-change` | 5 | 验证变更——核对代码与原型/规格一致性，出验证报告 |
| `/sdd:archive-change` | 6 | 归档变更——归档变更包，并把规格编译进 llmwiki（摄入）|
| `/sdd:explore` | — | 探索模式——分析想法可行性，引导进入正式变更流程 |
| `/sdd:wiki-lint` | — | wiki 体检——查 llmwiki 的矛盾/陈旧/孤立页，保持知识库健康 |
| `/sdd:reverse-engineer` | — | 逆向文档工程——对无源码的已有系统抓取内容，生成结构化功能文档 |
| `/sdd:prd-generator` | 辅助 | 基于规格生成 Word 格式 PRD |
| `/sdd:prd-validate` | 辅助 | 验证 PRD 与需求规格的契合度 |
| `/sdd:create-approval-record` | 辅助 | 创建或更新变更审批记录 |

## 两个核心阶段衔接规则

1. **前一阶段未完成，禁止进入下一阶段**——保证每步都有依据。
2. **同一时刻只允许一个进行中的变更**（active/ 目录唯一）——避免并行变更互相污染。

变更生命周期在 `openspec/changes/` 里流转：`active/`（进行中）→ `approved/`（阶段2审批后）→ `archive/`（阶段6归档后）。

## 两个知识库：llmwiki vs knowledge

框架把"知识"分成边界清晰、不重叠的两类：

- **llmwiki = 需求事实库（WHAT）**：回答"系统是什么"。存全量最新规格、历史快照、对外 PRD。由 `/sdd:propose`、`/sdd:explore` 读取以了解现状。
- **knowledge = 工程方法与记忆库（HOW）**：回答"该怎么做、做过怎样"。存编码/架构规范、踩坑与成功模式、历史代码与测试存档。

一句话：**规格去 llmwiki，规矩和记忆去 knowledge。**

### llmwiki 是累积的，不是临时检索

llmwiki 不是"查询时临时拼凑"的 RAG，而是**持续累积、自我维护的产物**，围绕三个操作运转：

- **Ingest 摄入**（`/sdd:archive-change`）：归档时把变更的最终规格**编译进** wiki——更新模块规格页、修订全景综合页、刷新交叉引用、生成不可变快照。
- **Query 查询**（`/sdd:propose`、`/sdd:explore`）：先读 `index.md` 定位相关规格，再钻入细节，避免盲扫；有价值的探索结论可**回填**。
- **Lint 体检**（`/sdd:wiki-lint`）：定期健康检查，防止知识库随规模增长而腐化。

详细规程见初始化后生成的 `llmwiki/README.md`。

## 目录结构（init 后生成）

```
项目根/
├── .claude/skills/      # 本框架本体（你正在看的目录）
├── CLAUDE.md            # 项目级说明（命令表+规则），供 Claude 读取
├── llmwiki/             # 需求事实库（WHAT，累积式 wiki）
│   ├── README.md        #   wiki schema：结构/约定/维护规程
│   ├── index.md         #   内容目录（导航入口）
│   ├── log.md           #   演进时间线（摄入/查询/体检）
│   ├── overview.md      #   系统功能全景（含 mermaid 依赖图/流转图）
│   ├── specs-released/  #   全量最新规格（每模块一页）
│   ├── archive/         #   规格历史快照（归档时生成）
│   └── prd/             #   PRD 文档管理
├── openspec/changes/    # 变更生命周期：active/ approved/ archive/
├── knowledge/           # 工程方法与记忆库（HOW）
│   ├── rules/           #   编码/架构规范
│   ├── memory/          #   经验记忆（踩坑/成功模式）
│   └── history-*/       #   历史代码与测试存档
└── src/                 # 源代码
```

## 可视化：Obsidian 友好

llmwiki 就是普通 markdown 文件夹，**直接用 Obsidian「打开文件夹作为 vault」指向 `llmwiki/` 即可**：

- **图谱视图**：规格页交叉引用是标准 markdown 链接，Obsidian 自动构建关系图，一眼看出枢纽页与孤立页。
- **mermaid 图**：`overview.md` 含全局模块依赖图与数据/状态流转图；规格页可含本模块流程图。Obsidian 与 GitHub 均原生渲染，无需插件。
- **标签/别名检索**：frontmatter 的 `tags`、`aliases` 支持按领域筛选、用中文名跳转。

> 链接刻意用标准 markdown 而非 `[[wikilink]]`，以保证在 GitHub 等渲染器中同样可点。浏览建议只读，规格写入统一由 SDD skill（归档摄入）完成。

## 文档规范

- 所有文档使用**中文**撰写。
- 文件以 `.md` 为主，需要时通过 `/sdd:prd-generator` 导出 `.docx`。
- 变更命名用 **kebab-case**（如 `user-management-v2`）。
- 归档命名格式：`YYYYMMDD-<变更名>`。

## 阶段3-5 的反馈机制

实施过程中如需修改：① 所有修改归属当前变更（禁止另起新变更）；② 修改后确认与 `prototypes.html` 一致；③ 一致后反向同步 spec 文档；④ 同步完成方可归档。

## 常见问题

- **从零开始？** → `/sdd:init`，然后 `/sdd:new` 开第一个变更。
- **只想评估一个点子值不值得做？** → `/sdd:explore`，它会判断可行性并在合适时引导你进入 `/sdd:new`。
- **接手一个没有源码/文档的老系统？** → `/sdd:reverse-engineer` 先把现状抓成结构化文档。
- **wiki 用久了感觉乱？** → 每隔几次归档跑一次 `/sdd:wiki-lint` 体检。
- **每个 skill 具体怎么干活？** → 打开对应 `sdd-*/SKILL.md`，里面有完整的触发方式、前置检查与执行步骤。
