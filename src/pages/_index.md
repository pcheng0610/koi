## 关于我

正在加载中...

## 关于这里

发点没用的，记点刚编的

## 最近更新

查看 [博客文章](/blog) 了解更多内容。

## 鸣谢

模板：https://github.com/tcdw/koi

基于 [Astro](https://astro.build/) 官方博客模板脚手架修改而来，提供了预先配置好的 Astro 配置文件 `astro.config.js` 和 Tailwind CSS 配置文件 `src/styles/global.css`，同时还将博客常见设置封装在 `src/consts.ts` 中，这样可以在不修改模板本体的代码的情况下，就能实现基础的个性化需求。

<details>
<summary>我想让首页直接展示博文列表</summary>

将本页面 `src/index.astro` 替换为如下内容：

```astro
---
import { filterPosts } from "@/utils/misc";
import { getCollection } from 'astro:content';
import BlogPostList from "@/layouts/BlogPostList.astro";
import { BLOG_PAGINATION_SIZE } from "@/consts";

const posts = filterPosts(await getCollection('blog'), {
    filterDraft: true,
    filterUnlisted: true,
});
const latestPosts = posts.slice(0, BLOG_PAGINATION_SIZE);
---

<BlogPostList data={latestPosts} currentPage={1} lastPage={Math.ceil(posts.length / BLOG_PAGINATION_SIZE)} />
```

然后删除 `src/_index.md` 即可。
</details>
