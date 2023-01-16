# Next.js 学习

- 路由采用约定式路由，基于文件组织方式
- 静态资源放到 `public` 文件夹下
- 图片使用内置的 `Image` 组件
- 第三方 js 文件，例如 sdk，使用内置的 `Script` 组件
- 预渲染在 next.js 中起到了重要的作用，有两种预渲染方式：
  - Static Generation (SSG)：在 build 的时候就渲染生成了 html，适用于纯静态页面，所有用户在任意时刻看到的都是同样的页面
  - Server-side Rendering (SSR)：在每次请求的时候渲染生成 html，适用于要动态获取数据的页面。
- 每个页面使用哪一种预渲染方式可以独立指定。如果可以，尽量使用 SSG 方式，可以更好的利用 CDN 加速。
- 复杂交互页面还是使用客户端渲染的方式。
- 一个页面组件如果同时导出了 `getStaticProps` 函数，则说明他将采用 SSG 方式进行预渲染
- 一个页面组件如果同时导出了 `getServerSideProps` 函数，则说明他将采用 SSR 方式进行预渲染
- 约定：`getStaticProps` 以及 `getServerSideProps` 函数只能在页面级组件中使用
