# 智能路由模块

> 分析用户输入的意图，自动选择对应的模块执行

## 意图分析逻辑

### 关键词匹配规则

| 意图 | 关键词 | 模块 |
|------|--------|------|
| 初始化 | `init`, `初始化`, `新建项目`, `开始新项目`, `从头开始` | init |
| 整理 | `整理`, `重构`, `规范化`, `reorganize`, `organize` | scan → organize |
| 扫描 | `scan`, `复盘`, `审视`, `看看状态`, `现在怎样`, `项目情况` | scan |
| 查询 | `query`, `查询`, `找`, `哪些`, `什么`, `哪里`, `怎么来的`, `来源`, `关联` | query |
| 归档 | `archive`, `归档`, `结束项目`, `完成项目` | scan → archive |
| 运行完成 | `跑完`, `完成了`, `运行结束`, `simulate done` | scan → update |
| 新建实验 | `新建实验`, `add experiment`, `新实验` | organize |
| 默认 | 无法匹配 | scan + 状态报告 |

### 意图分类优先级

1. **初始化** - 如果提到新建项目，优先初始化
2. **查询** - 如果问问题，优先查询
3. **归档** - 如果提到结束/归档，先扫描确认状态
4. **整理/扫描** - 其他操作意图
5. **默认** - 无法识别时执行 scan → organize（完整整理）

## 路由决策流程

```
用户输入
    ↓
提取关键词（中文+英文）
    ↓
匹配意图分类
    ↓
返回模块列表
    ↓
加载并执行模块
```

## 输出格式

返回要执行的模块列表，格式：

```
[TASK_ROUTING]
modules: [module-scan, module-organize, update-memory]
intent: 整理项目
confidence: high
```

## 路由表

### 精确匹配

| 用户输入模式 | 模块 |
|--------------|------|
| `init`, `初始化` | [init] |
| `scan`, `复盘` | [scan] |
| `query`, `q.*` | [query] |
| `整理`, `organize` | [scan, organize] |
| `归档`, `archive` | [scan, archive] |

### 模糊匹配

以下关键词任意一个出现即匹配：

**init 匹配：**
- 新建、开始、初始化、创建项目

**scan 匹配：**
- 看看、怎样、现在、状态、情况、审视、复盘

**organize 匹配：**
- 整理、规范化、reorganize、重构

**query 匹配：**
- 哪些、什么、哪里、几个、几类、怎么来的、来源

**archive 匹配：**
- 归档、结束、结项、完成

## 参数传递

路由决策后，将用户原始输入作为上下文传递给第一个模块：

```json
{
  "user_input": "帮我整理下这个项目",
  "modules": ["scan", "organize"],
  "context": {
    "force_organize": true
  }
}
```

## 特殊场景

### 1. 双重意图
用户可能同时表达多个意图（例如"整理下项目，然后看看状态"）：
→ 按顺序执行所有匹配的模块

### 2. 隐含意图
用户说"这个数据是怎么来的"：
→ query 直接追溯关系图，不先执行 scan

### 3. 默认行为
无法确定意图时：
→ 执行 scan → organize → update（完整整理流程）

---

## 使用示例

### 示例 1
```
用户: "帮我整理下这个项目"
路由结果: [scan, organize, update-memory]
```

### 示例 2
```
用户: "哪些实验还没跑完？"
路由结果: [query]
参数: { query_type: "experiment_status", filter: "pending|running" }
```

### 示例 3
```
用户: "初始化一个新项目"
路由结果: [init]
参数: { create_structure: true, scan_existing: false }
```