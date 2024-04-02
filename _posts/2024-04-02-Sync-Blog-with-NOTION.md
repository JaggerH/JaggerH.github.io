---
id: 7f620f02-7b7b-4e30-9d19-c81b00e0d3fd
title: Sync Blog with NOTION
created_time: 2024-04-02T04:05:00.000Z
last_edited_time: 2024-04-02T05:19:00.000Z
tags:
  - Github
  - Notion
date: '2024-04-02'
status: Ready
layout: post
_thumbnail: /assets/img/favicon_f8fO6Cee.svg

---

Codespace is good, but not good enough when you have lots of blog.

# Why Notion

我可以在Codespace中直接编辑markdown，但是当我有图片需要管理和同步的时候，这一切就开始变得复杂起来。

[Notion](https://notion.so/)可以做很多事，多到超出想象，尤其适合用于编辑文档，并且还开放了很多接口，普通用户也可以用于开发。

**是不是可以把Notion上的文章，同步到Github Page上？**

**可以！你可以在Notion上管理你的Blog！**

# How

实现同步需要用到这个项目

> [![favicon](/assets/img/favicon_f8fO6Cee.svg) **GitHub**](https://github.com/victornpb/notion-jam)\
> Sync pages from Notion to GitHub to be used as a static website (JAM) - victornpb/notion-jam\
> <https://github.com/victornpb/notion-jam>

感谢victornpb，大家可以给大神赞助一波。

## `notion-jam`是怎么工作的

你需要在notion上面维护一个notion database，并且将这个页面共享给github，Github Action执行时会抓取这个database下的页面。

# 怎么做

基础操作不再赘述，参照notion-jam的指引即可

我会着重说明一下，实施过程中原项目指引中缺省的一些问题

## Create Database

### Database的格式

你的Database应该长这样，注意大小写

| Name   | Type         | Options                   |
| ------ | ------------ | ------------------------- |
| Title  | Text         |                           |
| Tags   | Multi-select |                           |
| Status | Select       | Options: Ready, Published |
| layout | Select       | Options: post             |
| Date   | Text         |                           |

![](/assets/img/Untitled_FCnhTliu.png)

### Github Settings

*   Create a workflow in `.github/workflows/**.yml` of your repository

    这个Action会做什么事情

    *   读取notion database

    *   提交到Github

    *   编译页面并发布

```bash
# This is a basic workflow to help you get started with Actions
# step 1. read from notion database
# step 2. commit changes
# step 3. build pages
# step 4. deploy
name: Deploy Jekyll site to Pages

# Controls when the workflow will run
on:
  schedule:
    - cron: "0 21 * * *" # daily
  push:
    branches: [master, main]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: write
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3

      - name: notion-jam
        uses: victornpb/notion-jam@v0.0.13
        with:
          NOTION_SECRET: ${{ secrets.NOTION_SECRET }}
          NOTION_DATABASE: ${{ vars.NOTION_DATABASE }}
          FILTER_PROP: Status
          FILTER_VALUES: Ready,Published
          CONVERT_PROP_CASE: snake
          ARTICLE_PATH:  _posts/{date}-{title}.md
          ASSETS_PATH: assets/img
          PARALLEL_PAGES: 25
          PARALLEL_DOWNLOADS_PER_PAGE: 3
          DOWNLOAD_IMAGE_TIMEOUT: 30
          SKIP_DOWNLOADED_IMAGES: true
          DOWNLOAD_FRONTMATTER_IMAGES: true
          
      - name: Replace image paths in Markdown files
        run: |
          sed -i 's|/assets/img/|//assets/img/|g' $(find _posts -type f -name "*.md")
      
      - name: Save changes
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Commit changes

      - name: Setup Ruby
        uses: ruby/setup-ruby@8575951200e472d5f2d95c625da0c7bec8217c42 # v1.161.0
        with:
          ruby-version: '3.1' # Not needed with a .ruby-version file
          bundler-cache: true # runs 'bundle install' and caches installed gems automatically
          cache-version: 0 # Increment this number if you need to re-download cached gems
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Jekyll
        # Outputs to the './_site' directory by default
        run: bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
        env:
          JEKYLL_ENV: production
      - name: Upload artifact
        # Automatically uploads an artifact from the './_site' directory by default
        uses: actions/upload-pages-artifact@v3

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

*   在你的Repo中，Settings→Actions→General

    在**Workflow permissions**一节，选择**Read and Write rermissions**

# 一些踩过的坑

### Notion中怎么Connect notion-jam

在目前版本的notion中，你需要在Page中点击Connect to → notion-jam

![](/assets/img/1d0e3922-081d-497c-9c60-6c0b560ca95d_lQn4VmYx.png)
