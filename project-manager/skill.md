---
name: project-manager
description: "AI辅助项目管理：自动整理仿真项目、建立项目记忆、追踪实验状态"
author: LazyGuys
version: 1.0.0
user-invocable: true
---

# LazyGuys - Project Manager

> 让 AI 自动管理仿真/科研项目，最大程度省心

## 使用方式

在项目目录中，直接用自然语言描述需求，例如：

- "帮我整理下这个项目"
- "看看项目现在什么状态"
- "这张图是怎么来的？"
- "哪些实验还没跑完？"
- "实验跑完了，帮我更新下"
- "初始化一个新项目"

我会根据你的意图自动选择对应的操作。

---

## 核心模块

| 模块 | 文件 | 功能 |
|------|------|------|
| 智能路由 | `module-router.md` | 分析意图，选择对应模块 |
| 扫描复盘 | `module-scan.md` | 扫描项目，识别类型，建立关系图谱 |
| 整理结构 | `module-organize.md` | 生成标准化目录，归类文件 |
| 初始化 | `module-init.md` | 创建项目结构，生成配置文件 |
| 查询 | `module-query.md` | 自然语言查询项目内容 |
| 更新记忆 | `update-memory.md` | 更新 CLAUDE_PROJECT.md |

---

## 意图识别规则

根据用户输入的关键词，自动选择操作：

| 意图关键词 | 自动执行 |
|------------|----------|
| 整理、重构、规范化 | scan → organize → update |
| 看看、状态、现在怎样 | scan + 状态报告 |
| 怎么来的、来源、关联 | query 查关系图 |
| 什么、哪些、有多少 | query 查询 |
| 新建、初始化、开始新项目 | init 初始化 |
| 跑完、完成 | scan + update + 状态刷新 |
| 归档、结束 | archive 标记归档 |
| 其他 | scan + 状态报告（默认） |

---

## 文件类型识别

### 扩展名分类

| 类型 | 扩展名 |
|------|--------|
| 仿真脚本 | `.py`, `.m`, `.inp`, `.c`, `.cpp`, `.java`, `.for`, `.sh`, `.bat` |
| 数据文件 | `.csv`, `.xlsx`, `.xls`, `.txt`, `.mat`, `.json`, `.yaml`, `.yml` |
| 仿真输出 | `.out`, `.log`, `.dat`, `.h5`, `.hdf5`, `.res`, `.rpt` |
| 图表 | `.png`, `.jpg`, `.jpeg`, `.svg`, `.eps`, `.pdf`, `.fig` |
| 论文 | `.tex`, `.bib` |
| 文档 | `.md`, `.doc`, `.docx`, `.pdf` |

### 类型推断关键词

| 类型 | 关键词 |
|------|--------|
| 仿真脚本 | run, main, solve, sim, simulate, batch, execute |
| 数据输入 | input, param, config, settings, raw, initial |
| 数据输出 | output, result, log, summary, report |
| 图表 | fig, plot, chart, graph, diagram |

---

## 关系类型

| 关系 | 含义 |
|------|------|
| `generates` | 脚本生成输出文件 |
| `plotted_by` | 数据被绘图脚本使用 |
| `appears_in` | 图表被论文引用 |
| `used_by` | 数据被脚本使用 |

---

## 实验状态

| 状态 | 含义 | 触发条件 |
|------|------|----------|
| `pending` | 待运行 | 目录存在，无输出 |
| `running` | 运行中 | 有锁文件或日志显示运行中 |
| `completed` | 完成 | 输出存在且比输入新 |
| `failed` | 失败 | 日志显示错误或无正常输出 |
| `stale` | 可能过期 | 输出比输入旧，需重新运行 |

---

## 问题检测

| 问题 | 检测逻辑 |
|------|----------|
| 孤儿文件 | 无法关联到任何实验的文件 |
| 重复文件 | 同名但不同路径 |
| 过时结果 | 输出文件 mtime < 输入文件 mtime |

检测到问题时，标记到 CLAUDE_PROJECT.md 的 `unknowns` 或 `stale` 中，询问用户处理。

---

## CLAUDE_PROJECT.md 格式规范

项目记忆文件，生成在项目根目录：

```markdown
# 项目概览
- 项目名称: {name}
- 创建时间: {date}
- 最后更新: {date}
- 状态: 在研/结项/归档

# 实验列表
experiments:
  - name: exp_001
    purpose: 实验目的
    status: pending/running/completed/failed/stale
    created: {date}
    updated: {date}
    inputs: [file1, file2]
    outputs: [result1.csv, fig1.png]
    notes: 备注

# 文件索引
files:
  - path: simulations/run.py
    type: simulation-script
    experiment: exp_001
    purpose: 主仿真脚本
    outputs: [result.csv]
    last_modified: {date}

# 文件关系图
relationships:
  - source: simulations/run.py
    type: generates
    target: data/result.csv
  - source: data/result.csv
    type: plotted_by
    target: figures/fig1.png
  - source: figures/fig1.png
    type: appears_in
    target: papers/draft.tex

# 不确定项
unknowns:
  - file: orphan_file.dat
    reason: 无法关联到任何实验
    suggested_action: 关联到 exp_001 / 删除

# 过时检测
stale:
  - experiment: exp_002
    warning: 输出文件比输入文件旧，可能需要重新运行
    inputs_updated: {date}
    outputs_updated: {date}
```

---

## .claude/settings.json 配置

自动创建在项目根目录的 `.claude/settings.json`：

```json
{
  "hooks": {
    "onAgentStart": [
      "Read CLAUDE_PROJECT.md if exists to understand project context"
    ],
    "onAgentEnd": [
      "Update CLAUDE_PROJECT.md with any new findings, created files, or changed relationships"
    ]
  },
  "settings": {
    "project记忆": "已启用，每次工作会自动读取和更新 CLAUDE_PROJECT.md"
  }
}
```

---

## 目录结构模板

来自 `templates/default-structure.yaml`：

```
{project}/
├── experiments/          # 按实验编号组织的仿真实验
│   └── exp_001/
│       ├── input/        # 输入文件
│       ├── output/       # 仿真输出
│       └── scripts/      # 实验专用脚本
├── data/                 # 原始数据和外部输入
├── results/              # 处理结果和中间文件
├── figures/              # 生成的图表和图片
├── papers/               # 论文相关的 LaTeX 和文档
├── scripts/              # 工具脚本和自动化程序
├── docs/                 # 项目文档和说明
├── archive/              # 已完成或废弃的实验归档
└── CLAUDE_PROJECT.md     # 项目记忆（自动生成）
```

---

## 执行流程

1. **用户输入** → 分析意图，匹配关键词
2. **路由决策** → 加载对应的模块文件
3. **执行操作** → 扫描/整理/查询/初始化
4. **更新记忆** → 调用 update-memory.md 更新 CLAUDE_PROJECT.md
5. **返回结果** → 格式化输出给用户

---

## 配置引用

- 目录模板：`templates/default-structure.yaml`
- 文件类型规则：`rules/file-types.yaml`
- 智能路由：`module-router.md`
- 扫描复盘：`module-scan.md`
- 整理结构：`module-organize.md`
- 初始化：`module-init.md`
- 查询：`module-query.md`
- 更新记忆：`update-memory.md`