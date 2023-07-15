---
title: Github敏感信息搜索
tags:
---


## 出发点

最近做敏感信息收集项目，需要做 Github 的敏感信息收集，特此写一篇文章记录一下比较有用的搜索语法

## 常用搜索语法

### 搜索仓库

| 限定符 | 示例 | 
|-----------------|----------------------------------------------------| 
| `in:name` | `jquery in:name` 匹配名称中带有"jquery"的存储库。 | 
| `in:description` | `jquery in:name,description` 匹配名称或说明中带有"jquery"的存储库。 | 
| `in:topics` | `jquery in:topics` 将带"jquery"标签的存储库匹配为主题。 | 
| `in:readme` | `jquery in:readme` 匹配自述文件中提及"jquery"的存储库。 | 
| `repo:owner/name` | `repo:octocat/hello-world` 匹配特定的存储库名称。|

### 搜索 Commit

| 限定符 | 示例 | 
|--------------------|-----------------------------------------------| 
| `author:USERNAME` | `author:defunkt` 匹配作者 ``@defunkt` 的提交。 | 
| `committer:USERNAME` | `committer:defunkt` 匹配提交者 ``@defunkt` 的提交。 |
| `author-name:NAME` | `author-name:wanstrath` 匹配作者姓名中包含“wanstrath”的提交。| 
| `committer-name:NAME` | `committer-name:wanstrath` 匹配提交者姓名中包含“wanstrath”的提交。 |
| `author-email:EMAIL` | `author-email:chris@github.com` 匹配作者 `chris@github.com` 的提交。 | 
| `committer-email:EMAIL` | `committer-email:chris@github.com` 匹配提交者 `chris@github.com` 的提交。 |


### 搜索 Pull Requests 和 Issues

| 限定符 | 示例 | 
|--------------|-------------------------------------------| 
| `type:pr` | `cat type:p`r` 匹配包含 "cat" 一词的拉取请求。 | 
| `type:issue` | `github commenter:defunkt type:issue` 匹配包含 "github" 一词且包含 `@defunkt` 的评论的问题。 | 
| `is:pr` | `event is:pr` 匹配包含 "event" 一词的拉取请求。 | 
| `is:issue` | `is:issue label:bug is:closed` 匹配包含 "bug" 标签的已关闭问题。|
| `in:title` | `warning in:title` 匹配标题中包含 "warning" 的问题。 | 
| `in:body` | `error in:title,body` 匹配标题或正文中包含 "error" 的问题。 | 
| `in:comments` | `shipit in:comments` 匹配在评论中提及 "shipit" 的问题。 |
| `involves:USERNAME`      | `involves:defunkt involves:jlord` 匹配涉及 `@defunkt` 或 `@jlord` 的问题。     |
| `in:body involves:USERNAME` | `NOT bootstrap in:body involves:mdo` 匹配涉及 `@mdo` 且正文中不包含“bootstrap”一词的问题。 |
| `label:LABEL`                      | `label:"help wanted" language:ruby` 匹配标签为“help wanted”且位于 Ruby 存储库中的问题。|
| `in:body -label:LABEL label:LABEL` | `broken in:body -label:bug label:priority` 匹配正文中包含“broken”一词且没有“bug”标签但有“priority”标签的问题。|
| `label:LABEL label:LABEL`          | `label:bug label:resolved` 匹配标签为“bug”和“已解决”的问题。|
| `label:LABEL,LABEL`                | `label:bug,resolved` 匹配标签为“bug”或“已解决”的问题。|
|`mentions:USERNAME`| `resque mentions:defunkt` 匹配提及 `@defunkt` 且包含“resque”一词的问题。|
|`team:ORGNAME/TEAMNAME`|`team:jekyll/owners` 匹配提及 `@jekyll/owners` 团队的问题。|
|`team:ORGNAME/TEAMNAME` `is:open is:pr`|`team:myorg/ops is:open is:pr` 匹配提及 `@myorg/ops` 团队的打开的拉取请求。|
|`commenter:USERNAME`| `github commenter:defunkt org:github` 匹配 GitHub 拥有的存储库中含有“github”一词且包含 `@defunkt` 评论的问题。|

### 搜索 Fork 信息

|限定符|示例|
|---|---|
|`fork:true`| `github fork:true` 与所有包含“github”一词的存储库匹配，包括分支。|
|`language:_LANGUAGE_` `fork:true`| `android language:java fork:true` 与具有分支和常规存储库中用 Java 编写的“android”一词的代码相匹配。|
|`fork:only`| `github fork:only` 与所有包含“github”一词的分支存储库匹配。|
|`forks:>_n_` `fork:only`| `forks:>500 fork:only` 与具有超过 500 个分支的存储库匹配，并且仅返回分支存储库。|

### 搜索 Gist

|限定符|示例|
|:--|:--|
|`anon:true`| `cat anon:true` 在与 cat 相关的 Gist 搜索中包括匿名 Gist 。|
|`filename:_FILENAME_`| `filename:.bashrc` 查找包含“.bashrc”文件的所有 Gist 。|
|`language:_LANGUAGE_`|`cat language:html` 查找包含 HTML 文件且具有“cat”一词的所有 Gist 。|
|`extension:_EXTENSION_`|`join extension:coffee` 在具有 coffee 扩展的 Gist 中查找“join”的所有实例。|

### 查询长度限制

在 GitHub 上搜索时，查询的长度有一些限制：

- 查询字符不超过**256个**；
- 查询中的逻辑运算符（`AND`、`OR` 或 `NOT`） 不能使用超过**5个** ；

## 敏感信息检索示例

常用搜索语法：

```
content:<regex> in:description,readme

content:

```

常用正则表达式（regex）：`(/<regex>/)

```
# email
(/[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+/) 

# host
(/^[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$/)

# IP
/((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}/
```
