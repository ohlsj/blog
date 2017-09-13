---
title: 修改git历史提交的用户名和邮箱
date: 2017-09-12 17:00:14
tags:
 - git
---
# 问题

使用git时，一般都会先设置

```
git config --global user.name "username"
git config --global user.email "user@email.com"
```

这个属于git的全局设置，其配置文件一般是位于用户目录的`.gitconfig`。

如果有多个项目，需要不同的用户名和邮箱，可以针对项目进行设置。

首先切换当前目录至项目目录，然后设置:

```
git config user.name "username"
git config user.email "user@email.com"
```

其实就是把`--global`参数去掉。

问题来了：

*如果忘了设置项目，而且有了很多次提交了，导致历史提交中使用的用户名是全局用户名，该怎么办呢*

<!-- more -->

# 解决方案

`GitHub`官方给出了[解决方案](https://help.github.com/articles/changing-author-info/)

## 先clone一份代码

```
git clone --bare https://github.com/user/repo.git
cd repo.git
```

## 然后创建一个脚本并运行

脚本保存在项目目录中

```
#!/bin/sh

git filter-branch --env-filter '

OLD_EMAIL="your-old-email@example.com"
CORRECT_NAME="Your Correct Name"
CORRECT_EMAIL="your-correct-email@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

替换脚本中的如下值:

- `OLD_EMAIL`: 错误的email
- `CORRECT_NAME`: 正确的用户名
- `CORRECT_EMAIL`: 正确的邮箱

## 做一次`force push`

> 确保安全

```
git push --force --tags origin 'refs/heads/*'
```

## 完成

# 补充

可以取消全局用户名和邮箱

```
git config --global --unset user.name
git config --global --unset user.email
```

这样虽然每个项目都要设置一遍，但能避免上述问题。
