---
id: 0a179547-d14c-442c-8661-d9757449f195
title: Create Blog on Github in FIVE minutes
created_time: 2024-03-31T17:07:00.000Z
last_edited_time: 2024-04-02T05:14:00.000Z
cover_image: /assets/img/nasa_transonic_tunnel_Gzc9uijW.jpg
tags:
  - Github
  - Jekyll
date: '2024-04-01'
status: Ready
layout: single
_thumbnail: /assets/img/nasa_transonic_tunnel_Gzc9uijW.jpg

---

如果你考虑过把自己的经验写成Blog，记录自己的经历，画上五分钟，你可以拥有一个自己的博客。本文介绍如何使用 GitHub Pages + Jekyll 快速搭建个人博客。

<!--more-->

## 创建一个空的项目，命名必须是你的Github名称 + github.io

举个例子，你的名字叫`WTF`，那你的项目名称就是`WTF.github.io`

![](/assets/img/Untitled_0pHez1yE.png)

## 创建你的博客

### 进入Codespace

进入项目中，你会看到那个绿色的`code`，点他你会进入Codespace，这意味着你不需要在自己的电脑上配置任何环境。顺便一提Codespace这个东西真的强的一批，居然还带了VS Code编辑器。

![](/assets/img/Untitled_CxM6URJJ.png)

### 安装Jekyll

进入到Codespace之后，你的界面是这样的，在终端里面输入下面的代码

![](/assets/img/Untitled_zxPWQiPq.png)

```bash
gem install jekyll bundler

jekyll new ./

bundle install
bundle exec jekyll serve
```

### 恭喜你彦祖，你的Blog已经打开了

网页上会提醒你，你运行了4000端口，选择打开，之后你就会看到

![](/assets/img/Untitled_dUiCRCz6.png)

### 安装主题

可以在[rubygems](https://rubygems.org/search?query=jekyll-theme)中搜索主题，我选了[minimal](https://github.com/pages-themes/minimal)

```yaml
# Build settings
remote_theme: pages-themes/minimal@v0.2.0
plugins:
  - jekyll-remote-theme # add this line to the plugins list if you already have one
  - jekyll-feed
```

```bash
source "https://rubygems.org"
# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
gem "jekyll", "~> 4.3.3"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.5"
# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
# gem "github-pages", group: :jekyll_plugins
# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-remote-theme"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb", "~> 0.6.0", :platforms => [:jruby]
```

```bash
bundle install
bundle exec jekyll serve
```

在终端中运行，你会看到两个Warning，并且页面也是空白的无法显示

那是因为`about.markdown`和`index.markdown`使用的layout在minimal中不存在，你需要修改为default

```bash
---
layout: default
---
```

修改后你的页面可以正常显示了。

如果你习惯与使用Markdown编程，到这一步为止已经很完美了，如果你需要添加帖子，你只需要在\_post文件夹中，新建格式如 `2024-04-02-NEW POST.md`即可，依据Markdown的规范来编写，提交到Github之后，Github会运行Action自动编译，你的博客也会自动更新。

### 和Notion同步
