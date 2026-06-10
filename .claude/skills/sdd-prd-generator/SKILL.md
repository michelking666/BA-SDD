---
name: sdd:prd-generator
description: SDD 辅助：基于需求规格生成 Word 格式 PRD 文档
disable-model-invocation: true
---

# SDD PRD Generator — 生成PRD文档

## 触发方式
用户输入 `/sdd:prd-generator`

## 适用场景
基于已完成的需求规格，生成符合业务交付标准的 PRD（产品需求文档）。

## 执行步骤

### 1. 确定 PRD 来源
询问用户：
- 基于哪个变更的规格？（提供变更名）
- PRD 的受众是谁？（开发团队 / 业务干系人 / 外部客户）
- 是否需要包含原型截图说明？

### 2. 读取规格依据
- `openspec/changes/approved/<变更名>/spec.md`
- `openspec/changes/approved/<变更名>/prototypes.html`
- `openspec/changes/approved/<变更名>/proposal.md`

### 3. 生成 PRD Markdown 文档

在 `llmwiki/prd/active/<变更名>-prd.md` 生成：

```markdown
# 产品需求文档（PRD）

**项目名称**：<项目名>
**文档版本**：v1.0
**创建日期**：<YYYY-MM-DD>
**作者**：<作者>
**状态**：草稿 / 评审中 / 已批准

---

## 1. 文档说明
### 1.1 目的
### 1.2 范围
### 1.3 定义与缩写

## 2. 背景与目标
### 2.1 业务背景
### 2.2 问题描述
### 2.3 产品目标
### 2.4 成功指标

## 3. 用户分析
### 3.1 目标用户
### 3.2 用户故事

## 4. 功能需求
### 4.1 <功能模块1>
#### 4.1.1 功能描述
#### 4.1.2 详细需求
#### 4.1.3 业务规则
#### 4.1.4 界面说明（引用原型）

（按模块展开，每条 FR 对应章节）

## 5. 非功能需求
### 5.1 性能要求
### 5.2 安全要求
### 5.3 兼容性要求

## 6. 数据需求
### 6.1 数据模型
### 6.2 数据迁移（如有）

## 7. 接口需求
### 7.1 内部接口
### 7.2 外部接口

## 8. 约束条件
### 8.1 技术约束
### 8.2 业务约束

## 9. 验收标准
（对应 spec.md 中的验收标准）

## 10. 附录
### 10.1 原型说明
### 10.2 变更历史
```

### 4. 导出说明
告知用户：
- PRD Markdown 已生成：`llmwiki/prd/active/<变更名>-prd.md`
- 如需导出为 Word 格式，请使用 Pandoc：
  ```bash
  pandoc llmwiki/prd/active/<变更名>-prd.md -o llmwiki/prd/output/<变更名>-prd.docx
  ```
- 或者使用 `/sdd:prd-validate` 先验证 PRD 质量再导出
