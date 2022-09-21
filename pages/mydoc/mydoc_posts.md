---
title: LoadBalance(负载均衡)
tags: [getting_started, formatting, content_types]
keywords: posts, blog, news, authoring, exclusion, frontmatter
last_updated: Feb 25, 2016
summary: "TCP路由，网关协议转发等内容收录在此。"
sidebar: mydoc_sidebar
permalink: mydoc_posts.html
folder: network
---

## 2021.07.13 我们是这样崩的

```yaml
研发配置错误权重，导致4层LB七层SLB陷入死循环, 下游服务无法接受响应
```
[原文链接](https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NTgwMg==&mid=2247487272&idx=1&sn=038a30ce61706c97e3397eee982b1486&scene=21#wechat_redirect )

[拓展阅读:负载均衡的前世今生](https://cloud.tencent.com/developer/article/1458765)

## Allowed frontmatter

The frontmatter you can use with posts is as follows:

```yaml
---
title: My sample post
tags: content_types
keywords: pages, authoring, exclusion, frontmatter
sidebar: mydoc_sidebar
permalink: mydoc_pages.html
summary: "This is some summary frontmatter for my sample post."
---
```

| Frontmatter | Required? | Description |
|-------------|-------------|-------------|
| **title** | Required | The title for the page |
| **tags** | Optional | Tags for the page. Make all tags single words, with underscores if needed. Separate them with commas. Enclose the whole list within brackets. Also, note that tags must be added to \_data/tags_doc.yml to be allowed entrance into the page. This prevents tags from becoming somewhat random and unstructured. You must create a tag page for each one of your tags following the sample pattern in the tabs folder. (Tag pages aren't automatically created.)  |
| **keywords** | Optional | Synonyms and other keywords for the page. This information gets stuffed into the page's metadata to increase SEO. The user won't see the keywords, but if you search for one of the keywords, it will be picked up by the search engine.  |
| **sidebar** | Required | Refers to the sidebar data file for this page. Don't include the ".yml" file extension for the sidebar &mdash; just provide the file name. If no sidebar is specified, this value will inherit the `default` property set in your \_config.yml file for the page's frontmatter. |
| **permalink**| Required | This theme uses permalinks to facilitate the linking. You specify the permalink want for the page, and the \_site output will put the page into the root directory when you publish. Follow the same convention here as you do with page permalinks -- list the file name followed by the .html extension. |
| **summary** | Optional | A 1-2 word sentence summarizing the content on the page. This gets formatted into the summary section in the page layout. Adding summaries is a key way to make your content more scannable by users (check out [Jakob Nielsen's site](http://www.nngroup.com/articles/corporate-blogs-front-page-structure/) for a great example of page summaries.) The only drawback with summaries is that you can't use variables in them. |


{% include links.html %}
