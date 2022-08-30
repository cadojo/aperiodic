---
title: "Blog Posts"
subtitle: "_Aperiodically written._"
sidebar: false
feed: true

margin-header: ./subscribe.html

listing:
  contents: posts
  categories: numbered
  feed: true
  sort: "date desc"
  sort-ui: true
  filter-ui: true
  fields: [date, image, title, description, reading-time]

include-in-header:
  text: |
    <style>
    div.quarto-post .thumbnail img {border-radius: 7px;}
    </style>
  
aliases:
  - "/blog/posts"
  - "/blog/posts.html"
---

:::{.callout-tip appearance="simple"}
## RSS Feed
<https://aperiodic.dev/blog/index.xml>
:::
