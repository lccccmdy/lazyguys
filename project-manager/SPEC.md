# LazyGuys - Project Manager

> 让 AI 自动管理仿真/科研项目，最大程度省心

## 1. Overview

**Purpose:** 自动整理仿真项目文件，建立项目记忆，让 agent 每次工作都理解项目上下文。

**Core Features:**
- 文件结构标准化（初始化/重构）
- 主动追踪实验状态（检测过期/孤儿文件）
- 自动建立文件关系图谱
- 自然语言查询项目内容
- 项目记忆自动更新

**Target Users:** 动力学仿真/后处理/论文绘图等工程科研工作流

---

## 2. Architecture

```
lazyguys/
├── project-manager/
│   ├── README.md              # 说明文档
│   ├── CLAUDE.md              # Skill 自身说明
│   ├── skill.md               # Skill 实现（主要逻辑）
│   ├── templates/
│   │   └── default-structure.yaml   # 内置目录模板
│   └── rules/
│       └── file-types.yaml    # 文件类型识别规则
```

---

## 3. Skill Interface

**Command:** `/project` 或 `/project-reorganize`

### 3.1 Smart Routing

根据用户意图自动选择子命令：

| 用户输入 | 自动执行 |
|----------|----------|
| "看看项目状态" | scan + 状态报告 |
| "帮我整理下" | scan → organize（如果需要） |
| "这张图怎么来的" | query 查关系 |
| "新建实验" | organize 初始化 |
| "跑完了" | scan + update + 状态刷新 |
| "哪些实验还没跑" | query 获取状态 |

### 3.2 Sub-commands

**scan** —— 复盘项目
- 扫描所有文件
- 识别文件类型和用途
- 建立文件关系图谱
- 检测孤儿文件/冲突/过期结果
- 更新 `CLAUDE_PROJECT.md`

**organize** —— 整理结构
- 生成标准化目录结构
- 自动归类现有文件
- 重命名混乱文件（需确认）
- 标记无法归类的文件

**query** —— 查询项目
- 自然语言查询文件
- 查实验状态
- 查文件来源/用途
- 返回关联结果

**init** —— 初始化新项目
- 创建目录结构
- 生成 `CLAUDE_PROJECT.md`
- 配置 `.claude/settings.json`

---

## 4. File Types Detection

### 4.1 通用识别规则

| 类别 | 扩展名 | 识别关键词 |
|------|--------|------------|
| 仿真脚本 | .py, .m, .inp, .c, .cpp, .java | main, run, solve, sim |
| 数据输入 | .csv, .xlsx, .txt, .mat, .json, .yaml | input, param, config |
| 数据输出 | .out, .log, .txt, .dat, .h5, .hdf5 | output, result, log |
| 图表 | .png, .jpg, .svg, .eps, .pdf | fig, plot, chart |
| 论文 | .tex, .bib, .pdf | draft, paper, thesis |
| 文档 | .md, .doc, .docx, .txt | readme, note, doc |

### 4.2 关系推断规则

1. **基于文件名**
   - `run_*.py` → 输出 `results/*`
   - `fig*.png` ← 数据文件被绘图脚本生成

2. **基于文件内容**
   - 脚本中引用/读取的文件
   - LaTeX 中 `\includegraphics` 引用的图片

3. **基于目录结构**
   - 同目录下的相关文件
   - 父子目录的包含关系

---

## 5. Project Memory

**File:** `CLAUDE_PROJECT.md`（生成在项目根目录）

### 5.1 Format

```markdown
# 项目概览
- 项目名称: {自动检测或用户提供}
- 创建时间: {date}
- 最后更新: {date}
- 状态: 在研/结项/归档

# 实验列表
experiments:
  - name: exp_001
    purpose: [实验目的]
    status: pending/running/completed/failed
    created: {date}
    updated: {date}
    inputs: [file1, file2]
    outputs: [result1.csv, fig1.png]
    notes: [备注]

# 文件索引
files:
  - path: simulations/run.py
    type: simulation-script
    experiment: exp_001
    purpose: 主仿真脚本
    outputs: [result.csv]
    last_modified: {date}

  - path: data/raw.csv
    type: data-input
    purpose: 原始实验数据
    used_by: [run.py, analysis.py]

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
    target: papers/draft.tex (Figure 1)

# 不确定项
unknowns:
  - file: orphan_file.dat
    reason: 无法关联到任何实验
    suggested_action: "关联到 exp_001？" / "删除？"

# 过时检测
stale:
  - experiment: exp_002
    warning: "输出文件比输入文件旧，可能需要重新运行"
    inputs_updated: {input_date}
    outputs_updated: {output_date}
```

### 5.2 状态规则

| 状态 | 含义 | 触发条件 |
|------|------|----------|
| pending | 待运行 | 目录存在，无输出 |
| running | 运行中 | 有锁文件或日志显示运行中 |
| completed | 完成 | 输出文件存在且未过期 |
| failed | 失败 | 日志显示错误或无正常输出 |
| stale | 可能过期 | 输出比输入旧 |

---

## 6. Active Tracking

### 6.1 自动检测

| 检测项 | 逻辑 |
|--------|------|
| 孤儿文件 | 无法通过关系图关联到任何实验 |
| 重复文件 | 同名但不同路径的文件 |
| 过时结果 | 输出文件 mtime < 输入文件 mtime |
| 实验状态 | 根据文件存在性和时间戳自动更新 |

### 6.2 检测到问题时的处理

1. **孤儿文件** → 标记到 `unknowns`，询问用户关联/删除
2. **过时结果** → 在报告中提示，用户确认后重新运行
3. **重复文件** → 提示用户选择保留哪个
4. **实验失败** → 标记状态，提示查看日志

---

## 7. Global Strategy

**File:** `.claude/settings.json`（项目根目录，由 skill 自动创建）

```json
{
  "hooks": {
    "onAgentStart": [
      "Read CLAUDE_PROJECT.md if exists to understand project context"
    ],
    "onAgentEnd": [
      "Update CLAUDE_PROJECT.md with any new findings, created files, or changed relationships"
    ]
  }
}
```

---

## 8. Template

**File:** `templates/default-structure.yaml`

```yaml
name: "默认仿真项目结构"
description: "适用于动力学仿真/后处理/论文绘图工作流"

directories:
  - path: experiments/
    description: "按实验编号组织的仿真实验"
  - path: data/
    description: "原始数据和外部输入"
  - path: results/
    description: "处理结果和中间文件"
  - path: figures/
    description: "生成的图表和图片"
  - path: papers/
    description: "论文相关的 LaTeX 和文档"
  - path: scripts/
    description: "工具脚本和自动化程序"
  - path: docs/
    description: "项目文档和说明"

files:
  - path: CLAUDE_PROJECT.md
    description: "项目记忆（自动生成）"
    auto_generated: true
```

---

## 9. Implementation Notes

- Skill 使用 PowerShell 兼容的路径和命令
- 所有 AI 交互通过自然语言，无需用户记忆命令
- 配置文件使用 YAML，保持人类可读
- 项目记忆 JSON/YAML 并存，Markdown 优先（人类友好）
- Skill 调用后自动决定是否询问用户确认

---

## 10. Usage Flow

```
1. 用户在新项目调用 /project init
   → 创建目录结构 + CLAUDE_PROJECT.md + .claude/settings.json

2. 用户进行仿真实验
   → 可能手动或自动调用 /project scan 更新状态

3. 用户询问项目状态
   → /project query "哪些还没跑完"

4. 每次 agent 工作
   → 开始时读 CLAUDE_PROJECT.md
   → 结束时更新 CLAUDE_PROJECT.md

5. 项目结束归档
   → /project archive 标记状态为归档
```