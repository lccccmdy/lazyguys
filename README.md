# LazyGuys

> 让 AI 成为你的懒人助手，最大程度省心

## 理念

> 让 AI 做繁琐的事，你只专注于真正重要的事情

## 工具集

### 1. project-manager
AI 辅助项目管理，专注于工程仿真/科研工作流。

**解决的问题：**
- 项目文件越积越多，无从整理
- 实验/论文/数据之间关系混乱
- 每次和新 agent 合作都要重新解释项目
- 不知道哪些实验结果过时了

**核心能力：**
- 自动整理项目结构
- 主动追踪实验状态
- 建立文件关系图谱
- 项目记忆自动维护

**使用方式：**
```bash
# 直接用自然语言描述需求
/project reorganize        # 整理项目
/project init              # 初始化新项目
/project "哪些实验还没跑完"  # 查询
```

**适用：** 动力学仿真、后处理数据、论文绘图、工程科研项目

---

## 设计原则

1. **智能路由** - 用户只说意图，skill 自动选择操作
2. **主动追踪** - 最大程度省心，不用手动维护状态
3. **项目记忆** - 自动生成并维护 CLAUDE_PROJECT.md
4. **自包含** - 所有配置和规则都在 lazyguys 文件夹内，可直接上传 GitHub 复用

---

## 文件结构

```
lazyguys/
├── README.md              # 本文档
├── CLAUDE.md              # 项目说明
└── project-manager/       # 项目管理 skill
    ├── README.md
    ├── CLAUDE.md
    ├── skill.md
    ├── templates/
    │   └── default-structure.yaml
    └── rules/
        └── file-types.yaml
```

上传 `lazyguys` 文件夹到 GitHub，clone 后可直接使用。