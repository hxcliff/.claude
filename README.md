# Claude Code 配置仓库

本仓库用于管理 Claude Code 的自定义配置，包括 skills、commands 等。

## 快速开始

### 克隆仓库后初始化

```bash
# 初始化所有 submodule
git submodule update --init --recursive

# 配置 sparse-checkout（只检出需要的文件夹）
cd .skills-upstream
git sparse-checkout set skills/doc-coauthoring skills/docx skills/pdf skills/pptx skills/xlsx skills/web-artifacts-builder
cd ..
```

### 更新上游 skills

```bash
# 方式一：更新到上游最新提交
git submodule update --remote .skills-upstream

# 方式二：进入 submodule 手动操作
cd .skills-upstream && git pull origin main && cd ..

# 如果上游新增了你需要的文件夹，追加到 sparse-checkout
cd .skills-upstream
git sparse-checkout add skills/新文件夹名
cd ..
```

---

## 如何添加其他仓库的特定文件夹

如果你想把其他 Git 仓库的某个文件夹关联到本地，可以按以下步骤操作。

### 场景示例

假设你想把 `https://github.com/example/repo.git` 仓库中的 `src/components` 文件夹关联到本地的 `my-components/` 目录。

### 步骤 1：添加 submodule

```bash
# 添加远程仓库作为 submodule（放到隐藏目录避免干扰）
git submodule add https://github.com/example/repo.git .repo-upstream
```

### 步骤 2：配置 sparse-checkout

```bash
cd .repo-upstream

# 初始化 sparse-checkout
git sparse-checkout init --cone

# 设置只检出需要的文件夹（可以指定多个）
git sparse-checkout set src/components src/utils

cd ..
```

### 步骤 3：创建符号链接

```bash
# 创建目标目录（如果不存在）
mkdir -p my-components

# 创建符号链接（注意相对路径）
ln -s ../.repo-upstream/src/components my-components/components
ln -s ../.repo-upstream/src/utils my-components/utils
```

### 步骤 4：提交更改

```bash
git add .gitmodules .repo-upstream my-components/
git commit -m "feat: 添加 example/repo submodule"
```

---

## 常用命令速查

| 操作 | 命令 |
|------|------|
| 初始化所有 submodule | `git submodule update --init --recursive` |
| 更新指定 submodule | `git submodule update --remote <submodule路径>` |
| 更新所有 submodule | `git submodule update --remote` |
| 查看 submodule 状态 | `git submodule status` |
| 添加 sparse-checkout 路径 | `cd <submodule> && git sparse-checkout add <路径>` |
| 查看当前 sparse-checkout 配置 | `cd <submodule> && git sparse-checkout list` |
| 删除 submodule | 见下方说明 |

### 删除 submodule

```bash
# 1. 从 .gitmodules 和 .git/config 移除
git submodule deinit -f <submodule路径>

# 2. 从工作目录和索引移除
git rm -f <submodule路径>

# 3. 清理 .git/modules 缓存
rm -rf .git/modules/<submodule路径>

# 4. 删除相关符号链接
rm <符号链接路径>

# 5. 提交
git commit -m "chore: 移除 submodule"
```

---

## 目录结构

```
.claude/
├── .skills-upstream/          # anthropics/skills submodule (sparse-checkout)
├── commands/                  # 自定义命令
├── skills/                    # skills（含符号链接）
│   ├── doc-coauthoring -> ../.skills-upstream/skills/doc-coauthoring
│   ├── docx -> ../.skills-upstream/skills/docx
│   ├── pdf -> ../.skills-upstream/skills/pdf
│   ├── pptx -> ../.skills-upstream/skills/pptx
│   ├── web-artifacts-builder -> ../.skills-upstream/skills/web-artifacts-builder
│   └── xlsx -> ../.skills-upstream/skills/xlsx
├── settings.json              # Claude Code 设置
└── README.md                  # 本文件
```

---

## 注意事项

1. **符号链接的相对路径**：创建符号链接时使用相对路径，确保仓库在不同机器上都能正常工作。

2. **sparse-checkout 模式**：使用 `--cone` 模式更高效，它只支持目录级别的过滤。如果需要文件级别过滤，去掉 `--cone` 参数。

3. **submodule 版本锁定**：默认情况下 submodule 会锁定到特定 commit。使用 `--remote` 参数更新到上游最新。

4. **Windows 兼容性**：Windows 上符号链接可能需要管理员权限或开启开发者模式。
