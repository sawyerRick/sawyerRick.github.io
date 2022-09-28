---
title: AddingImgsToHexoBlogByTyporaTheSimpleWay
date: 2022-09-29 02:16:47
tags: Misc
cover: topTodos.jpeg
typora-root-url: AddingImgToHexoBlogByTyporaTheSimpleWay
---

# Adding Imgs To Hexo Blog By Typora The SimpleWay

Only 2 steps will do the job.

## step1

Go to your hexo root directory, edit `_config.yml`.

```
post_asset_folder: true
```

As a result, every time I run hexo new [fooBlog] command I got a empty [fooBlog] directory in my _posts.

> Hint: switch [fooBlog] to your real blog title.

## step2

Add this line to fooBlog's metadata section.

```json
typora-root-url: fooBlog
```

## sample

Say I have a img named topTodos.jpeg in [fooBlog] directory mentioned above.

Img inserting markdown code be like:`![](topTodos.jpeg)`

Result:

![topTodos](topTodos.jpeg)

## Reference

First paragraph on https://hexo.io/zh-cn/docs/asset-folders.html

Paragraph 'Relative path to certain folder' on https://support.typora.io/Images/

