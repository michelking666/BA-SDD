---
name: sdd:generate-tests
description: SDD 阶段4：基于需求规格生成测试用例文档
disable-model-invocation: true
---

# SDD Generate Tests — 阶段4：生成测试

## 触发方式
用户输入 `/sdd:generate-tests`

## 前置检查
- 确认 `openspec/changes/approved/<变更名>/change.md` 中阶段3已完成
- 如未完成阶段3，提示用户先运行 `/sdd:apply-change`

## 执行步骤

### 1. 读取测试依据
- `openspec/changes/approved/<变更名>/spec.md` — 功能需求和验收标准
- `openspec/changes/approved/<变更名>/prototypes.html` — 交互流程
- `knowledge/history-test-cases/` — 历史测试案例参考

### 2. 生成测试用例文档

在 `openspec/changes/approved/<变更名>/` 下创建 `test-cases.md`：

```markdown
# 测试用例：<变更名>

> 生成时间：<YYYY-MM-DD>
> 基于规格版本：<spec.md 最后更新时间>

## 功能测试

### TC-F001：<功能名称>
- **前置条件**：
- **测试步骤**：
  1.
  2.
- **预期结果**：
- **对应需求**：FR-001

（按 spec.md 中每条 FR 生成对应测试用例）

## 边界测试

### TC-B001：<边界场景>
- **测试数据**：
- **预期结果**：

## 异常测试

### TC-E001：<异常场景>
- **触发条件**：
- **预期结果**：

## 回归测试清单
检查本次变更可能影响的已有功能：
- [ ] <受影响功能1>
- [ ] <受影响功能2>

## 验收标准核对
| 验收标准 | 测试用例 | 状态 |
|----------|----------|------|
```

### 3. 同步到 src/testcase/
将测试用例同步到：`src/testcase/<变更名>/test-cases.md`

同时将历史测试存档到：`knowledge/history-test-cases/<变更名>-<YYYY-MM-DD>.md`

### 4. 完成确认
更新 `change.md` 阶段状态：
```
- [x] 阶段4：生成测试 — <YYYY-MM-DD>
```
提示用户：下一步运行 `/sdd:verify-change` 进入阶段5（验证变更）
