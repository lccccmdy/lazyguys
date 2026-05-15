# Project Manager Implementation Plan

> **For agentic workers:** Use superpowers:subagent-driven-development to implement this plan task-by-task.

**Goal:** 实现 lazyguys/project-manager skill，让 AI 自动管理仿真项目

**Architecture:** 这是一个 Claude Code skill，主要通过指令和规则配置来实现，不需要传统编程。核心是优化 skill.md 中的 AI 指令逻辑，让 AI 能正确执行 scan/organize/init/query 操作。

**Tech Stack:** Claude Code Skill 格式 + YAML 配置 + Markdown 项目记忆

---

## 文件结构

```
lazyguys/project-manager/
├── skill.md                    # Skill 主入口（重写）
├── module-scan.md              # scan 命令指令
├── module-organize.md          # organize 命令指令
├── module-init.md              # init 命令指令
├── module-query.md             # query 命令指令
├── module-router.md            # 智能路由逻辑
├── update-memory.md            # CLAUDE_PROJECT.md 更新逻辑
├── CLAUDE.md                   # Skill 说明（更新）
├── README.md                   # 使用文档（更新）
└── ...
```

---

## Task 1: 重写 skill.md 主入口

**Files:**
- Modify: `lazyguys/project-manager/skill.md`

重写为完整的 Skill 说明，优化指令结构，让 AI 能直接理解任务：

- [ ] **Step 1: 添加 Skill Header**

```markdown
---
name: project-manager
description: "AI辅助项目管理：自动整理仿真项目、建立项目记忆、追踪实验状态"
author: LazyGuys
version: 1.0.0
trigger: "/project"
---

# LazyGuys - Project Manager

> 让 AI 自动管理仿真/科研项目，最大程度省心

## 使用方式

在项目目录中，直接用自然语言描述需求，例如：
- "帮我整理下这个项目"
- "看看项目现在什么状态"
- "这张图是怎么来的？"
- "哪些实验还没跑完？"

## 核心模块

加载对应的模块文件执行操作：
- `module-router.md` - 智能路由
- `module-scan.md` - 扫描复盘
- `module-organize.md` - 整理结构
- `module-init.md` - 初始化
- `module-query.md` - 查询
- `update-memory.md` - 更新项目记忆
```

- [ ] **Step 2: 添加完整的功能说明**

在 skill.md 中添加：
- 文件类型定义
- 状态定义
- 关系类型定义
- CLAUDE_PROJECT.md 格式规范
- .claude/settings.json 配置模板

- [ ] **Step 3: 添加执行流程**

```markdown
## 执行流程

1. 用户输入 → 调用 module-router.md 分析意图
2. 根据意图加载对应模块
3. 执行操作
4. 调用 update-memory.md 更新项目记忆
5. 返回结果给用户
```

- [ ] **Step 4: 提交**

```bash
git add lazyguys/project-manager/skill.md
git commit -m "feat: 重写 skill.md 主入口"
```

---

## Task 2: 创建 module-router.md（智能路由）

**Files:**
- Create: `lazyguys/project-manager/module-router.md`

- [ ] **Step 1: 路由逻辑**

```markdown
# 意图识别与路由

## 路由规则

分析用户输入，匹配关键词选择对应模块：

### 初始化
关键词：init, 初始化, 新建项目, 开始新项目
→ 加载 module-init.md

### 整理
关键词：整理, 重构, 规范化, reorganize
→ 加载 module-scan.md → module-organize.md

### 扫描
关键词：scan, 复盘, 审视, 看看状态, 现在怎样
→ 加载 module-scan.md

### 查询
关键词：query, 查询, 找, 哪些, 什么, 哪里, 怎么来的, 来源
→ 加载 module-query.md

### 归档
关键词：archive, 归档, 结束, 完成
→ 标记项目为归档状态

### 默认
无法识别意图 → 执行 scan + 生成状态报告

## 路由决策

根据关键词权重和上下文综合判断，返回最合适的模块列表。

## 输出

返回要加载的模块列表，格式：
```
MODULES: [module-scan, module-organize, update-memory]
```
```

- [ ] **Step 2: 提交**

```bash
git add lazyguys/project-manager/module-router.md
git commit -m "feat: 添加智能路由模块"
```

---

## Task 3: 创建 module-scan.md（扫描复盘）

**Files:**
- Create: `lazyguys/project-manager/module-scan.md`

- [ ] **Step 1: scan 指令**

```markdown
# Scan - 项目复盘

## 功能

1. 扫描项目所有文件
2. 识别文件类型和用途
3. 建立文件关系图谱
4. 检测问题（孤儿文件/过期结果）
5. 更新实验状态

## 执行步骤

### 1. 文件扫描

使用 Glob/Read 工具扫描项目：
- 列出所有文件
- 记录文件路径、大小、修改时间
- 按扩展名分类

### 2. 类型识别

按 file-types.yaml 规则识别：
- 仿真脚本 (.py, .m, .inp, .c, ...)
- 数据文件 (.csv, .xlsx, .txt, .mat, ...)
- 图表 (.png, .jpg, .svg, .eps, ...)
- 论文 (.tex, .bib, ...)
- 文档 (.md, .txt, ...)

### 3. 关系推断

基于：
- 文件名模式 (run_*.py → output/)
- 内容引用 (脚本中的 import/load, LaTeX 的 \includegraphics)
- 目录结构（同一个 experiments/ 下的文件）

### 4. 状态检测

对每个实验目录：
- 有 input/ 无 output/ → pending
- 有 .lock 文件或日志运行中 → running
- output/ 存在且输出比输入新 → completed
- 存在 error 日志 → failed
- 输出比输入旧 → stale（警告）

### 5. 问题检测

- 孤儿文件：无法关联到任何实验
- 重复文件：同名不同路径
- 过时结果：需要重新运行

### 6. 更新项目记忆

调用 update-memory.md 更新 CLAUDE_PROJECT.md

## 输出格式

```
# 扫描报告

## 项目概览
- 文件总数: N
- 实验数: N
- 待处理问题: N

## 实验状态
| 实验 | 状态 | 输入 | 输出 |
|------|------|------|------|

## 检测到的问题
1. 问题描述
   - 文件：xxx
   - 建议：xxx

## 项目结构
[显示目录树]
```

## 配置文件

- `../templates/default-structure.yaml` - 目录模板
- `../rules/file-types.yaml` - 文件类型规则
```

- [ ] **Step 2: 提交**

```bash
git add lazyguys/project-manager/module-scan.md
git commit -m "feat: 添加 scan 复盘模块"
```

---

## Task 4: 创建 module-organize.md（整理结构）

**Files:**
- Create: `lazyguys/project-manager/module-organize.md`

- [ ] **Step 1: organize 指令**

```markdown
# Organize - 整理项目结构

## 功能

1. 生成标准化目录结构
2. 归类现有文件到正确目录
3. 标记无法归类的文件
4. 重命名混乱文件（需确认）

## 执行步骤

### 1. 检查目录结构

对比当前结构与模板：
- 缺少的目录 → 列出待创建
- 多余的目录 → 询问是否归档
- 混乱的文件 → 标记待移动

### 2. 生成目录结构

使用 Write/PowerShell 创建目录：
```
experiments/
data/
results/
figures/
papers/
scripts/
docs/
archive/
```

### 3. 文件归类

根据 file-types.yaml 规则：
- 仿真脚本 → experiments/{name}/scripts/ 或 scripts/
- 数据文件 → data/ 或 experiments/{name}/input/
- 图表 → figures/
- 论文 → papers/

### 4. 不能确定的文件

对于无法归类的文件：
1. 分析文件名和内容
2. 询问用户："这个文件应该归类到哪里？"
3. 用户确认后移动

### 5. 重命名建议

对于命名混乱的文件（如 a.py, temp1.dat）：
1. 分析内容推测用途
2. 显示建议："a.py → run_simulation.py？"
3. 用户确认后重命名

### 6. 更新项目记忆

调用 update-memory.md

## 输出格式

```
# 整理报告

## 目录结构
- 新建: 3 个目录
- 保持: 5 个目录
- 归档: 1 个目录

## 文件移动
- xxx.csv → data/
- xxx.png → figures/

## 待确认
- [?] xxx.py - 无法确定用途，确认后移动

## 状态
整理完成，共移动 N 个文件
```
```

- [ ] **Step 2: 提交**

```bash
git add lazyguys/project-manager/module-organize.md
git commit -m "feat: 添加 organize 整理模块"
```

---

## Task 5: 创建 module-init.md（初始化）

**Files:**
- Create: `lazyguys/project-manager/module-init.md`

- [ ] **Step 1: init 指令**

```markdown
# Init - 初始化新项目

## 功能

1. 创建标准目录结构
2. 生成 CLAUDE_PROJECT.md
3. 创建 .claude/settings.json

## 执行步骤

### 1. 获取项目信息

询问用户：
- 项目名称（可选，默认用目录名）
- 项目描述（可选）
- 是否有现有文件需要整理（y/n）

### 2. 创建目录结构

按 default-structure.yaml 创建：
```
experiments/exp_000/     # 预留第一个实验目录
data/
results/
figures/
papers/
scripts/
docs/
archive/
```

### 3. 生成 CLAUDE_PROJECT.md

```markdown
# 项目概览
- 项目名称: {name}
- 创建时间: {date}
- 最后更新: {date}
- 状态: 在研

# 实验列表
experiments: []

# 文件索引
files: []

# 文件关系图
relationships: []

# 不确定项
unknowns: []
```

### 4. 创建 .claude/settings.json

```json
{
  "hooks": {
    "onAgentStart": [
      "Read CLAUDE_PROJECT.md if exists to understand project context"
    ],
    "onAgentEnd": [
      "Update CLAUDE_PROJECT.md with any new findings or changes"
    ]
  },
  "settings": {
    "project记忆": "已启用，每次工作会自动读取和更新 CLAUDE_PROJECT.md"
  }
}
```

### 5. 如果有现有文件

调用 scan 模块复盘现有项目

### 6. 生成 README.md（可选）

在项目根目录生成简单的 README.md

## 输出格式

```
# 初始化完成 ✓

项目名称: {name}
创建时间: {date}

目录结构:
├── experiments/
├── data/
├── results/
├── figures/
├── papers/
├── scripts/
├── docs/
└── archive/

项目记忆: CLAUDE_PROJECT.md ✓
全局策略: .claude/settings.json ✓

下一步：
- 运行 "/project scan" 复盘现有文件
- 或直接开始你的工作，agent 会自动维护项目记忆
```
```

- [ ] **Step 2: 提交**

```bash
git add lazyguys/project-manager/module-init.md
git commit -m "feat: 添加 init 初始化模块"
```

---

## Task 6: 创建 module-query.md（查询）

**Files:**
- Create: `lazyguys/project-manager/module-query.md`

- [ ] **Step 1: query 指令**

```markdown
# Query - 查询项目内容

## 功能

1. 自然语言查询文件和实验
2. 查文件来源/用途
3. 查实验状态
4. 查关系链路

## 常见查询模式

### 查询实验状态
用户: "哪些实验还没跑完？"
→ 读取 CLAUDE_PROJECT.md，列出 pending/running 的实验

### 查询文件来源
用户: "这张图是怎么来的？"
→ 在 relationships 中查找图的来源
→ 追溯: figure ← script ← data

### 查询文件用途
用户: "run_sim.py 是做什么的？"
→ 在 files 中查找，返回文件信息和关联

### 查询关联
用户: "draft.tex 引用了哪些图？"
→ 在 relationships 中查找所有指向 papers/draft.tex 的记录

### 通用搜索
用户: "找所有 .csv 文件"
→ 搜索 files 中匹配的文件

## 执行步骤

### 1. 解析查询意图

识别查询类型：
- 状态查询 → 查 experiments
- 来源查询 → 追溯 relationships
- 用途查询 → 查 files
- 关联查询 → 查 relationships
- 模式匹配 → 按条件过滤

### 2. 读取项目记忆

读取 CLAUDE_PROJECT.md 获取：
- experiments 列表
- files 索引
- relationships 图谱

### 3. 执行查询

根据查询类型执行相应的过滤/追溯

### 4. 返回结果

格式化的查询结果，带清晰的可读性

## 输出示例

### 示例 1: 实验状态查询
```
# 实验状态

| 实验 | 状态 | 最后更新 | 备注 |
|------|------|----------|------|
| exp_001 | ✓ 已完成 | 2026-05-10 | 参数A=1.5 |
| exp_002 | ⏸ 待运行 | 2026-05-08 | 等待 exp_001 结果 |
| exp_003 | ⚠ 可能过期 | 2026-05-05 | 输入已更新 |
```

### 示例 2: 文件来源查询
```
# fig_force_curve.png 来源追溯

fig_force_curve.png
  └─ 绘制者: scripts/plot_force.m
      └─ 数据源: data/force_001.csv
          └─ 来自实验: exp_001
              └─ 仿真脚本: simulations/first_run.py
```
```

- [ ] **Step 2: 提交**

```bash
git add lazyguys/project-manager/module-query.md
git commit -m "feat: 添加 query 查询模块"
```

---

## Task 7: 创建 update-memory.md（更新项目记忆）

**Files:**
- Create: `lazyguys/project-manager/update-memory.md`

- [ ] **Step 1: 更新逻辑指令**

```markdown
# Update Memory - 更新项目记忆

## 功能

根据操作结果更新 CLAUDE_PROJECT.md，保持项目记忆最新。

## 更新场景

### 1. scan 后更新

新增或更新：
- files 索引（新增文件）
- relationships（发现的关系）
- unknowns（发现的问题）
- 修正 experiments 状态

### 2. organize 后更新

新增或更新：
- 所有文件路径（移动后）
- relationships（路径变更）
- experiments 状态（如有新建）

### 3. 新建文件后更新

新增到 files：
```markdown
- path: relative/path/to/file
  type: {type}
  purpose: {推测或用户提供}
  last_modified: {date}
  created_by: 这次操作
```

### 4. 发现关系后更新

新增到 relationships：
```markdown
- source: path/to/source
  type: {generates|plotted_by|appears_in|used_by}
  target: path/to/target
  detected_by: scan
  confidence: high/medium/low
```

### 5. 确认未知项后更新

从 unknowns 移除，添加到 experiments 或 files

## 执行原则

1. 保留已有内容，只更新变化部分
2. 添加新条目时追加到相应列表
3. 保持格式一致性
4. 记录更新时间
5. 简洁明了，避免冗余

## 格式模板

```markdown
更新时间: {date} {time}
更新操作: {操作描述}
更新内容:
- {具体更新内容1}
- {具体更新内容2}
```

---

## 注意事项

- 不要完全重写 CLAUDE_PROJECT.md，只做增量更新
- 保持 Markdown 格式整洁
- 对于模糊的判断，标记 `confidence: low` 并在 unknowns 中列出
```

- [ ] **Step 2: 提交**

```bash
git add lazyguys/project-manager/update-memory.md
git commit -m "feat: 添加更新记忆模块"
```

---

## Task 8: 更新文档

**Files:**
- Modify: `lazyguys/project-manager/README.md`
- Modify: `lazyguys/project-manager/CLAUDE.md`

- [ ] **Step 1: 更新 README.md**

添加模块列表和使用说明

- [ ] **Step 2: 更新 CLAUDE.md**

更新当前状态为已完成

- [ ] **Step 3: 提交**

```bash
git add lazyguys/project-manager/README.md lazyguys/project-manager/CLAUDE.md
git commit -m "docs: 更新项目文档"
```

---

## 验证清单

实现完成后，确认：

- [ ] skill.md 语法正确，可被 Claude Code 加载
- [ ] 所有模块文件存在且指令清晰
- [ ] 模块间的引用关系正确
- [ ] README.md 说明了使用方法
- [ ] 在测试项目上运行 /project init 成功
- [ ] 在测试项目上运行 /project scan 成功
- [ ] 在测试项目上运行 /project query 成功
- [ ] CLAUDE_PROJECT.md 格式正确
- [ ] .claude/settings.json 格式正确（如果配置了）

---

## 依赖关系

```
skill.md (主入口)
    ↓
module-router.md → 加载其他模块
    ↓
┌─── module-scan.md ─────────────────────────────────┐
├─── module-organize.md ───────────────────────────────┤
├─── module-init.md ──────────────────────────────────┤
└─── module-query.md ─────────────────────────────────┘
    ↓
update-memory.md (更新项目记忆)
```

---

Plan complete and saved to `lazyguys/project-manager/PLAN.md`.

**Two execution options:**

**1. Subagent-Driven (recommended)** - 我为每个 Task 启动独立的 subagent，并行或串行执行

**2. Inline Execution** - 在此会话中逐步执行每个 Task

你想用哪种方式？