---
title: VS Code的python环境配置
date: 2020-01-26 13:58:13
categories:
- 花里胡哨的东西
tags:
- vscode
- python
---

>我个人习惯把VS Code作为编辑器来写python, 但是使用过程中会遇到一些小小的问题，所以在这里把问题的解决方案记录下来，以便以后查阅，目前还在持续更新中。

<!--more-->

### pylint提示no-member

比如对于一些第三方库，pytorch或者自己写的库之类的，pylint会经常提示一些函数no-member，我们可以在配置(setting.json)里面加入要忽略错误的第三方库。

```json
"python.linting.pylintArgs": [
 # List of members which are set dynamically and missed by pylint inference
 # system, and so shouldn't trigger E1101 when accessed. Python regular
 # expressions are accepted.
 # https://github.com/pytorch/pytorch/issues/701
 	"--generated-members=numpy.* ,torch.* ,cv2.* , cv.*"
 ],
```

### 使用cmder作为默认终端

cmder集成了一些linux的指令，比如head,ls,cat,echo等，所以使用起来非常方便，可以很大的提高效率。

```json
# https://code.visualstudio.com/docs/editor/integrated-terminal#_configuration 
"terminal.integrated.shell.windows": "cmd.exe",
"terminal.integrated.shellArgs.windows": ["/K", "D:/cmder/vendor/init.bat"] # 这个填自己电脑里cmder的位置就可
```

顺便一提，如果你下的cmder不是便携版而是完整版，那它是自带git的，也省得下载git了，vscode配置git可以加上

```json
"git.path": "d:/cmder/vendor/git-for-windows/bin/git.exe"
```

### 一些不错的插件

- 名称: **autoDocstring**
  id: njpwerner.autodocstring
  说明: Generates python docstrings
  版本: 0.4.0
  发布者: Nils Werner
  VS Marketplace 链接: https://marketplace.visualstudio.com/items?itemName=njpwerner.autodocstring
- 名称: **Todo Tree**
  id: gruntfuggly.todo-tree
  说明: Show TODO, FIXME, XXX, etc. comment tags in a tree view
  版本: 0.0.169
  发布者: Gruntfuggly
  VS Marketplace 链接: https://marketplace.visualstudio.com/items?itemName=Gruntfuggly.todo-tree
- 名称: Bracket Pair Colorizer
  id: coenraads.bracket-pair-colorizer
  说明: A customizable extension for colorizing matching brackets
  版本: 1.0.61
  发布者: CoenraadS
  VS Marketplace 链接: https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer