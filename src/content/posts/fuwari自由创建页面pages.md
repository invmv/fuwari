---
published: 2024-12-08
title: Fuwari自由创建页面Pages
description: 默认只有about页面，创建一个路由。自动识别新创建的页面
draft: false
category: other
tags:
  - 博客
---
# 创建路由

创建文件：```src\pages\[...slug].astro```

代码
```
---

import MainGridLayout from '../layouts/MainGridLayout.astro'

import { getEntry, getCollection } from 'astro:content'


import Markdown from '@components/misc/Markdown.astro'

// 静态路径生成
export const getStaticPaths = async () => {
    const pages = await getCollection('spec')
    return pages.map((page) => ({
        params: { slug: page.slug },
    }))
}

// 获取动态内容
const { slug } = Astro.params
const pageContent = await getEntry('spec', slug)
const { Content } = await pageContent.render()
---

<MainGridLayout title={slug}>
    <section class="flex w-full rounded-[var(--radius-large)] overflow-hidden relative min-h-32">
        <div class="card-base z-10 px-9 py-6 relative w-full">
            <Markdown class="mt-2">
                <Content />
            </Markdown>
        </div>
    </section>
</MainGridLayout>
```

# 创建页面

本身About页面就放在```spec```，就不修改其他目录了。

比如我想创建路径为 new 的页面，则 ```src\content\spec\new.md```

new.md内容 正常填写md内容即可，
```
# 我是大标题
新页面测试

```
不需要前置文章头，如果你想插入html代码，直接插入即可。实时渲染这与hugo不同

我则整合了decap cms，用title当slug。后台就可以正常创建和编辑pages了

```
---
title: new
---
新页面测试

```
