---
image: /uploads/decap.jpg
published: 2024-12-04
title: 搭建Astro博客使用Fuwari主题+Decap 无头CMS+Cloudflare Pages部署+GitHub Oauth认证
description: 全静态Pages方案，现使用cloudflare pages+ github Oauth. 转至Astro Fuwari搭建博客
  cloudflare pages静态和decap CMS无头后台一次部署省心的更新博客
draft: false
category: other
tags:
  - 博客
---
![](/uploads/clipboard-image-1733367110.avif)

# 克隆

先fork或使用模板到自己仓库，https://github.com/saicaca/fuwari

# 部署

使用cloudflare Pages按照astro教程部署 https://docs.astro.build/en/guides/deploy/

并设置 自定义域名

# 创建Oauth apps

https://github.com/settings/developers

`Homepage URL` 和 `Authorization callback URL` 填写你的自定义域名

你会得到`GITHUB_CLIENT_ID`和`GITHUB_CLIENT_SECRET`，将它们填写给cloudflare Pages

# Decap CMS

仓库代码：https://github.com/i40west/netlify-cms-cloudflare-pages

functions放在根目录

static/admin  改名为pulic。即放在/pulic/admin下

并修改config.yml，主要修改repo和base_url

# config

```yaml
backend:
  name: github
  repo: user/repo #你的github仓库
  branch: main # Branch to update (optional; defaults to master)
  base_url: https://home.com #你的pages网址
  auth_endpoint: /api/auth
media_folder: public/uploads
public_folder: /uploads
collections:
  - name: "Posts"
    label: "Posts"
    folder: "src/content/posts"
    slug: "{{title | slugify}}"  # 自动生成的文件名格式
    summary: "{{published}} {{title}}"
    create: true
    fields:
      - { label: "封面", name: "image", widget: "image", required: false}
      - { label: "日期", name: "published", widget: "datetime", format: "YYYY-MM-DD" }
      - { label: "标题", name: "title", widget: "string"}
      - { label: "描述", name: "description", widget: "text" }

      - label: "草稿"
        name: "draft"
        widget: "boolean"
        default: false

      # 分类字段，用户可以从预定义分类中选择
      - label: "分类"
        name: "category"
        widget: "select"
        options:
          - "review"
          - "mytrade"
          - "other"
        default: "other"  # 设置默认值
      
      # 标签字段，用户可以自由添加多个标签
      - label: "标签"
        name: "tags"
        widget: "list"
        allow_add: true
        hint: "标签使用英文逗号+空格隔开，如标签1, 标签2, 标签3"
      
      - name: "body"
        label: "正文"
        widget: "markdown"
        modes: ["raw", "rich_text"]

  - name: "spec"
    label: "spec"
    folder: "src/content/spec"
    slug: "{{title | slugify}}"
    create: true
    fields:
      - { label: "路径", name: "title", widget: "string"} #即文件名也是路径，正文不显示
      - { label: "Body", name: "body", widget: "markdown" }
```
