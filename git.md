### 拉取指定分支的代码：

```bash
# branchName 远程分支名
$ git clone -b branchname https://github.com/xx/xx.git
```

### 删除远程分支：
```bash
# 删除远程分支 
$ git push origin :branchName
```

### 将本地分支推送到远程分支（如果远程分支不存在，会创建该分支）：
```bash
# localName  本地分支名
# originName 远程分支名
$ git push origin localName:originName
```

### 重命名远程分支：
远程分支 A -> 远程分支 B
- 拉取目标远程分支 A
- 新建并切换至新的本地分支 B
- 删除目标远程分支 A
- 把本地新分支 B 推送到远程分支 B
