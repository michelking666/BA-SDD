# 编码规范

> 适用范围：src/ 目录下所有代码

## 通用原则

1. **可读性优先**：代码首先是给人读的，其次才是给机器执行的
2. **最小修改原则**：只改动需求范围内的代码，不做顺手重构
3. **零注释原则**：好的命名不需要注释；必须注释时，只写"为什么"，不写"做什么"
4. **无防御性代码**：不为不可能发生的情况添加 if/else

## JavaScript / TypeScript

### 变量声明
- 优先使用 `const`，需要重新赋值时用 `let`，禁止 `var`
- 一个变量只做一件事，不复用变量存储不同语义的值

### 函数
- 函数长度：不超过 30 行；超过则拆分
- 参数：不超过 3 个；超过则用对象参数 `{ param1, param2, ... }`
- 返回值：尽早 return，避免深层嵌套

### 异步
- 统一使用 `async/await`，不混用 `.then()` 链式写法
- 错误处理只在边界层（API 层、UI 事件层），内部函数直接抛出

### TypeScript 特有
- 不使用 `any`，实在不知类型用 `unknown` + 类型守卫
- 接口（interface）用于描述对象形状；类型别名（type）用于联合类型/工具类型
- 优先使用 `interface` 定义 API 请求/响应结构

## React（如适用）

```typescript
// 组件：函数式，明确 props 类型
interface UserCardProps {
  userId: string
  onSelect: (id: string) => void
}

export function UserCard({ userId, onSelect }: UserCardProps) {
  // ...
}
```

- 状态提升：只在需要共享时提升，不过早抽取到全局
- useEffect 依赖数组必须完整，禁止 `// eslint-disable-next-line`
- 列表渲染必须有稳定的 `key`（不用 index）

## Python（如适用）

- 遵循 PEP 8
- 类型注解：函数参数和返回值必须有类型注解
- 异常：只捕获预期的异常类型，不使用裸 `except:`

## CSS / 样式

- 避免内联样式（特殊情况除外）
- 类名使用 `kebab-case`
- 颜色值统一使用 CSS 变量，不硬编码十六进制
- 避免 `!important`

## 禁止事项

以下情况需在 code review 中阻止：

- `console.log` 遗留在生产代码中
- 密码、Token、API Key 硬编码在代码里
- 注释掉的代码块（直接删除，git 有历史）
- 超过 200 行的单个文件（考虑拆分）
- 循环内部的数据库查询（N+1 问题）
