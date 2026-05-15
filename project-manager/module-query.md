# Query - 查询项目内容模块

> 自然语言查询文件、实验状态、文件关系

## 功能清单

1. 解析查询意图
2. 读取项目记忆 CLAUDE_PROJECT.md
3. 执行各类查询（状态/来源/用途/关联）
4. 格式化返回结果

---

## 查询类型

### 1. 状态查询

**示例问题：**
- "哪些实验还没跑完？"
- "有多少个实验完成了？"
- "实验状态怎么样？"
- "显示所有 pending 的实验"

**查询逻辑：**

读取 CLAUDE_PROJECT.md 中的 `experiments` 列表，按状态过滤：

| 状态 | 含义 |
|------|------|
| pending | 待运行 |
| running | 运行中 |
| completed | 完成 |
| failed | 失败 |
| stale | 可能过期 |

**输出格式：**

```markdown
# 实验状态查询

| 实验 | 状态 | 输入 | 输出 | 最后更新 |
|------|------|------|------|----------|
| exp_001 | ✓ 完成 | 3 | 5 | 2026-05-10 |
| exp_002 | ⏸ 待运行 | 2 | 0 | 2026-05-08 |
| exp_003 | ⚠ 可能过期 | 3 | 5 | 2026-05-05 |
| exp_004 | ✗ 失败 | 3 | 0 | 2026-05-01 |

**统计:**
- 总计: 4 个实验
- 完成: 1 | 待运行: 1 | 过期: 1 | 失败: 1
```

---

### 2. 来源追溯查询

**示例问题：**
- "这张图是怎么来的？"
- "fig1.png 的数据源是什么？"
- "results.csv 是哪个实验产生的？"
- "show me the origin of this figure"

**查询逻辑：**

从 CLAUDE_PROJECT.md 的 `relationships` 中追溯：

```
目标文件 (fig1.png)
    ↓ appears_in
源文件 (papers/draft.tex)

    ↓ plotted_by (反向查询)
绘图脚本 (scripts/plot.py)
    ↓ used_by (反向查询)
数据文件 (data/force.csv)
    ↓ used_by (反向查询)
仿真脚本 (experiments/exp_001/scripts/run.py)
```

**输出格式：**

```markdown
# 文件来源追溯

**文件:** figures/fig1_force_curve.png

```
fig1_force_curve.png
    └─ 出现在: papers/draft_v2.tex (Figure 1)
        └─ 绘制者: scripts/plot_force.m
            └─ 数据源: data/force_001.csv
                └─ 实验: exp_001
                    └─ 仿真脚本: experiments/exp_001/scripts/simulation.py
```

**完整链路:**
1. simulation.py → 生成 force_001.csv
2. plot_force.m 读取 force_001.csv → 生成 fig1_force_curve.png
3. fig1_force_curve.png 在 draft_v2.tex 中作为 Figure 1 引用
```

---

### 3. 用途查询

**示例问题：**
- "run_sim.py 是做什么的？"
- "这个脚本有什么用途？"
- "what does this file do"
- "data/params.yaml 是什么？"

**查询逻辑：**

从 CLAUDE_PROJECT.md 的 `files` 中查找文件信息：

```markdown
# 文件用途查询

**文件:** experiments/exp_001/scripts/run_sim.py

```
类型: simulation-script
实验: exp_001
用途: 主仿真脚本，用于求解动力学方程
最后修改: 2026-05-10
生成: result.csv, force.dat
被使用: plot_force.py (绘图)
```
```

---

### 4. 关联查询

**示例问题：**
- "draft.tex 引用了哪些图？"
- "exp_001 关联了哪些文件？"
- "找出所有与 fig1 相关的文件"
- "show all files related to this experiment"

**查询逻辑：**

从 `relationships` 中查找所有入边和出边：

```markdown
# 关联查询

**目标:** experiments/exp_001

```
exp_001/
    ├─ 仿真脚本: scripts/run_sim.py
    │   └─ 生成: output/result.csv, output/force.dat
    ├─ 输入文件:
    │   ├─ input/params.yaml
    │   ├─ input/initial_conditions.csv
    │   └─ data/geometry.mat
    ├─ 输出文件:
    │   ├─ output/result.csv
    │   ├─ output/force.dat
    │   └─ figures/fig1_force.png (通过 plot_force.py)
    └─ 出现在论文:
        └─ papers/draft.tex (Figure 1)
```

**关键关系:**
| 关系 | 源 → 目标 |
|------|-----------|
| generates | run_sim.py → result.csv |
| generates | run_sim.py → force.dat |
| plotted_by | result.csv → plot_force.py |
| appears_in | fig1_force.png → draft.tex |
```

---

### 5. 模式匹配查询

**示例问题：**
- "找出所有 .csv 文件"
- "有哪些图表？"
- "列出所有仿真脚本"
- "show me all figures in the project"

**查询逻辑：**

从 `files` 列表中按类型或模式过滤：

```markdown
# 模式匹配查询

**查询:** 所有 .csv 数据文件

| 文件 | 类型 | 实验 | 用途 |
|------|------|------|------|
| data/raw_001.csv | data-input | - | 原始数据 |
| experiments/exp_001/output/csv_result.csv | data-output | exp_001 | 仿真结果 |
| experiments/exp_002/input/params.csv | data-input | exp_002 | 参数配置 |
| results/processed.csv | data-output | - | 处理后数据 |

**统计:** 4 个文件
```

---

### 6. 问题查询

**示例问题：**
- "有哪些待处理的问题？"
- "显示所有孤儿文件"
- "哪些实验可能过期了？"

**查询逻辑：**

从 `unknowns` 和 `stale` 中获取：

```markdown
# 问题查询

## 待确认项 (unknowns)
- **temp_script.py**: 无法关联到任何实验 → 建议: 删除或移到 scripts/
- **data_backup.csv**: 可能是旧版本 → 建议: 确认后移到 archive/

## 过时警告 (stale)
- **exp_003**: 输入已更新（2026-05-10）但输出未刷新（2026-05-05）
  → 建议: 重新运行仿真

## 其他问题
- **重复文件**: fig1.png 出现在 fig/ 和 figures/ 两个位置

**总计:** 4 个待处理问题
```

---

## 通用查询处理

### 无法匹配时的处理

当无法确定查询类型时，执行通用搜索：

```markdown
无法确定查询类型，执行全文搜索...

**搜索关键词:** "{query}"

**匹配结果:**

### 文件名匹配
- experiments/exp_001/scripts/run_sim.py
- scripts/run_batch.py

### 备注匹配
- exp_002: "run_sim.py 的参数调整版本"
- fig1.png: "力曲线图，来自 exp_001"

**建议:** 如果这不是你想要的，请更具体地描述你的问题
```

### 无结果时的处理

```markdown
# 查询结果

**查询:** "xxx"

没有找到匹配结果。

**建议:**
- 检查拼写
- 尝试使用更通用的关键词
- 运行 `/project scan` 更新项目记忆后再试
```

---

## 查询参数

### 参数传递格式

```json
{
  "query": "哪些实验还没跑完",
  "query_type": "status",
  "filters": {
    "status": ["pending", "running"]
  }
}
```

### 查询类型自动识别

根据用户输入的关键词自动识别：

| 关键词 | 查询类型 |
|--------|----------|
| 哪些、几个、多少、有多少 | count/status |
| 怎么来的、来源、origin | origin |
| 做什么、用途、是什么 | usage |
| 关联、相关、关系 | relation |
| 找出、找到、列出 | pattern |
| 问题、警告、不确定、过期 | issues |
| （其他） | generic |

---

## 输出规范

所有查询结果统一格式：

```markdown
# 查询结果

**查询:** {用户原始输入}
**查询类型:** {type}
**时间:** {date}

---

{查询结果内容}

---

**提示:** 使用 `/project scan` 可以更新项目记忆，确保查询结果最新
`````````

---

## 注意事项

1. **读取记忆优先** - 查询主要基于 CLAUDE_PROJECT.md，不实际扫描文件
2. **无记忆时引导** - 如果项目没有 CLAUDE_PROJECT.md，提示用户先运行 scan
3. **追溯深度限制** - 来源追溯最多 5 层，避免无限循环
4. **缓存结果** - 查询结果可缓存，下次相同查询直接返回缓存