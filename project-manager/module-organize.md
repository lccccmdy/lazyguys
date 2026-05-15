# Organize - 整理项目结构模块

> 生成标准化目录结构，归类现有文件，处理混乱文件

## 功能清单

1. 检查当前目录结构
2. 创建缺失的标准目录
3. 归类现有文件到正确目录
4. 检测并处理问题文件
5. 重命名混乱文件（需用户确认）
6. 调用 update-memory 持久化

---

## 执行步骤

### Step 1: 检查当前结构

对比当前项目结构与标准模板（`templates/default-structure.yaml`）：

**标准目录：**
```
experiments/          # 实验目录
data/                 # 原始数据
results/              # 处理结果
figures/              # 图表
papers/               # 论文
scripts/              # 工具脚本
docs/                 # 文档
archive/              # 归档（旧实验）
```

**生成结构对比报告：**

```markdown
## 目录检查

| 目录 | 状态 | 文件数 |
|------|------|--------|
| experiments/ | ✓ 存在 | 5 |
| data/ | ✗ 缺失 | - |
| results/ | ✓ 存在 | 12 |
| figures/ | ✗ 缺失 | - |
| ...
```

### Step 2: 创建缺失目录

使用 PowerShell 创建缺失的目录：

```powershell
New-Item -ItemType Directory -Force -Path "data"
New-Item -ItemType Directory -Force -Path "figures"
New-Item -ItemType Directory -Force -Path "papers"
# ...
```

### Step 3: 文件归类

根据文件类型规则（`rules/file-types.yaml`）自动归类：

**归类规则：**

| 文件类型 | 目标目录 | 条件 |
|----------|----------|------|
| 仿真脚本 | `experiments/{name}/scripts/` 或 `scripts/` | 有对应实验目录 |
| 数据输入 | `data/` 或 `experiments/{name}/input/` | 有对应实验 |
| 数据输出 | `results/` 或 `experiments/{name}/output/` | 有对应实验 |
| 图表 | `figures/` | .png/.jpg/.svg 等 |
| 论文 | `papers/` | .tex/.bib |
| 文档 | `docs/` | .md/.txt（除 README 外） |

**特殊处理：**

1. **有归属实验的文件** → 移动到对应实验目录
2. **通用工具脚本** → 移动到 `scripts/`
3. **根目录混乱文件** → 分析用途后询问用户

### Step 4: 处理无法归类的文件

对于无法自动归类的文件，分析后列出供用户确认：

**分析策略：**

1. 读取文件头部（猜测文件类型）
2. 检查是否被其他文件引用（可能是依赖文件）
3. 检查修改时间（最近使用的可能是重要文件）
4. 检查文件大小（小文件可能是临时文件）

**询问格式：**

```markdown
## 待确认文件

以下文件无法自动归类，请确认如何处理：

1. **temp_script.py** (已修改: 2026-05-01, 大小: 2KB)
   - 可能是临时脚本
   - 操作: [删除] [移到 scripts/] [保留原位]

2. **data_old.csv** (已修改: 2026-04-15, 大小: 15MB)
   - 可能是旧数据文件
   - 操作: [移到 data/] [移到 archive/] [删除]

3. **fig_backup.png** (已修改: 2026-04-20, 大小: 200KB)
   - 看起来是图表备份
   - 操作: [移到 figures/] [移到 archive/] [删除]

请回复序号和选择，例如: "1 移到scripts, 2 移到archive, 3 删除"
```

### Step 5: 重命名建议

对于命名混乱的文件，提出重命名建议：

**问题文件模式：**
- `a.py`, `test.py`, `temp.py` → 语义不明
- `data1.csv`, `data2.csv` → 无描述性
- `run_sim_backup_final_v2.py` → 版本混乱

**处理方式：**

```markdown
## 重命名建议

| 当前名称 | 建议名称 | 理由 |
|----------|----------|------|
| a.py | run_simulation.py | 分析内容为主仿真脚本 |
| data1.csv | force_data.csv | 与 fig_force.png 关联 |
| run_sim_backup_final_v2.py | run_simulation.py | 作为唯一主脚本 |

是否执行这些重命名？ [Y/n]
```

### Step 6: 生成整理报告

```markdown
# 项目整理报告

**整理时间:** {date}
**项目路径:** {path}

---

## 目录结构变更

### 新建目录 (3)
- `data/` - 原始数据
- `figures/` - 图表
- `papers/` - 论文

### 保持不变 (5)
- `experiments/` - 已有合适结构
- `scripts/` - 工具脚本
- `results/` - 处理结果
- `docs/` - 文档
- `archive/` - 归档目录

---

## 文件移动

| 原位置 | 新位置 | 操作 |
|--------|--------|------|
| root/analysis.py | scripts/analysis.py | 移动 |
| root/results.csv | results/simulation_001.csv | 移动 + 重命名 |
| exp_001/temp.pdf | figures/exp_001_result.pdf | 移动 |

**共移动:** 12 个文件
**共重命名:** 3 个文件

---

## 待确认 (3)

已标记到 CLAUDE_PROJECT.md 的 unknowns，待下次对话确认：
- temp_script.py
- data_old.csv
- fig_backup.png

---

## 整理后结构

```
project/
├── experiments/
│   ├── exp_001/
│   │   ├── input/
│   │   ├── output/
│   │   └── scripts/
│   └── exp_002/
├── data/                 ← 新建
├── results/
├── figures/              ← 新建
├── papers/               ← 新建
├── scripts/
├── docs/
└── archive/
```

---

## 下一步

1. 运行 `/project scan` 验证项目状态
2. 检查 CLAUDE_PROJECT.md 是否正确更新
3. 开始你的工作！
```

### Step 7: 更新项目记忆

归类完成后，调用 update-memory 更新文件路径和关系。

---

## 交互式确认

整理过程中需要用户确认的环节：

1. **无法归类的文件** → 询问处理方式
2. **重命名建议** → 确认是否执行
3. **删除建议** → 确认是否删除（不自动删除任何文件）

---

## 注意事项

1. **不删除文件** → 任何删除操作都需要用户明确确认
2. **保留备份** → 移动前不删除原文件，确认后再删
3. **更新引用** → 文件移动后，需要更新 CLAUDE_PROJECT.md 中的路径
4. **实验目录优先** → 有归属实验的文件优先归到实验目录