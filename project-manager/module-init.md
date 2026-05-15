# Init - 初始化新项目模块

> 创建标准目录结构，生成 CLAUDE_PROJECT.md 和 .claude/settings.json

## 功能清单

1. 获取项目信息（名称、描述）
2. 创建标准目录结构
3. 生成 CLAUDE_PROJECT.md
4. 创建 .claude/settings.json
5. 生成项目 README.md（可选）
6. 扫描现有文件（如果有）
7. 调用 update-memory 初始化

---

## 执行步骤

### Step 1: 获取项目信息

**如果用户未提供信息，询问：**

```markdown
## 初始化新项目

请提供以下信息（可选，直接回车使用默认值）：

1. **项目名称** (默认: 使用当前目录名)
   > my-dynamics-project

2. **项目描述** (可选)
   > 动力学仿真对比研究

3. **是否有现有文件需要整理？** (y/N)
   > 如果有，初始化后会运行 scan 复盘现有文件
```

**默认值处理：**
- 项目名称：取当前目录名
- 项目描述：空
- 扫描现有文件：N

### Step 2: 创建目录结构

根据 `templates/default-structure.yaml` 创建目录：

**基础目录：**
```powershell
New-Item -ItemType Directory -Force -Path "experiments/exp_000"
New-Item -ItemType Directory -Force -Path "experiments/exp_000/input"
New-Item -ItemType Directory -Force -Path "experiments/exp_000/output"
New-Item -ItemType Directory -Force -Path "experiments/exp_000/scripts"
New-Item -ItemType Directory -Force -Path "data"
New-Item -ItemType Directory -Force -Path "results"
New-Item -ItemType Directory -Force -Path "figures"
New-Item -ItemType Directory -Force -Path "papers"
New-Item -ItemType Directory -Force -Path "scripts"
New-Item -ItemType Directory -Force -Path "docs"
New-Item -ItemType Directory -Force -Path "archive"
```

**说明：**
- `exp_000` 作为第一个实验的模板/占位目录
- 用户可以重命名或删除它

### Step 3: 生成 CLAUDE_PROJECT.md

创建在项目根目录：

```markdown
# 项目概览
- 项目名称: {project_name}
- 创建时间: {date}
- 最后更新: {date}
- 状态: 在研
- 描述: {description}

# 实验列表
experiments: []

# 文件索引
files: []

# 文件关系图
relationships: []

# 不确定项
unknowns: []

# 备注
notes: |
  - 项目初始化于 {date}
  - 使用 LazyGuys project-manager 管理
```

### Step 4: 创建 .claude/settings.json

创建 `.claude/settings.json`：

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
    "projectManager": "enabled",
    "自动追踪": true
  }
}
```

**并创建 .claude 目录（如果不存在）：**
```powershell
New-Item -ItemType Directory -Force -Path ".claude"
```

### Step 5: 生成 README.md（可选）

如果用户同意，在项目根目录创建一个简单的 README.md：

```markdown
# {project_name}

{description | 项目描述（可选）}

## 项目结构

```
{project_name}/
├── experiments/      # 实验目录 (exp_001, exp_002, ...)
├── data/             # 原始数据
├── results/          # 处理结果
├── figures/          # 图表
├── papers/           # 论文
├── scripts/          # 工具脚本
├── docs/             # 文档
└── archive/          # 归档
```

## 使用工具

本项目使用 [LazyGuys project-manager](https://github.com/lazyguys/project-manager) 管理。

### 命令

- `/project scan` - 复盘项目状态
- `/project organize` - 整理项目结构
- `/project query "..."` - 查询项目内容

### 项目记忆

项目状态和文件关系记录在 `CLAUDE_PROJECT.md` 中。

## 更新日志

- {date}: 项目初始化
```
```

### Step 6: 扫描现有文件（如果有）

如果用户选择扫描现有文件：

```markdown
正在扫描现有文件...

[执行 module-scan.md 中的扫描逻辑]
```

扫描结果会更新到 CLAUDE_PROJECT.md。

### Step 7: 生成初始化报告

```markdown
# 项目初始化完成 ✓

**项目名称:** {project_name}
**创建时间:** {date}
**项目路径:** {path}

---

## 目录结构

```
{project_name}/
├── .claude/              ← Claude Code 全局策略
├── experiments/
│   └── exp_000/          ← 示例实验目录（可重命名/删除）
│       ├── input/
│       ├── output/
│       └── scripts/
├── data/                 ← 原始数据
├── results/              ← 处理结果
├── figures/              ← 图表
├── papers/               ← 论文
├── scripts/              ← 工具脚本
├── docs/                 ← 文档
├── archive/              ← 归档
├── CLAUDE_PROJECT.md     ← 项目记忆（自动维护）
└── README.md             ← 项目说明（可选）
```

**新增文件:**
- `.claude/settings.json` - Claude Code 策略配置
- `CLAUDE_PROJECT.md` - 项目记忆
- `README.md` - 项目说明（可选）

---

## 接下来

1. **重命名实验目录** - 将 `exp_000` 改名为你的第一个实验，例如 `exp_001`
2. **运行 scan** - 运行 `/project scan` 复盘并验证项目状态
3. **开始工作** - 直接开始你的仿真工作，agent 会自动维护项目记忆！

---

## 项目记忆已启用

今后 agent 每次工作时会：
- 开始前读取 `CLAUDE_PROJECT.md` 了解项目上下文
- 工作后自动更新 `CLAUDE_PROJECT.md` 记录变更
``````

---

## 注意事项

1. **不覆盖已有文件** - 如果 CLAUDE_PROJECT.md 已存在，会询问是否覆盖
2. **保留现有结构** - 如果用户已有合理的目录结构，不会强制重构
3. **增量初始化** - 如果某些目录已存在，跳过创建
4. **.claude 目录** - 创建在项目根目录，不是主目录