# Project Manager Skill

## 用途

AI 辅助项目管理的 Claude Code Skill，专注于工程仿真/科研工作流。

## 模块列表

| 文件 | 功能 | 状态 |
|------|------|------|
| `skill.md` | Skill 主入口，加载此文件 | ✓ |
| `module-router.md` | 意图识别，智能路由 | ✓ |
| `module-scan.md` | 扫描项目，建立关系图谱 | ✓ |
| `module-organize.md` | 整理项目结构 | ✓ |
| `module-init.md` | 初始化新项目 | ✓ |
| `module-query.md` | 查询项目内容 | ✓ |
| `update-memory.md` | 更新项目记忆 | ✓ |
| `templates/default-structure.yaml` | 目录结构模板 | ✓ |
| `rules/file-types.yaml` | 文件类型识别规则 | ✓ |

## 工作流程

```
用户输入 → module-router 分析意图
    ↓
加载对应模块执行
    ↓
update-memory 更新 CLAUDE_PROJECT.md
    ↓
返回结果给用户
```

## 使用方式

在 Claude Code 中加载此 Skill，直接用自然语言描述需求。

## 当前状态

实现完成，待测试验证。