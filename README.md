<h1 align=center>道草博客</h1>

## 关于我

```js
// On this page, you’ll find  about me.
const my = {
  name: '道草',
  tags: ['Web Developer','Photog','Cycling enthusiast', 'INFJ'],
  adress: 'NeiMengGu - TongLiao',
  email: 'bdstravel@126.com',
  github: 'https://github.com/bdsdc',
  blog: 'https://bdsdc.github.io/',
  RSSReader: 'https://bdsdc.github.io/rss/',
}
```

<h2 align=center>ops/ trader / Work at Shanghai</h2>


### 基础使用

    新建博客: hugo new site NewBolgSite
    新建文章: hugo new blog/xxx-文章名.md
    启动博客: hugo server -D
    编译博客：hugo -D

    ---
    title: Git 仓库的分支 (Branch) 规范
    date: 2018-11-27 19:31:30
    lastmod: xxx
    tags: ["Git", "代码管理", "规范"]
    series: ["代码管理"]
    category: ["blog"]
    featured: true
    ---

    title 文章题目
    date 发布日期
    tags 标签分类，用于描述文章的主题、关键词。通过标签，可以根据兴趣和需求，快速找到相关的文章
    series 系列文章，用于将同一系列相关的文章组织在一起。会在下方推荐阅读中推荐同系列文章
    category 类别分类、比如技术、生活、旅行等
    featured 是否在主页面中展示，true or false

## 最小配置

Clone your repository.

Build and run hugo server by `hugo server -D` and open in browser http://localhost:1313/.

```yml

baseURL = "https://hugodoit.pages.dev"
# [en, zh-cn, fr, pl, ...] determines default content language
# [en, zh-cn, fr, pl, ...] 设置默认的语言
defaultContentLanguage = "en"
# website title
# 网站标题
title = "DoIt"

# whether to use robots.txt
# 是否使用 robots.txt
enableRobotsTXT = true
# whether to use git commit log
# 是否使用 git 信息
enableGitInfo = true
# whether to use emoji code
# 是否使用 emoji 代码
enableEmoji = true

# ignore some build errors
# 忽略一些构建错误
# ignoreErrors = ["error-remote-getjson", "error-missing-instagram-accesstoken"]

# theme
# 主题
theme = "DoIt"
# themes directory
# 主题目录
themesDir = "../.."

# Or, when using theme as hugo module
# [module]
# [[module.imports]]
# path = "github.com/HEIGE-PCloud/DoIt"
```

新的图标可以通过修改配置文件添加，params.social 字段标明 名称，图标，自定义地址，图标可以在这个网站找到 feathericons.com。

Modifying the default configuration. Then push it to your repository.

## Special Thanks

-   Hugo Ladder is inspired by [DoIt](https://github.com/HEIGE-PCloud/DoIt)
