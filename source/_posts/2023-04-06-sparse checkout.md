---
layout: post
title: Git指定文件提交和指定拉取详解
categories: Git
description: GitSparseCheckout
index_img:  https://xn--6or.us.kg/api/?raw
date: 2024-04-06 10:00:00
tags: [Git]
---

## 一、.gitignore使用指南
通常，在项目上使用Git的工作时，你会希望排除将特定文件或目录推送到远程仓库库中的情况。`.gitignore`文件可以指定Git应该忽略的未跟踪文件。

如何使用.gitignore忽略Git中的文件和目录。包括常见匹配模式\*星号，斜杠/，#井号注释，?问号，\[]方括号等通匹配符，一个.gitignore文件的示例，自定义排除忽略规则，全局的.gitignore配置，调试.gitignore文件，显示所有被忽略的文件

#### **应该忽略哪些文件**

> 被忽略的文件通常是特定于平台的文件或从构建系统自动创建的文件。一些常见的例子包括：运行时文件，例如日志，锁定文件，缓存或临时文件。具有敏感信息的文件，例如密码或API密钥。已编译的代码，例如`.class`或`.o`。依赖目录，例如`/vendor`或`/node_modules`。构建的输出目录，例如`/public`，`/out`或`/dist`。系统文件，例如`.DS_Store`或`Thumbs.db`。IDE或文本编辑器配置文件。

#### **`.gitignore`模式**

> `.gitignore`文件是纯文本文件，其中每行包含一个模式，用于忽略文件或目录。`.gitignore`使用 globbing pattern模式来匹配带通配符的文件名。如果文件或目录包含在通配符，则可以使用单个反斜杠（`\`）来转义字。

#### **注释**

> 以井号（`#`）开头的行是注释，将被忽略。空行可以用来提高文件的可读性，并可以对相关的模式行进行分组。

#### **斜杠符**

> 斜杠符号（`/`）是目录的分隔符。斜杠开头模式相对于`.gitignore`所在的目录。如果模式以斜杠开头，则仅从仓库的根目录中开始匹配文件和目录。如果模式不是以斜杠开头，则它将匹配任何目录或子目录中的文件和目录。
>
> 如果模式以斜杠结尾，则仅匹配目录。当目录被忽略时，其所有文件和子目录也将被忽略。

#### **文件名**

> 最直接的模式是没有任何特殊字符的文件名。例如/access.log仅匹配access.log。而access.log将会匹配当前目录与子目录 access.log，logs/access.log ，var/logs/access.log。当以/斜杠符号结束时则匹配目录。例如build/匹配build目录。

#### **通配符**

> `*`星号符号匹配零个或多个字符。例如\*.log模式将匹配error.log，logs/debug.log，build/logs/error.log等所有目录下以.log作为扩展名的文件。
>
> `**`两个相邻的星号符号匹配任何文件或零个或多个目录。当后跟斜杠（`/`）时，它仅与目录匹配。例如，logs/**将会匹配logs目录中所有文件与目录。**/build将匹配所有目录中出现以build命名目录与文件var/build，pub/build。
>
> 模式foo/\*\*/bar将匹配foo/bar，foo/a/bar，foo/a/b/c/bar。
>
> ?问号匹配单个任意字符。例如模式access?.log将会匹配access0.log，access1.log，accessA.log 。

#### **方括号**

> `[...]`方括号匹配方括号中包含的字符。当两个字符之间用连字符`-`隔开时，表示一个字符范围。该范围包括这两个字符之间的所有字符。范围可以是字母或数字。如果`[`之后的第一个字符是感叹号（`!`），则该模式匹配除指定集合中的字符以外的任何字符。
>
> 例如模式\*.\[oa]将匹配文件file.o，file.a。模式\*.\[!oa]将匹配file.s，file.1但不匹配file.0与file.a。

#### **反模式**

> 以感叹号（`!`）开头的模式将否定先前模式。此规则的例外是，如果排除了其父目录，则重新包含文件。例如模式 \*.log与!error.log这将会匹配所有以.log作为扩展名文件，但不匹配error.log。

#### **`.gitignore`范例**

> 以下是`.gitignore`文件的示例：

```bash
# 忽略node_modules目录
node_modules/

# 忽略Logs
logs
*.log

# 忽略/dist目录，相对.gitignore文件所在目录
/dist

# 忽略.env文件
.env

# 忽略IDE的配置文件
.idea/
.vscode/
*.sw*
```

#### **本地`.gitignore`**

> 本地`.gitignore`文件通常放置在仓库库的根目录中。但是，你可以在仓库的不同子目录中创建多个`.gitignore`文件。`.gitignore`文件中的模式相对于文件所在目录匹配。
>
> 在子目录中的文件中定义的模式优先于高于根目录中的模式。本地`.gitignore`文件与其他开发人员共享，并且应包含对存储库的所有其他用户有用的模式。

#### **个人忽略规则**

> 应在`.git/info/exclude`文件中配置特定于本地仓库且不应分发到其他仓库的模式。例如，你可以使用此文件忽略个人项目工具中生成的文件。

#### **全局`.gitignore`**

> Git还允许你创建全局`.gitignore`文件，你可以为本地系统上的每个Git仓库定义忽略规则。该文件可以命名为任意名称，并存储在任何位置。保存此文件的最常见位置是主目录。你必须手动创建文件并配置Git使用它。
>
> 例如，要将`~/.gitignore_global`设置为全局Git忽略文件，你可以执行以下操作。首先创建文件：

```bash
touch ~/.gitignore_global
```

> 将文件添加到Git配置：

```bash
git config --global core.excludesfile ~/.gitignore_global
```

> 使用文本编辑器打开文件并向其中添加规则。全局规则对于忽略你永远不想提交的特定文件（例如带有敏感信息或已编译的可执行文件的文件）特别有用。

#### **忽略以前提交的文件**

> 你的工作副本中的文件可以被追踪，也可以不被追踪。要忽略先前提交的文件，你需要取消暂存并从索引中删除该文件，然后在`.gitignore`中添加该文件模式：

```bash
git rm --cached filename
```

> `--cached`选项告诉git不要从工作树中删除文件，而只是从索引中删除它。要递归删除目录，请使用`-r`选项：

```bash
git rm --cached filename
```

> 如果要从索引和本地文件系统中删除文件，请忽略`--cached`选项。以递归方式删除文件时，使用`-n`选项将执行空运行并显示要删除的文件：

```bash
git rm -r -n directory
```


#### **调试`.gitignore`文件**

> 有时候，确定为什么要忽略特定文件可能会很困难，尤其是当你使用多个`.gitignore`文件或复杂格式时。这是`git check-ignore`命令的用处，告诉git显示匹配模式的详细信息。
>
> 例如，要检查为什么忽略`www/yarn.lock`文件，可以运行：

```bash
git check-ignore -v www/yarn.lock
```

> 输出显示`gitignore`文件的路径，匹配行的编号和实际模式。

```bash
www/.gitignore:31:/yarn.lock www/yarn.lock
```

> 该命令还接受多个文件名作为参数，并且文件不必存在于你的工作树中。

#### **显示所有被忽略的文件**

> 带有`--ignored`选项的`git status`命令显示所有被忽略文件的列表：

`git status --ignored`

## 二、Git sparse Checkout使用指南

Git的clone，默认是直接拉取整个远程仓库，如果项目比较大，在进行clone时，会导致拉取到大量和自己无关的内容到本地，占用很多硬盘空间。

Git在1.7版本后，已经支持只Checkout部分内容，这个功能叫做 Sparse Checkout（稀疏检出），使用该功能可以节省本地硬盘空间。\

**使用步骤如下：**

#### 0.准备工作：如果本地还没有版本库，则先执行下述命令

```bash
git init <project>
cd <project>
git remote add origin ssh://<user>@<repository's url>
```

#### 1.开启sparse checkout功能

```bash
git config core.sparsecheckout true
```

或者\
git sparse-checkout init\

#### 2.写入要获取的文件

1）写入文件：\
echo “x x x” >> .git/info/sparse-checkout

例如：

```bash
echo "Test" >> .git/info/sparse-checkout 
```

表示只拉取Test文件夹，\
或者使用命令\
git sparse-checkout set Test

##### 附：sparse checkout文件设置

```
子目录的匹配
```

在 sparse-checkout 文件中，如果目录名称前带斜杠，如/docs/，将只匹配项目根目录下的docs目录，如果目录名称前不带斜杠，如docs/，其他目录下如果也有这个名称的目录，如test/docs/也能被匹配。\
而如果写了多级目录，如docs/01/，则不管前面是否带有斜杠，都只匹配项目根目录下的目录，如test/docs/01/不能被匹配。

```
通配符 “*“ (星号)
```

在 sparse-checkout 文件中，支持通配符 “\*“，如可以写成以下格式：\
_docs/\
index._\
\*.gif

```
排除项 “!” (感叹号)
```

在 sparse-checkout 文件中，也支持排除项 “!”，如只想排除排除项目下的 “docs” 目录，可以按如下格式写：\
/\*\
!/docs/

注意：如果要关闭sparsecheckout功能，全取整个项目库，可以写一个”\*“号，但如果有排除项，必须写成”/星号“，同时排除项要写在通配符后面。

2）执行git pull/fetch命令\
（每次更改sparse-checkout增加或删除目录时，都需要执行一次该命令）。\

#### 3.关闭sparse checkout功能

使用命令：\
git sparse-checkout disable