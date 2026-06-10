# llmwiki 需求事实库（Schema）

> SDD 框架的需求事实中心，回答"系统是什么"。本文件即 wiki 的 **schema**：定义结构、约定与维护规程，让 LLM 成为有纪律的 wiki 维护者，而非每次从零检索的问答机器。

## 核心理念：累积，而非检索

llmwiki 不是"上传文档、查询时临时拼凑"的 RAG。它是一个**持续累积、自我维护的产物**：

- 每次变更归档（摄入）时，规格被**编译进** wiki —— 更新模块规格页、修订全景综合页、刷新交叉引用、记录与旧规格的冲突。
- 知识被编译一次后**保持更新**，而不是每次 `/sdd:propose` 重新推导。
- wiki 越用越富：交叉引用已就位、矛盾已标记、综合已反映所有已归档变更。

LLM 负责全部 wiki 维护（摘要、交叉引用、归档、记账）；人负责定方向、提问、决策。

## 与 knowledge 的边界

- **llmwiki（本目录）= WHAT，需求事实**：最新规格、规格历史快照、PRD、综合页。读者是 `/sdd:propose`、`/sdd:explore`。
- **knowledge = HOW，工程方法与记忆**：编码/架构规范（rules）、踩坑与成功模式（memory）、历史代码与测试存档（history-*）。

规格只存 llmwiki，knowledge 不再保留任何 spec 副本。原始需求草稿在变更生命周期内归 `openspec/changes/active/`。

## 目录结构

```
llmwiki/
├── index.md            # 内容目录：所有规格页与综合页的catalog（导航入口）
├── log.md              # 演进时间线：append-only，记录每次摄入/查询/体检
├── overview.md         # 综合页：系统功能全景与跨模块关系
├── specs-released/     # 模块规格（最新全量，每模块一个 <模块名>-spec.md）
├── archive/            # 规格历史快照（每次归档生成，不可修改）
│   └── <YYYYMMDD-变更名>/
│       ├── change-summary.md
│       ├── spec-snapshot.md
│       └── verification-report.md
└── prd/
    ├── active/         # 活跃中的 PRD 源文件（Markdown）
    └── output/         # 导出的 .docx 文件
```

按需涌现（首次相关内容出现时由 LLM 创建，不预建）：`glossary.md`（术语表）、`data-model.md`（跨模块数据模型）。

## 三个操作

llmwiki 的生命围绕三个操作运转，分别由对应 skill 承载：

### Ingest 摄入 — `/sdd:archive-change`（阶段6）
归档一个变更 = 向 wiki 摄入一份新事实源。LLM 应：
1. 把变更的最终规格编译进 `specs-released/<模块>-spec.md`（新建或更新，遵循 `spec-page-template`：带 frontmatter + 交叉引用）。
2. 修订 `overview.md` 全景：新模块入表、模块关系变化、演进脉络。
3. 刷新受影响的交叉引用（被依赖模块、术语、数据模型）。标记新规格与旧声明的冲突。
4. 更新 `index.md` 目录条目（摘要、源变更数、相关模块）。
5. 生成 `archive/<YYYYMMDD-变更名>/` 不可变快照。
6. 向 `log.md` 追加一条 `ingest` 记录。

一次摄入通常触及多个 wiki 页面，这正是"累积"的体现。

### Query 查询 — `/sdd:propose`、`/sdd:explore`
针对 wiki 提问/取上下文。LLM 应：**先读 `index.md` 定位相关页 → 再钻入具体规格页**，而非盲扫整个目录。
> 关键：有价值的查询结果应**回填**进 wiki。`/sdd:explore` 中产生的跨模块分析、新发现的关联，可沉淀为综合页或追加到 `overview.md`，并在 `log.md` 记 `query` 一条——让探索像摄入一样累积。

### Lint 体检 — `/sdd:wiki-lint`
定期健康检查，保持 wiki 不腐化。检查项见该 skill。每次体检向 `log.md` 追加 `lint` 一条。

## 页面约定

- **模块规格页**：遵循 `spec-page-template`（见 sdd-init 模板）。必须含 YAML frontmatter（module / summary / status / aliases / tags / source_changes / related_modules / last_updated）与"交叉引用"小节。
- **交叉引用写法**：统一用标准 markdown 链接 `[模块名](模块名-spec.md#FR-XXX)`。这样在 Obsidian 与 GitHub 等渲染器中都可点击。每条规格页至少有一条出链或入链，避免孤立页。
- **frontmatter 的 tags/aliases**：`tags` 供标签检索与 Dataview 仪表盘使用（如 `spec`、`auth`）；`aliases` 让中文名/缩写也能在 Obsidian 中被搜索和链接到。
- **mermaid 图**：`overview.md`"模块关系"维护全局**模块依赖图**与**数据/状态流转图**；规格页按需含本模块的**流程图/状态图**。统一用 ` ```mermaid ` 代码块，Obsidian 与 GitHub 均原生渲染、无需插件。图由 `/sdd:archive-change` 摄入时同步更新，`/sdd:wiki-lint` 检查其与模块全景/交叉引用是否一致。
- **index.md / log.md / overview.md**：见各自文件头部说明，均带导航类 tag（`wiki/index`、`wiki/log`、`wiki/overview`）。

## 文档流转关系

```
需求提出
    ↓
openspec/changes/active/<变更名>/    ← /sdd:new + /sdd:propose 生成
    ↓ (审批通过)
openspec/changes/approved/<变更名>/  ← 阶段2审批后复制
    ↓ (实施完成)
openspec/changes/approved/<变更名>/  ← 阶段3-5 持续更新
    ↓ (归档 = 摄入)
llmwiki/specs-released/<模块>-spec.md   ← 编译进最新规格
llmwiki/overview.md + index.md + log.md ← 修订综合层与导航
llmwiki/archive/<YYYYMMDD-变更名>/       ← 生成不可变快照
openspec/changes/archive/<YYYYMMDD-变更名>/  ← 完整变更包
```

## 读取规格的场景

| 场景 | 读取位置 |
|------|----------|
| 开始新提案（定位相关规格） | 先 `index.md`，再钻入 `specs-released/` |
| 了解系统全貌 | `overview.md` |
| 查阅某功能的历史实现 | `archive/<YYYYMMDD-变更名>/` |
| 了解 wiki 最近发生了什么 | `log.md` |
| 生成 PRD | `openspec/changes/approved/<变更名>/spec.md` |

> 历史变更时间线统一记于 `log.md`（取代旧版 README 末尾的历史表）。

## 在 Obsidian 中查看

llmwiki 就是一个普通的 markdown 文件夹，**直接用 Obsidian"打开文件夹作为 vault"指向 `llmwiki/` 即可**，无需任何转换。它对团队成员浏览很友好：

- **图谱视图（Graph view）**：交叉引用是标准 markdown 链接，Obsidian 会自动构建关系图，一眼看出哪些规格是枢纽、哪些是孤立页（孤立页正是 `/sdd:wiki-lint` 要揪出的）。
- **反向链接面板（Backlinks）**：打开任一规格页，右侧能看到"谁引用了我"，即被依赖关系。
- **标签搜索**：用 `tag:#spec` 或 `tag:#auth` 快速筛选；导航页带 `wiki/index`、`wiki/log`、`wiki/overview` 标签。
- **快速跳转**：`aliases` 让你用中文模块名或缩写也能在 `Ctrl/Cmd+O` 里直接跳转。

推荐插件（可选，不装也能正常浏览）：
- **Dataview**：基于 frontmatter 生成动态表格。例如在一个页面写 Dataview 查询，自动列出所有 `status: active` 的规格及其 `last_updated`，免去手动维护清单。
- **Marp**（可选）：若需从规格直接生成评审用幻灯片。

注意事项：
- 链接刻意采用标准 markdown 而非 Obsidian 的 `[[wikilink]]`，以保证在 GitHub 等其他渲染器中同样可点。Obsidian 两种都支持，无需改动。
- wiki 本身是 git 仓库的一部分，版本历史、协作天然具备。多人浏览建议只读，规格的写入仍统一由 SDD skill（归档摄入）完成，避免手改导致与变更流程脱节。
