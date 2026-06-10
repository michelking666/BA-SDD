# 命名规范

> 适用范围：SDD 框架内所有文件命名、变量命名、ID 命名

## 文件和目录命名

### 变更名称
- 格式：`kebab-case`（全小写，单词间用连字符）
- 长度：3-5 个英文单词为宜
- 示例：`user-auth-v2`、`order-export-pdf`、`dashboard-charts`
- 禁止：下划线、驼峰、中文、特殊字符

### 归档命名
- 格式：`YYYYMMDD-<变更名>`
- 示例：`20240315-user-auth-v2`

### 规格文件
- 格式：`<模块名>-spec.md`
- 示例：`user-management-spec.md`、`payment-spec.md`

### PRD 文件
- 格式：`<变更名>-prd.md`

## 需求编号规范

在单个变更内：

| 类型 | 格式 | 起始 |
|------|------|------|
| 功能需求 | FR-001, FR-002... | FR-001 |
| 非功能需求 | NFR-001... | NFR-001 |
| 业务规则 | BR-001... | BR-001 |
| 测试用例（功能） | TC-F001... | TC-F001 |
| 测试用例（边界） | TC-B001... | TC-B001 |
| 测试用例（异常） | TC-E001... | TC-E001 |
| 任务（前端） | FE-001... | FE-001 |
| 任务（后端） | BE-001... | BE-001 |
| 任务（测试） | TEST-001... | TEST-001 |
| 验收标准 | AC-001... | AC-001 |

## 代码命名规范

### 通用原则
- 变量/函数：`camelCase`（JS/TS）或 `snake_case`（Python）
- 类/组件：`PascalCase`
- 常量：`UPPER_SNAKE_CASE`
- 文件：`kebab-case`（组件文件用 `PascalCase`）

### 前端（React/Vue）
```
组件文件：UserProfile.tsx / UserProfile.vue
工具函数：utils/format-date.ts
样式文件：user-profile.module.css
测试文件：UserProfile.test.tsx
```

### 后端（Node.js/Python）
```
路由文件：user-routes.js / user_routes.py
服务文件：user-service.js / user_service.py
模型文件：User.js / user_model.py
测试文件：user.test.js / test_user.py
```

### API 路径
- 格式：`/api/v<版本号>/<资源复数名>`
- 示例：`/api/v1/users`、`/api/v1/orders/:id`
- 动作用 HTTP 方法表达（GET/POST/PUT/DELETE），避免在路径中使用动词

### 数据库
- 表名：`snake_case` 复数（`user_profiles`、`order_items`）
- 列名：`snake_case`（`created_at`、`user_id`）
- 索引：`idx_<表名>_<列名>`

## 原型 HTML ID 命名

在 `prototypes.html` 中：
- 页面 section：`id="page-<页面名>"`（如 `page-login`、`page-dashboard`）
- 组件：`id="<组件类型>-<功能名>"`（如 `modal-confirm`、`form-user-create`）
