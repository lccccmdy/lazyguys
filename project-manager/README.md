# LazyGuys - Project Manager

> 让 AI 自动管理仿真/科研项目，最大程度省心

---

## 功能特性

### 1. 智能路由
只用自然语言描述需求，AI 自动选择最合适的操作：
- "帮我整理下这个项目" → scan + organize
- "哪些实验还没跑完？" → query 状态
- "这张图是怎么来的？" → query 来源

### 2. 自动项目管理
- 初始化标准目录结构
- 归类现有文件到正确位置
- 重命名混乱文件（需确认）

### 3. 主动状态追踪
自动检测实验运行状态：
| 状态 | 含义 |
|------|------|
| `pending` | 待运行 |
| `running` | 运行中 |
| `completed` | 完成 |
| `failed` | 失败 |
| `stale` | 可能过期 |

检测过时结果（输入更新但输出未刷新）

### 4. 文件关系图谱
自动建立并维护：
- 仿真脚本 → 输出数据
- 数据 → 图表
- 图表 → 论文引用

### 5. 项目记忆
自动生成并维护 `CLAUDE_PROJECT.md`：
- 实验列表和状态
- 文件索引
- 关系图谱
- 待处理问题

### 6. Agent 全局策略
自动配置 `.claude/settings.json`，让 agent：
- 工作前读取项目记忆
- 工作后更新项目记忆

---

## 快速开始

### 1. 初始化新项目

```
/project init
```

或直接说："初始化一个新项目"

### 2. 查看项目状态

```
/project scan
```

或直接说："看看项目现在什么状态"

### 3. 整理项目

```
/project reorganize
```

或直接说："帮我整理下这个项目"

### 4. 查询项目

```
/project "哪些实验还没跑完？"
/project "这张图是怎么来的？"
/project "找出所有 .csv 文件"
```

---

## 项目结构

```
project-manager/
├── skill.md                    # Skill 主入口
├── module-router.md            # 智能路由
├── module-scan.md              # 扫描复盘
├── module-organize.md          # 整理结构
├── module-init.md              # 初始化
├── module-query.md             # 查询
├── update-memory.md            # 更新记忆
├── templates/
│   └── default-structure.yaml  # 目录模板
└── rules/
    └── file-types.yaml         # 文件类型规则
```

---

## 适用场景

- 动力学仿真项目（ADAMS/ANSYS/Matlab...）
- 后处理数据分析
- 论文绘图工作流
- 多实验/多论文并行管理
- 任何需要整理大量文件的工程科研项目

---

## 自定义

### 自定义目录结构

编辑 `templates/default-structure.yaml`

### 自定义文件类型规则

编辑 `rules/file-types.yaml`

---

## 上传到 GitHub

直接上传 `lazyguys` 文件夹到 GitHub，clone 后可复用：

```bash
# 上传
cd lazyguys
git init
git add .
git commit -m "Initial commit"
git remote add origin <your-repo-url>
git push -u origin main
```

---

## 状态

- [x] 设计文档 SPEC.md
- [x] skill.md 主入口
- [x] 智能路由 module-router.md
- [x] 扫描复盘 module-scan.md
- [x] 整理结构 module-organize.md
- [x] 初始化 module-init.md
- [x] 查询 module-query.md
- [x] 更新记忆 update-memory.md
- [x] 模板配置
- [x] 规则配置
- [ ] 测试验证（待完成）