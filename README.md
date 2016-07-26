个人博客，主题修改自 [https://github.com/taizilongxu/jekyll_blog_source](https://github.com/taizilongxu/jekyll_blog_source)

使用请fork 这里 [https://github.com/BorntoGO/jekyll_blog_source](https://github.com/BorntoGO/jekyll_blog_source)

使用注意：

- 首页文章列表改为最近10篇中的3篇 --> index.html/{% for post in site.related_posts limit: 3 %}；
- 修改多说评论为Disqus,请替换帐号ID --> _includes/custom/disqus.html；
- 请替换 Google analytics 帐号ID --> _includes/head.html
- github发布时，请修改 `.gitignore` 文件，去除 `_posts` 过滤；


