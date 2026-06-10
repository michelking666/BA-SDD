---
name: sdd:verify-change
description: SDD 阶段5：验证代码与原型一致性，生成验证报告
disable-model-invocation: true
---

# SDD Verify Change — 阶段5：验证变更

## 触发方式
用户输入 `/sdd:verify-change`

## 前置检查
- 确认 `openspec/changes/approved/<变更名>/change.md` 中阶段4已完成
- 如未完成阶段4，提示用户先运行 `/sdd:generate-tests`

## 执行步骤

### 1. 读取验证基准
- `openspec/changes/approved/<变更名>/prototypes.html` — 视觉基准
- `openspec/changes/approved/<变更名>/spec.md` — 功能基准
- `openspec/changes/approved/<变更名>/test-cases.md` — 测试用例
- `src/frontend/` — 前端实现代码
- `src/backend/` — 后端实现代码

### 2. 原型一致性检查

逐项对照 `prototypes.html` 检查前端实现：

| 检查项 | 原型描述 | 实现状态 | 差异说明 |
|--------|----------|----------|----------|
| 页面布局 | | | |
| 交互流程 | | | |
| 按钮/控件 | | | |
| 文案内容 | | | |
| 色彩规范 | | | |
| 错误状态 | | | |

### 3. 需求覆盖检查

逐条验证 `spec.md` 中的功能需求：

| 需求编号 | 需求描述 | 实现状态 | 备注 |
|----------|----------|----------|------|
| FR-001 | | ✅/❌/⚠️ | |

### 4. 测试用例执行状态检查
核对 `test-cases.md` 中测试用例的执行结果（人工确认或自动化测试结果）。

### 5. 生成验证报告（verification-report.md）

```markdown
# 验证报告：<变更名>

> 验证时间：<YYYY-MM-DD>
> 验证结论：✅ 通过 / ❌ 不通过 / ⚠️ 有条件通过

## 原型一致性：<通过/不通过>
<差异列表，如有>

## 需求覆盖率：<X/Y 条 (XX%)>
<未覆盖需求列表，如有>

## 测试执行情况
- 通过：X 个
- 失败：X 个
- 未执行：X 个

## 遗留问题
<如有遗留问题，列出并说明影响>

## 验证结论
<通过/不通过原因>
```

### 6. 处理验证失败
如验证不通过：
- 列出所有不一致项
- 回到阶段3修复（不创建新变更）
- 修复完成后重新运行 `/sdd:verify-change`

### 7. 验证通过后
更新 `change.md` 阶段状态：
```
- [x] 阶段5：验证变更 — <YYYY-MM-DD>（通过）
```
提示用户：下一步运行 `/sdd:archive-change` 进入阶段6（归档变更）
