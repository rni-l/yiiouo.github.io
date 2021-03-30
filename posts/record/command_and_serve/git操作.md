---

title: git操作
date: 2017-08-24 23:56:01
tags: ["git"]
categories: ["记录"]
draft: true
---

# git日常操作

* 在 gh-pages 显示： git push origin master:gh-pages (http://[name].github.io/[xxx])

## 查看版本·

* 查看： git head
* 查看某个版本： git head~2/3/*
* 查看指定版本： git show sha-1
* 查看引用日志： git reflog

## 删除

* 删除：git rm
* 删除文件夹： git rm -r

## 打标签

* 查看标签： git tag
* 查看标签详细信息：git tag -ln
* 新建标签： git tag -a v1.0 -m '...'
* 推送标签： git push origin v1.0
* 删除标签： git tag -d v1.0
* 删除远程标签：git push origin --delete tag <branchName> 
* 拉取标签： git checkout  -b <branch_name> <tag_name>
* 获取上游标签：git fetch upstream tag <tag_name>
* 将相关的分支也推送到仓库上：git push --follow-tags origin master

## 合并/分支

* 查看分支： git branch
* 生成并切换分支： git checkout -b dev
* 删除分支： git branch -d dev
* 合并分支： git merge dev (dev 分支 commit 后，切换别的分支，再合并)
* 删除远程分支：git push origin --delete <branchName> 
* 分支重命名：git branch -m old new

## 日志

* 查看日志：git log
* 查看日志（美化版）：git log --graph --pretty=oneline
* 查看日志（终极美化版，head 信息减少）：git log --graph --pretty=oneline --abbrev-commit
* 查看差异：git diff
* 查看缓存区的差异： git diff --cached
* 查看文件修改情况： git log —stat
* 查看某次记录: git show ${VERSION}
* git commit --amend，当你 commit 后，再修改，使用此命令，不会多一个 commit 记录
* 记录一行显示：git log —online

## 回滚

* 回滚：git revert HEAD
* 回滚报错取消：git revert --abort

`git revert head` 会返回上一个版本， 如果要跳到某个版本的话，会造成冲突，有个方法，一级级进行回滚，就不会造成冲突

* 清空缓存区：git reset

* 清除缓存区： git reset --hard
  `git add .` 后，使用 `git reset --hard` 会清掉这次添加的记录，并且文件也会回滚（慎用），如果不加 `--hard` 并不会丢失修改的文件内容

`git reset --hard` 后，想 push 到远程，要先 pull 远程代码下来，解决冲突，再提交，另外一种方法是用 `git push origin master -f` 强制推送，远程的代码就变成你推送时的代码

### 回到指定版本

使用 git revert head，一级级得进行回退

## 缓存（stash）

使用 stash 进行缓存文件，进行另外的开发

比如你在远程有一个 master 分支，你在本地 master 分支开发。当你正在开发的时候，接到一个紧急通知，要修改一个小 bug ，当你又不能把现在本地的代码提交上去，这时候使用 stash 解决

~~stash 不能把添加文件的操作进行缓存，而修改和删除的操作可以~~

1. git stash（不会把新添加的文件进行缓存）
2. git stash save -a （会把所有的文件进行缓存，包括 .gitignore）
3. git stash save -u  (不包括 .gitignore)
3. git stash save 'msg' （添加文本）
4. git add, git commit, git merge (修改 bug)
5. git stash pop (把缓存代码恢复)
6. 修改冲突（如果有冲突）  

* 缓存代码： git stash
* 查看缓存代码列表： git stash list
* 恢复缓存代码并删除： git stash pop
* 恢复缓存代码： git stash apply
* 删除缓存代码： git stash drop
* 恢复特定的缓存代码： git stash pply stash@{0}

## rebase

commit 合并操作

合并 top 几条的记录：git rebase -i HEAD~{number}

执行了 rebase 命令后，会出现编辑内容：

```
pick commitId feat 1
pick commitId feat 2
pick commitId feat 3
pick commitId feat 4
...
...
pick：保留该commit（缩写:p）

reword：保留该commit，但我需要修改该commit的注释（缩写:r）

edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）

squash：将该commit和前一个commit合并（缩写:s）

fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）

exec：执行shell命令（缩写:x）

drop：我要丢弃该commit（缩写:d）
```

可以修改每行 commit 记录最左边的操作符，根据不同的操作符有不同的效果

PS: 多用于在自己的分支下，对多个 commit 记录进行合并

资料：

1. [gitbase use](https://github.com/zuopf769/how_to_use_git/blob/master/使用git rebase合并多次commit.md)




## 忽略文件

.gitignore

如果要忽略某个原本已经在仓库的文件，要先删除它才行

## typechange

```shell
git status | grep typechange | awk '{print $2}' | xargs git checkout 
```

快速解决 typechange 问题

## emoji

在 commit 使用 emoji 表情

    git commit -m ':memo: 更新文档'

就会出现一个图标，详细的图标列表，[在这链接里](https://github.com/yiiouo/how-to-use-git)



# git commit 规范化和 changelog 自动生成



![整个 commit 提交的流程图](http://ppev1tg5s.bkt.clouddn.com/md/git commit 流程.png)



## git commit 规范化工具

### 参考链接

1. [cz-cli](https://github.com/commitizen/cz-cli)：用于规范化你的 commit 工具
2. [commit 规范 - 阮一峰](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)：使用 angluar commit 规范，changelog 自动生成教程
3. [git hooks 介绍](https://imweb.io/topic/5b13aa38d4c96b9b1b4c4e9d)
4. [npm package.json 介绍](<http://javascript.ruanyifeng.com/nodejs/packagejson.html>)



## cz-cli 安装和使用

```shell
// 全局安装工具
npm install -g commitizen

// 全局安装适配器，可选
npm install -g cz-conventional-changelog

// 设置适配器，可选
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc

// 在项目中初始化适配器
commitizen init cz-conventional-changelog --save-dev --save-exact

// 在项目中安装依赖
npm i cz-conventional-changelog cz-conventional-changelog -D

// 提交代码
git add ./
git cz

```

### commit 规范

>  以下内容，摘自[阮一峰老师的文章](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)，进行一定的删减、添加

Commit message 都包括三个部分：Header，Body 和 Footer

#### Header

包括三个字段：`type`（必需）、`scope`（可选）和`subject`（必需）

type:

- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- chore：构建过程或辅助工具的变动

scope:

用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同

比如我修改了用户模块，那就填写：用户模块/user module

subject:

commit 的简短描述，类似 title

- 以动词开头，使用第一人称现在时，比如`change`，而不是`changed`或`changes`
- 第一个字母小写
- 结尾不加句号（`.`）



#### Body

Body 部分是对本次 commit 的详细描述，可以分成多行，使用 `\n` 进行换行



#### Footer

Footer 部分只用于两种情况

1. 不兼容改动

    如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法

2. 关闭 issue

    Closes #234, #235



## commit 校验

### 参考链接

1. [commitlint 使用](https://conventional-changelog.github.io/commitlint/#/)
2. [@commitlint/config-angular](https://www.npmjs.com/package/@commitlint/config-angular)

### 安装使用

```shell
// 安装相关的依赖
npm install -D husky commit-msg @commitlint/{cli,config-angular}

// 新建配置文件
echo "module.exports = {extends: ['@commitlint/config-angular']};" > commitlint.config.js

```

配置 package.json 文件

```json
// package.json
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }  
  }
}
```

commitlint.config.js 配置参数

```js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    // Place your rules here
    'scope-enum': [2, 'always', ['a', 'b']] // error if scope is given but not in provided list
  }
}

{
  "types": ["a", "b"],              // 'type-enum': [2, 'always', ['a', 'b']]
  "scope": {
    "required": true,               // 'scope-empty': [2, 'never']
    "allowed": ["a", "b"],          // 'scope-enum': [2, 'always', ['a', 'b']]; specify [0] for allowed: ["*"]
    "validate": false,              // 'scope-enum': [0], 'scope-empty': [0]
    "multiple": false               //  multiple scopes are not supported in commitlint
  },
  "warnOnFail": false,              // no equivalent setting in commitlint
  "maxSubjectLength": 100,          // 'header-max-length': [2, 'always', 100]
  "subjectPattern": ".+",           // may be configured via `parser-preset`, contact us
  "subjectPatternErrorMsg": "msg",  // no equivalent setting in commitlint
  "helpMessage": "",                // no equivalent setting in commitlint
  "autoFix": false                  // no equivalent setting in commitlint
}
```

commitlint 配置项链接：

1. [commitlint 配置参数](https://conventional-changelog.github.io/commitlint/#/guides-upgrade?id=validate-commit-msg)
2. [rules 配置](https://conventional-changelog.github.io/commitlint/#/reference-rules)

配置完后，当你提交 commit 就会触发 npm hooks ，对 commit 内容进行检验



## changelog 自动生成

### 参考链接

1. [standard-version 使用](https://github.com/conventional-changelog/standard-version#how-is-standard-version-different-from-semantic-release)

### 安装使用

```shell
npm i -D standard-version
```

配置 package.json

```json
{
  "scripts": {
    "release": "standard-version"
  }
}
```

```shell
// use
npm run release -- --release-as 4.0.0 // 发布版本，并生成 changelog 和相关版本的 tag

git push --follow-tags origin master // 推送代码和 tag 到仓库

```

使用 `standard-version`，会多生成一个 commit，是提交 changelog 内容的









