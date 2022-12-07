## 拉取指定分支的代码：

```bash
# branchName 远程分支名
$ git clone -b branchName https://github.com/xx/xx.git
```

## 删除远程分支：
```bash
# 删除远程分支 
$ git push origin -d branchName
```

## 将本地分支推送到远程分支（如果远程分支不存在，会创建该分支）：
```bash
# localName  本地分支名
# originName 远程分支名
$ git push origin localName:originName
```

## 重命名远程分支：
### 方法一
直接在github仓库Settings里面修改

### 方法二
远程分支 branchNameA -> 远程分支 branchNameB

1.拉取目标远程分支 branchNameA
```bash
# 本地是空仓库
git clone -b branchNameA https://github.com/xx/xx.git
```
or
```bash
# 本地已有仓库
# 语法一
git checkout -b branchNameA origin/branchNameA
# 语法二
git fetch origin branchNameA:branchNameA
```

2.新建并切换至新的本地分支 B
```bash
git checkout -b branchNameB
```

3.删除目标远程分支 A
```bash
git push origin -d branchNameA
```

4.把本地新分支 B 推送到远程分支 B
```bash
git push origin branchNameB:branchNameB
```

## 回滚到指定版本
```bash
# 查看提交日志，选取其中要回滚的日志 id
$ git log

# 重置本地 log id
$ git reset --hard log_id

# 因为是回滚，需要加 f 参数覆盖，仅建议一人维护的仓库使用；有多人合作的情况一定要确保回滚的内容是所有人不需要的，否则会导致代码丢失无法找回
$ git push -f
```

## 清除git提交记录
确保本地仓库代码是最新完整的版本，在本地仓库使用命令创建新分支，例如 clearbh
```bash
git checkout --orphan clearbh
```

使用 --orphan 选项，可创建1个无任何提交历史的分支。但由于HEAD指向的引用中没有commit值，必须进行一次提交。
```bash
git add .
git commit -m "init repo"
```

删除原来的分支，例如 main
```bash
git branch -D main
```

把新建的分支重命名回原来的名字
```bash
git branch -m main
```

强制推送到远程仓库（会覆盖远程仓库，仅建议一人维护的仓库使用；有多人合作的情况一定要确保所有人最新的代码都在里面，否则绝对不要使用此方法）
```bash
git push -f origin main
```
