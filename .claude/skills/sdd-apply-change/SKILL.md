---
name: sdd:apply-change
description: SDD 阶段3：按原型实施变更，前端优先，严格匹配 prototypes.html
disable-model-invocation: true
---

# SDD Apply Change — 阶段3：实施变更

## 触发方式
用户输入 `/sdd:apply-change`

## 前置检查
- 检查 `openspec/changes/approved/` 下是否有已批准的变更
- 读取 `change.md` 确认阶段2已完成（状态为"已批准"）
- 如未批准，提示用户先运行 `/sdd:propose`

## 执行步骤

### 1. 读取实施上下文
读取以下文件：
- `openspec/changes/approved/<变更名>/spec.md` — 需求规格
- `openspec/changes/approved/<变更名>/prototypes.html` — 原型（实施的唯一视觉基准）
- `openspec/changes/approved/<变更名>/plan.md` — 实施计划
- `openspec/changes/approved/<变更名>/tasks.md` — 任务清单
- `knowledge/rules/coding-standards.md` — 编码规范
- `knowledge/rules/architecture-spec.md` — 架构规范

### 2. 实施原则

**前端优先原则**：
- 严格匹配 `prototypes.html` 的视觉和交互设计
- 每个界面实现后与原型逐项对比
- 不得擅自调整原型中已定义的布局、颜色、交互流程

**代码质量原则**：
- 遵循 `knowledge/rules/coding-standards.md` 规范
- 不添加超出需求范围的功能
- 保持代码简洁，避免过度抽象

### 3. 逐任务实施
按 `tasks.md` 中的任务顺序执行，每完成一个任务：
- 在 `tasks.md` 将对应任务从"待办"移至"已完成"
- 在 `active/<变更名>/change.md` 同步更新进度

### 4. 处理用户反馈（重要）

实施过程中，如用户要求修改：

1. **所有修改归属当前变更**，禁止创建新变更
2. 修改完成后询问用户："此修改是否与 `prototypes.html` 一致？"
   - **一致**：继续实施
   - **不一致**：先更新 `prototypes.html` 和 `spec.md`，再实施代码
3. 代码与原型一致后，反向同步到 `spec.md`（确保规格与实现匹配）

### 5. 实施完成确认
所有任务完成后：
- 更新 `change.md` 阶段状态：
  ```
  - [x] 阶段3：实施变更 — <YYYY-MM-DD>
  ```
- 同步更新 `approved/` 目录下对应文件（与 `active/` 保持一致）
- 提示用户：下一步运行 `/sdd:generate-tests` 进入阶段4
