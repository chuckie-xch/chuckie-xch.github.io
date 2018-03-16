---
title: Git的常用命令
date: 2017-12-25 15:40:03
tags: 团队协作
icon: fa-crop
---

# Git的常用命令

## 利用Github创建远程仓库

### 1. 在Github上创建一个仓库
### 2. 在本地创建项目，并创建readme.md和.gitignore
> touch README.md
> touch .gitignore

.gitignore文件内容如下：

```java
*.class

#package file
*.war
*.ear

#kdiff3 ignore
*.orig

#maven ignore
target/

#eclipse
.settings/
.project
.classpath

#idea
.idea/
/idea/
*.ipr
*.iml
*.iws

#temp file
*.log
*.cache
*.diff
*.path
*.tmp

#system
.DS_Store
Thumbs.db
```

### 3. 执行如下命令： 

```java
git init  // 初始化本地仓库
git add .  // 将工作区的变化提交到暂存区(index)
git commit -am 'first commit'  // 提交到head
git remote add "https://github.com/youproject.git"  // 添加更改到远程仓库
git push -u -f origin master // 第一次提交要加上'-f'参数强制提交
git checkout -b dev // 创建dev 分支
git push origin HEAD -u // 提交分支到远程仓库

```







