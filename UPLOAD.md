# 上传到 GitHub

## 快速上传

```bash
# 进入 lazyguys 目录
cd lazyguys

# 初始化 (如果还没 git)
git init

# 添加所有文件
git add .

# 提交
git commit -m "Initial commit: LazyGuys project-manager"

# 创建 GitHub 仓库后关联
git remote add origin https://github.com/YOUR_USERNAME/lazyguys.git
git branch -M main
git push -u origin main
```

## GitHub 创建仓库步骤

1. 登录 [GitHub](https://github.com)
2. 点击右上角 **+** → **New repository**
3. 填写：
   - **Repository name:** `lazyguys`
   - **Description:** `让 AI 做繁琐的事，你只专注于真正重要的事情`
   - **Public** 或 **Private** (Public 免费)
4. 点击 **Create repository**
5. 按照页面上的命令推送代码

## 上传后

在 GitHub 仓库页面，可以添加 Topics：
- `claude-code`
- `productivity`
- `project-management`
- `simulation`

---

## 目录结构确认

```
lazyguys/
├── .gitignore
├── README.md
├── CLAUDE.md
└── project-manager/
    ├── SPEC.md
    ├── PLAN.md
    ├── skill.md
    ├── module-router.md
    ├── module-scan.md
    ├── module-organize.md
    ├── module-init.md
    ├── module-query.md
    ├── update-memory.md
    ├── README.md
    ├── CLAUDE.md
    ├── templates/
    │   └── default-structure.yaml
    └── rules/
        └── file-types.yaml
```