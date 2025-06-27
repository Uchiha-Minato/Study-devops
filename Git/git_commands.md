# Git 常用bash命令

## 基本信息设置

--global为全局设置

```bash
git config --global user.name "abc123"

git config --global user.email "abc123@abc.com"
```

## 建立本地仓库并初始化

```bash
git init
```

这行代码需要目录中没有名为`.git`的隐藏文件夹。

命令运行完成后自动创建。

## 设置remote仓库

```bash
git remote add origin <远程分支url>
```

**origin是远程仓库的默认名称。**

## 查看可用分支

```bash
# 查看所有<远程>分支，通常以"origin/"为前缀
git branch < -r >

# 查看所有分支
git branch -a
```

## 下载项目

```bash
git clone [-b < 分支名 >] < url >  
```

## 同步项目

### 方式一：git pull

```bash 
# 查询当前远程分支
git remote -v

# 直接拉取并合并最新代码
git pull origin < 远程分支名 >
# git pull origin dev
```

### 方式二：git fetch + git merge

#### 额外新建本地分支型

```bash
git fetch origin < 远程分支名 >:< 本地新建分支名 >
# git fetch origin dev:dev111

# 查看本地新建分支与当前分支的差异
git diff < 本地新建分支名 >
# git diff dev1

# 合并新建分支到本地分支
git merge < 本地新建分支名 >

# 删除新建分支
git branch -D < 本地新建分支名 >
# -d 删除，-D 强制删除
```

#### 不新建型

```bash 
git fetch origin < 远程分支名 >
# git fetch origin dev

# 查看版本差异
git log -p < 本地分支名 >..< 远程分支名 > 
# git log -p dev..origin/dev

# 合并远程分支到当前本地分支
git merge < 远程分支名 >
```

## 提交

```bash
git add < 需要提交的文件名 >
# git add . 代表添加当前目录下所有已更改的文件到暂存区

git commit -m "提交信息"
```

## 推送

```bash
git push origin < 远程分支名 >

git push [-u] origin < 新建远程分支名 >
# -u 或 --set-upstream
# 会将本地分支与远程分支关联起来，方便以后的拉取和推送操作。
```

## 切换本地分支

### git checkout

```bash
git checkout [-b] < 分支名 >
# -b 为新建本地分支
# 如果当前分支有未提交的更改，则 git checkout 命令将覆盖这些更改。

git checkout -
# 切换到上一个分支

git checkout < commit-hash >
# 切换到某次特定提交的状态（分离头指针状态）

```

除了切换分支，`git checkout` 还有恢复文件/目录的功能。

```bash
git checkout -- < 文件名/目录 >
# 撤销对文件的所有未提交更改，将文件恢复到当前分支的 HEAD（最新提交）所记录的状态。
```

### git switch

Git *2.23* 版本新增，**只用于切换分支**操作。

```bash
git switch [-c] < 分支名 >
# -c/--create 参数为新建分支

git switch -

git switch < commit-hash >
```

## 查看提交记录

```bash
git log [选项] [分支名/提交hash]
```

常见选项:

- `-n <number>` 限制显示log多少条
- `--since="2024-01-01"` 显示自指定日期之后的提交
- `--until="2024-07-01"` 显示指定日期之前的提交