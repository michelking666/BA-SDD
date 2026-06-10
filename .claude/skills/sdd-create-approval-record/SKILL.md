---
name: sdd:create-approval-record
description: SDD 辅助：创建或更新变更审批记录
disable-model-invocation: true
---

# SDD Create Approval Record — 创建审批记录

## 触发方式
用户输入 `/sdd:create-approval-record`

## 适用场景
- 变更提案需要正式审批记录
- 补充或更新已有变更的审批状态
- 为合规性或流程要求生成审批文档

## 执行步骤

### 1. 确认审批对象
询问用户：
- 哪个变更需要审批记录？
- 审批类型：变更立项 / 提案批准 / 实施授权 / 上线批准
- 审批人姓名/角色
- 审批结论：批准 / 有条件批准 / 驳回
- 审批意见（可选）

### 2. 创建或更新审批记录

在变更目录下创建/更新 `approval-record.md`：

```markdown
# 审批记录：<变更名>

---

## 审批记录 #<序号>

**审批类型**：<变更立项 / 提案批准 / 实施授权 / 上线批准>
**审批时间**：<YYYY-MM-DD>
**审批人**：<姓名/角色>
**审批结论**：✅ 批准 / ⚠️ 有条件批准 / ❌ 驳回

**审批意见**：
<审批人意见>

**附加条件**（如有条件批准）：
- <条件1>
- <条件2>

**关联阶段**：阶段<X>：<阶段名称>

---
```

文件路径：
- 进行中：`openspec/changes/active/<变更名>/approval-record.md`
- 已批准：`openspec/changes/approved/<变更名>/approval-record.md`

### 3. 如有附加条件
将审批附加条件添加到 `tasks.md` 的待办任务列表中：
```
- [ ] APPROVAL-001 <满足审批条件的具体任务>
```

### 4. 确认完成
告知用户：
- 审批记录已创建/更新：`<路径>/approval-record.md`
- 如有附加条件任务，已添加到 `tasks.md`
