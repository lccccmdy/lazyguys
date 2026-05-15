# Update Memory - 更新项目记忆模块

> 将操作结果持久化到 CLAUDE_PROJECT.md

## 功能清单

1. 读取当前 CLAUDE_PROJECT.md
2. 增量更新内容
3. 保持格式一致性
4. 记录更新日志

---

## 操作场景

### 1. scan 后更新

在 scan 操作后更新：

**更新内容：**
- `files` - 新发现的文件
- `relationships` - 新建立的关系
- `experiments` - 新检测到的实验及其状态
- `unknowns` - 新发现的孤儿文件
- `stale` - 新检测到的过时实验
- `最后更新` - 时间戳

**更新逻辑：**
```markdown
# 文件索引 (部分更新)
files:
  # ... 已有文件保持不变 ...
  - path: {new_file}
    type: {type}
    experiment: {exp_name}
    purpose: {purpose}
    last_modified: {date}
    detected_by: scan
    confidence: high/medium/low
```

### 2. organize 后更新

在 organize 操作后更新：

**更新内容：**
- `files` - 更新所有移动文件的路径
- `relationships` - 更新所有路径相关的引用
- `最后更新` - 时间戳

**更新逻辑：**
```markdown
# 文件路径更新示例
- path: new/path/to/file  # 旧路径已移除
  # 其他字段保持不变
```

### 3. 新建文件后更新

在日常工作中更新：

**更新内容：**
- `files` - 新创建的文件
- `relationships` - 新建立的关系
- `experiments` - 新增实验或更新状态
- `最后更新` - 时间戳

**更新格式：**
```markdown
# 新增文件
- path: {relative_path}
  type: {type}
  purpose: {purpose}
  last_modified: {date}
  created_by: agent_session
  notes: ""
```

### 4. 删除文件后更新

在确认删除文件后更新：

**更新内容：**
- 从 `files` 移除已删除的文件
- 从 `relationships` 移除相关关系
- 从 `unknowns` 移除已处理的项目

### 5. 确认未知项后更新

在用户确认孤儿文件处理方式后：

**更新内容：**
- 将文件正确归类到 `files`
- 从 `unknowns` 移除
- 关联到对应实验

---

## 更新时间戳

在文件头部更新：

```markdown
- 最后更新: {date} {time}
```

---

## 更新日志

每次更新在文件末尾追加更新记录：

```markdown
---

## 更新日志

### {date} {time}
- 操作: scan
- 更新内容:
  - 新增文件: 12
  - 新增实验: 3
  - 新增关系: 8
  - 待确认项: 2
```

---

## 完整更新流程

### Step 1: 读取当前记忆

读取项目根目录的 `CLAUDE_PROJECT.md`

### Step 2: 合并变更

```python
# 伪代码
new_files = current_files + scan_result.new_files
new_rels = current_rels + scan_result.new_relationships
new_exps = merge_experiments(current_exps, scan_result.experiments)
```

### Step 3: 去重

- 文件列表：根据 `path` 去重，用新的覆盖旧的
- 关系列表：根据 `source` + `type` + `target` 去重
- 实验列表：根据 `name` 去重

### Step 4: 更新格式

确保更新后的内容符合格式规范

### Step 5: 写回文件

使用 Write 工具覆盖 CLAUDE_PROJECT.md

### Step 6: 追加日志

在更新日志部分追加本次更新

---

## 格式模板

### 完整 CLAUDE_PROJECT.md 结构

```markdown
# 项目概览
- 项目名称: {name}
- 创建时间: {date}
- 最后更新: {date} {time}
- 状态: 在研/结项/归档
- 描述: {description}

# 实验列表
experiments:
  - name: {exp_name}
    purpose: {purpose}
    status: pending/running/completed/failed/stale
    created: {date}
    updated: {date}
    inputs: [{file1}, {file2}]
    outputs: [{file1}, {file2}]
    notes: {notes}

# 文件索引
files:
  - path: {relative/path/to/file}
    type: simulation-script/data-input/data-output/figure/paper/documentation
    experiment: {exp_name} or null
    purpose: {description}
    outputs: [{files this generates}]
    used_by: [{files that use this}]
    last_modified: {date}
    created_by: scan/user/agent
    notes: {notes}

# 文件关系图
relationships:
  - source: {path}
    type: generates/plotted_by/appears_in/used_by
    target: {path}
    detected_by: scan/manual
    confidence: high/medium/low

# 不确定项
unknowns:
  - file: {path}
    reason: {reason}
    suggested_action: {action}
    user_decided: null  # null = pending, or the chosen action

# 过时实验
stale:
  - experiment: {exp_name}
    warning: {warning message}
    newest_input: {file} ({date})
    newest_output: {file} ({date})
    resolved: false  # or true after rerun

# 更新日志
## {date} {time}
- 操作: {operation}
- 更新内容:
  - {content1}
  - {content2}
```

---

## 常见问题处理

### CLAUDE_PROJECT.md 不存在

```markdown
无法找到 CLAUDE_PROJECT.md

提示: 项目可能未初始化。使用 /project init 初始化项目。
```

### 文件格式错误

```markdown
CLAUDE_PROJECT.md 格式异常，尝试自动修复...

修复完成。如果仍有错误，请检查文件格式。
```

### 大量更新

如果单次更新文件很多（>20），分批更新并显示进度：

```markdown
正在更新项目记忆... [1/3]
- 更新实验列表... ✓
- 更新文件索引... ✓
- 更新关系图... ✓
正在保存... ✓

更新完成: 新增 45 个文件，12 个关系
```

---

## 注意事项

1. **只做增量更新** - 不完全重写，只添加或更新变化的条目
2. **保持格式一致** - 所有条目必须符合格式规范
3. **记录更新时间** - 每次更新都要更新 "最后更新" 时间戳
4. **追加更新日志** - 记录每次更新的概要，便于追溯
5. **去重处理** - 相同条目只保留最新版本