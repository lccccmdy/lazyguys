# Scan - 项目复盘模块

> 扫描项目所有文件，识别类型，建立关系图谱，检测问题

## 功能清单

1. 扫描项目所有文件
2. 识别文件类型和用途
3. 建立文件关系图谱
4. 检测问题（孤儿文件/过期结果/重复文件）
5. 自动推断实验目录结构
6. 更新实验状态
7. 调用 update-memory 持久化到 CLAUDE_PROJECT.md

---

## 执行步骤

### Step 1: 文件扫描

使用 Glob 工具递归扫描项目，获取所有文件：

```
Glob: **/*
```

排除以下文件和目录：
- `.git/`
- `node_modules/`
- `__pycache__/`
- `*.pyc`
- `.DS_Store`
- `Thumbs.db`

记录每个文件：
- 路径 (relative path from project root)
- 文件名
- 扩展名
- 大小
- 修改时间 (last modified)

### Step 2: 类型识别

根据扩展名和文件名识别文件类型：

| 类型 | 扩展名 | 识别逻辑 |
|------|--------|----------|
| `simulation-script` | .py, .m, .inp, .c, .cpp, .for, .sh, .bat | 扩展名匹配 |
| `data-input` | .csv, .xlsx, .xls, .txt, .json, .yaml, .yml, .mat | 扩展名匹配 |
| `data-output` | .out, .log, .dat, .h5, .hdf5, .res, .rpt | 扩展名匹配 |
| `figure` | .png, .jpg, .jpeg, .svg, .eps, .pdf, .fig | 扩展名匹配 |
| `paper` | .tex, .bib | 扩展名匹配 |
| `documentation` | .md, .doc, .docx | 扩展名匹配 |

**用途推断：**

根据文件名关键词进一步推断具体用途：
- `*run*.py`, `*sim*.py` → 仿真脚本
- `*plot*.py`, `*visualize*.m` → 绘图脚本
- `*config*.yaml`, `*param*.json` → 配置文件
- `*input*.csv`, `*raw*.csv` → 原始数据
- `fig*`, `plot*` → 图表
- `*draft*.tex`, `*paper*.tex` → 论文草稿

### Step 3: 关系推断

建立文件之间的关系：

**3.1 基于目录结构的关系**

- `experiments/{name}/` 下的文件属于该实验
- 子目录 `input/`, `output/`, `scripts/` 内的文件用途明确

**3.2 基于文件名模式的关系**

| 模式 | 关系 |
|------|------|
| `run_*.py` + 同名 `.csv` | generates → result.csv |
| `plot_*.py` + `fig*.png` | plotted_by → figure |
| `{name}.m` + `{name}.mat` | generates → output |

**3.3 基于内容的关系（高级）**

读取脚本内容，识别：
- `import pandas` / `pd.read_csv()` → 读取的 CSV 文件
- `loadmat()` / `scipy.io.loadmat()` → 读取的 .mat 文件
- LaTeX `\includegraphics{file}` → 引用的图片

记录关系类型：
- `generates` - A 生成 B
- `used_by` - A 被 B 使用
- `plotted_by` - 数据被绘图脚本使用
- `appears_in` - 图表出现在论文中

### Step 4: 实验检测

扫描 `experiments/` 目录，自动识别实验：

**实验命名规则：** `exp_XXX` 或 `experiment_XXX` 或纯数字

**实验结构检测：**

| 状态 | 判定条件 |
|------|----------|
| `pending` | 目录存在，有 input/，无 output/ 或 output/ 为空 |
| `running` | 存在 `.lock` 文件或运行日志 |
| `completed` | output/ 存在文件，且最新 output mtime > 最新 input mtime |
| `failed` | 存在 error.log 或 output/ 只有错误文件 |
| `stale` | output 存在，但 output mtime < input mtime |

**实验信息提取：**

```yaml
experiments:
  - name: exp_001
    path: experiments/exp_001/
    status: completed
    created: detected
    updated: max(file_mtimes)
    input_files: [list of files in input/ or matching *.in]
    output_files: [list of files in output/ or matching *.out]
    figures: [derived from outputs]
    notes: ""
```

### Step 5: 问题检测

**5.1 孤儿文件**

无法关联到任何实验的文件：
- 不在 `experiments/` 目录下
- 不在已知的功能目录（data/, figures/, papers/ 等）
- 没有其他文件引用它

**5.2 重复文件**

同名但不同路径的文件：
```yaml
duplicates:
  - filename: "run_sim.py"
    locations: ["exp_001/scripts/", "exp_002/scripts/"]
    suggestion: "保留一个，删除另一个或重命名区分"
```

**5.3 过时检测**

```yaml
stale:
  - experiment: exp_002
    warning: "输出文件比输入文件旧，可能需要重新运行"
    newest_input: "input.par (2026-05-10)"
    newest_output: "result.csv (2026-05-05)"
    suggestion: "重新运行仿真以更新结果"
```

### Step 6: 生成报告

输出格式化的扫描报告给用户：

```markdown
# 项目扫描报告

**扫描时间:** {date}
**项目路径:** {path}

---

## 项目概览

| 指标 | 数量 |
|------|------|
| 总文件数 | N |
| 实验数 | N |
| 仿真脚本 | N |
| 数据文件 | N |
| 图表 | N |
| 论文 | N |

---

## 实验状态

| 实验 | 状态 | 输入 | 输出 | 更新时间 |
|------|------|------|------|----------|
| exp_001 | ✓ 完成 | 3 | 5 | 2026-05-10 |
| exp_002 | ⏸ 待运行 | 2 | 0 | 2026-05-08 |
| exp_003 | ⚠ 可能过期 | 3 | 5 | 2026-05-05 |
| exp_004 | ✗ 失败 | 3 | 0 | 2026-05-01 |

---

## 检测到的问题

### 孤儿文件 (N)
- `temp/untitled.m` - 无法关联到任何实验
- `~backup.csv` - 临时备份文件

### 过时结果 (N)
- **exp_003**: 输入已更新（2026-05-10）但输出未刷新（2026-05-05）

### 重复文件 (N)
- `run.py` 存在于 exp_001 和 exp_002

---

## 项目结构

```
project/
├── experiments/
│   ├── exp_001/
│   │   ├── input/  (3 files)
│   │   ├── output/ (5 files)
│   │   └── scripts/
│   ├── exp_002/
│   └── ...
├── data/
├── figures/
│   ├── fig1_force.png
│   └── fig2_velocity.png
└── papers/
    └── draft.tex
```

---

## 下一步建议

1. 运行 `/project organize` 整理现有文件到标准结构
2. 检查过时的实验是否需要重新运行
3. 确认孤儿文件的处理方式
```

### Step 7: 更新项目记忆

调用 update-memory 模块，将扫描结果持久化到 `CLAUDE_PROJECT.md`。

---

## 配置文件引用

- 文件类型规则：`rules/file-types.yaml`
- 目录模板：`templates/default-structure.yaml`

---

## 注意事项

1. **性能考虑：** 大项目文件扫描可能较慢，使用 Glob 递归时注意范围
2. **关系置信度：** 基于文件名推断的关系标记 `confidence: medium`，需要用户确认
3. **状态检测：** 优先使用时间戳判断，再结合锁文件和日志
4. **孤儿文件：** 不自动删除，只标记并询问用户