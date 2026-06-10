---
name: sdd:archive-change
description: SDD 阶段6：归档变更并同步 llmwiki，清理 active/approved 目录
disable-model-invocation: true
---

# SDD Archive Change — 阶段6：归档变更

## 触发方式
用户输入 `/sdd:archive-change`

## 前置检查
- 确认 `openspec/changes/approved/<变更名>/change.md` 中阶段5已完成且验证通过
- 如未完成阶段5，提示用户先运行 `/sdd:verify-change`

## 执行步骤

> 本阶段在 SDD 中等价于向 llmwiki **摄入（Ingest）一份新事实源**：不只是复制文件，而是把本次变更的规格**编译进** wiki，刷新综合层与导航。详见 `llmwiki/README.md` 的"三个操作"。

### 1. 确定归档名称
归档命名格式：`<YYYYMMDD>-<变更名>`，例如：`20240315-user-auth-v2`

### 2. 归档变更文件

将 `openspec/changes/approved/<变更名>/` 复制到：
`openspec/changes/archive/<YYYYMMDD-变更名>/`

更新归档中的 `change.md` 最终状态：
```
- [x] 阶段6：归档变更 — <YYYY-MM-DD>（归档完成）
```

### 3. 同步 llmwiki/specs-released/（编译规格）

将本次变更涉及的规格编译进 `llmwiki/specs-released/`：
- 如果规格文件已存在，**更新**（合并本次变更，保留最新全量规格）
- 如果是新规格文件，**新增**
- 文件命名：`<模块名>-spec.md`
- **必须遵循 `spec-page-template`**（见 sdd-init 模板）：包含 YAML frontmatter（module / summary / status / aliases / tags / source_changes / related_modules / last_updated）与"交叉引用"小节。其中 `tags` 填领域标签、`aliases` 填模块中文名/缩写，供 Obsidian 标签检索与跳转

同步规则：
- 以 `spec.md` 中的最终版本为准
- 在 `source_changes` 追加本次 `<YYYYMMDD-变更名>`，更新 `last_updated`
- 若本次规格与既有规格页存在冲突，在对应位置**标记冲突**并以最新为准

### 4. 刷新综合层与交叉引用

- **overview.md**：新模块入"功能模块全景"表；若模块关系/数据流有变化，更新"模块关系"与"演进脉络"
- **overview.md 的 mermaid 图（必须同步）**："模块关系"小节维护两张 mermaid 图，每次摄入都要随模块/数据流变化更新：
  - **模块依赖图**（` ```mermaid ` + `graph LR`）：节点为各模块与共享数据实体（用 `[(表名)]` 圆柱表示）、外部角色（用 `([角色])` 圆角表示）；有向边标注关系（如"写入""读取/筛选""依赖"）。新模块归档时新增其节点与连边。
  - **数据/状态流转图**（`flowchart LR`）：画核心业务对象随模块流转的路径与状态变化；若本次变更引入状态机（如 pending→processing→done），用判定/分支节点表达，并在图下方备注单向/约束规则及对应 BR 编号。
  - 图必须与"功能模块全景"表、各规格页"交叉引用"保持一致（`/sdd:wiki-lint` 会比对）。Obsidian 与 GitHub 均原生渲染 mermaid，无需插件。
- **交叉引用**：为受影响的相关模块补/改交叉引用链接（被依赖方的"被依赖"小节也要回填），避免产生孤立页
- **概念页（按需）**：若出现多个规格共享的术语或数据模型，创建/更新 `glossary.md`、`data-model.md`

> mermaid 图示例（双模块、含共享表与状态流转），可作为格式参考：
> ```mermaid
> graph LR
>     visitor([外部角色]) -->|动作| modA[模块A]
>     modA -->|写入| db[(共享表)]
>     db -->|读取| modB[模块B]
>     modB -->|更新状态| db
> ```

### 5. 创建 llmwiki 历史快照

在 `llmwiki/archive/<YYYYMMDD-变更名>/` 下保存本次变更的完整快照（不可变）：
```
llmwiki/archive/<YYYYMMDD-变更名>/
├── change-summary.md     # 变更摘要
├── spec-snapshot.md      # 规格快照
└── verification-report.md # 验证报告副本
```

**change-summary.md** 格式：
```markdown
# 变更摘要：<变更名>

> 归档时间：<YYYY-MM-DD>
> 变更周期：<开始日期> → <归档日期>

## 变更目标
<摘自 change.md>

## 核心变更内容
<简要描述实际完成的变更>

## 影响范围
<列出变更涉及的文件/模块>

## 关联文档
- 完整变更包：`openspec/changes/archive/<YYYYMMDD-变更名>/`
- 规格最新版：`llmwiki/specs-released/<模块名>-spec.md`
```

### 6. 更新导航文件（index.md + log.md）

**index.md** — 更新"模块规格"表中本次涉及模块的条目（摘要、源变更数 +1、相关模块、最后更新）；新模块则新增一行。

**log.md** — 追加一条摄入记录：
```
## [<YYYY-MM-DD>] ingest | <变更名>
摄入 <模块> 规格；触及页面：<列出更新的 wiki 页>；<一句关键结论或冲突说明>。
```

### 7. 清理 active/ 目录
确认以下操作：
- `openspec/changes/approved/<变更名>/` 目录已归档 → **删除**
- `openspec/changes/active/<变更名>/` 目录 → **删除**

删除前向用户确认：
> 即将删除 active/ 和 approved/ 中的变更目录，归档已保存至 archive/。确认继续？

### 8. 归档完成确认
输出归档摘要：
- 归档路径：`openspec/changes/archive/<YYYYMMDD-变更名>/`
- 规格已编译：`llmwiki/specs-released/` 下 X 个文件
- 综合层已刷新：overview.md（含 mermaid 依赖图/流转图）/ index.md（+ 受影响交叉引用）
- 历史快照：`llmwiki/archive/<YYYYMMDD-变更名>/`
- log.md 已追加 ingest 记录
- active/ 和 approved/ 已清理

告知用户：变更 `<变更名>` 已完整归档并摄入 wiki，可通过 `/sdd:new` 开始下一个变更。建议每隔几次归档运行 `/sdd:wiki-lint` 做一次健康体检。
