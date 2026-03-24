# 同步上游 WSTG 英文原版操作指南

本项目为 OWASP Web Security Testing Guide (WSTG) 的简体中文翻译项目。由于您是从 [OWASP/wstg](https://github.com/OWASP/wstg) fork 的，本文档说明如何同步上游英文原版的更新。

## 目录

1. [前置说明](#前置说明)
2. [首次同步配置](#首次同步配置)
3. [日常同步流程](#日常同步流程)
4. [处理文件冲突](#处理文件冲突)
5. [翻译状态说明](#翻译状态说明)

---

## 前置说明

### 仓库对应关系

| 仓库 | 地址 | 说明 |
|------|------|------|
| 上游原版 | https://github.com/OWASP/wstg | 英文原版，持续更新中 |
| 您的仓库 | https://github.com/2ai9hou/wstg-Chinese | 您的中文翻译 Fork |

### 同步策略

由于本项目已进行全量中文翻译，同步上游更新时：
- **英文原文变更** → 直接合并
- **中文翻译文件变更** → 可能产生冲突，需要手动处理
- **其他文件变更** → 直接合并

---

## 首次同步配置

如果您是首次配置同步，执行以下步骤：

### 1. 添加上游仓库

```bash
git remote add upstream https://github.com/OWASP/wstg.git
```

### 2. 验证配置

```bash
git remote -v
```

预期输出：

```
origin  https://github.com/2ai9hou/wstg-Chinese (fetch)
origin  https://github.com/2ai9hou/wstg-Chinese (push)
upstream        https://github.com/OWASP/wstg.git (fetch)
upstream        https://github.com/OWASP/wstg.git (push)
```

---

## 日常同步流程

### 步骤 1：确保本地 master 分支最新

```bash
git checkout master
git pull origin master
```

### 步骤 2：获取上游更新

```bash
git fetch upstream
```

### 步骤 3：合并上游更改

**方式 A：合并（推荐）**

```bash
git merge upstream/master
```

**方式 B：变基（如需保持线性历史）**

```bash
git rebase upstream/master
```

### 步骤 4：推送到您的远程

```bash
git push origin master
```

---

## 处理文件冲突

当合并上游更新时，如果 `document/` 目录中的中文翻译文件与上游英文原文有冲突，Git 会报告冲突。

### 冲突识别

合并时出现冲突的文件会显示类似：

```
Auto-merging document/4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md
CONFLICT (content): Merge conflict in document/4-Web_Application_Security_Testing/01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md
```

### 步骤 1：查看冲突文件列表

```bash
git status
```

或

```bash
git diff --name-only --diff-filter=U
```

### 步骤 2：检查冲突内容

打开冲突文件，冲突部分以 `<<<<<<< HEAD`、`=======`、`>>>>>>>` 标记：

```
<<<<<<< HEAD
# 搜索引擎侦察以发现信息泄露（中文翻译）
=======
# Conduct Search Engine Discovery Reconnaissance for Information Leakage（英文原文）
>>>>>>> upstream/master
```

### 步骤 3：解决冲突

**原则**：

| 情况 | 处理方式 |
|------|----------|
| 上游仅更新英文内容 | 保留中文翻译（HEAD 部分） |
| 上游新增文件 | 翻译新增内容，或标记待处理 |
| 上游删除文件 | 删除对应的中文翻译文件 |
| 两者都改了相同部分 | 保留中文翻译，标注需要审阅 |

**操作示例**：

```bash
# 编辑冲突文件，保留中文翻译部分
vim document/xxx.md

# 标记冲突已解决
git add document/xxx.md
```

### 步骤 4：提交合并

```bash
git commit -m "merge: resolve conflicts with upstream/master"
```

### 步骤 5：推送

```bash
git push origin master
```

---

## 翻译状态说明

### 文件状态分类

| 状态 | 说明 | 处理方式 |
|------|------|----------|
| `translated` | 已翻译为中文 | 保留中文，忽略上游更新 |
| `upstream-new` | 上游新增文件 | 翻译新文件或标记待处理 |
| `upstream-deleted` | 上游已删除 | 删除对应中文文件 |
| `both-modified` | 双方都有修改 | 手动合并，优先保留中文 |

### 快速检查脚本

创建文件 `check-translation-status.sh`：

```bash
#!/bin/bash

echo "=== 检查翻译状态 ==="
echo ""

echo "1. 上游新增的英文文件（可能需要翻译）："
git log --oneline origin/master..upstream/master --name-only | grep "\.md$" | sort -u

echo ""
echo "2. 可能存在冲突的文件（已翻译的文件在上游有更新）："
for file in $(git diff --name-only origin/master upstream/master -- document/); do
    if grep -q "^# " "$file" 2>/dev/null; then
        first_line=$(head -1 "$file")
        if [[ "$first_line" != "# "* ]]; then
            echo "  $file (可能已翻译)"
        fi
    fi
done
```

执行：

```bash
chmod +x check-translation-status.sh
./check-translation-status.sh
```

---

## 批量处理脚本

创建文件 `sync-upstream.sh`：

```bash
#!/bin/bash

set -e

echo "=== 同步上游 WSTG 更新 ==="
echo ""

# 1. 确保在 master 分支
git checkout master

# 2. 拉取本地远程更新
echo "[1/6] 拉取本地远程更新..."
git pull origin master

# 3. 获取上游更新
echo "[2/6] 获取上游更新..."
git fetch upstream

# 4. 合并上游
echo "[3/6] 合并上游更改..."
git merge upstream/master --no-edit

# 5. 检查冲突
CONFLICTS=$(git diff --name-only --diff-filter=U | wc -l)
if [ "$CONFLICTS" -gt 0 ]; then
    echo "[4/6] 发现 $CONFLICTS 个冲突文件！"
    echo "冲突文件列表："
    git diff --name-only --diff-filter=U
    echo ""
    echo "请手动解决冲突后，运行以下命令完成合并："
    echo "  git add ."
    echo "  git commit"
    echo "  git push"
    exit 1
fi

# 6. 推送
echo "[5/6] 推送更新..."
git push origin master

echo "[6/6] 完成！"
echo ""
echo "=== 同步完成 ==="
```

执行：

```bash
chmod +x sync-upstream.sh
./sync-upstream.sh
```

---

## 注意事项

1. **不要直接编辑上游仓库** - 您没有上游仓库的写权限
2. **定期同步** - 建议每周或每两周同步一次，避免累积大量冲突
3. **翻译优先级** - 如果冲突过多，优先处理与安全测试相关的核心文件
4. **保留英文参考** - 有疑问时，可参考上游英文原版内容

---

## 常见问题

### Q: 冲突太多了怎么办？

A: 如果冲突文件超过 20 个，建议分批处理：

```bash
# 先只获取上游更新，暂不合并
git fetch upstream

# 查看有哪些文件会冲突
git diff --name-only origin/master upstream/master -- document/
```

然后逐个或分批处理。

### Q: 可以只同步特定目录吗？

A: 可以，但不建议：

```bash
git merge upstream/master --no-commit --no-ff
git checkout --ours document/  # 保留中文版本
git add document/
git commit
```

### Q: 同步后想回退怎么办？

A: 使用 reflog 找回之前的版本：

```bash
git reflog
git reset --hard HEAD@{N}  # N 是回退前的版本号
```

---

## 参考链接

- [Git 官方文档 - 远程仓库](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93)
- [GitHub 官方文档 - 同步 Fork](https://docs.github.com/cn/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork)
- [OWASP WSTG 仓库](https://github.com/OWASP/wstg)
