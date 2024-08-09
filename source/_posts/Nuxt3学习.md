---
title: Nuxt3学习
tags:
  - Nuxt3
  - vue3
  - ts
author: Eric
date: 2024-8-5
comments: false
cover: /img/3.png
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

1. SSR 为什么要用
   > - 更快的首屏加载
   > - 统一的模式
   > - 更好的 SEO 优化：搜索引擎爬虫可以直接看到
2. 什么是 SEO

   > - SEO（Search Engine Optimization）即搜索引擎优化，是指通过对网站进行技术优化和内容优化，提升网站在搜索引擎中的排名，从而增加网站的曝光度和流量的过程。SEO 的目标是通过优化网站，使其能够更好地> 被搜索引擎理解和收录，提高网站在搜索结果中的排名，从而获得更多的有机流量。
   > - SEO 的定义可以从两个方面来解释。首先，从技术角度来看，SEO 是通过对网站的结构、代码、速度等方面进行优化，使其更符合搜索引擎的算法要求，从而提高网站在搜索结果中的排名。其次，从用户体验角度来看，>SEO 也包括对网站内容的优化，使其更具有吸引力和价值，从而吸引更多的用户点击进入网站。
   > - SEO 的定义中还包括了两个重要的概念，即搜索引擎和优化。搜索引擎是指通过算法对互联网上的网页进行检索和排名，为用户提供相关的搜索结果的工具。优化是指通过技术手段和优化策略，使网站更符合搜索引擎的>要求，从而提高网站在搜索结果中的排名。
   > - 总而言之，SEO 是一种通过优化网站技术和内容，提高网站在搜索引擎中的排名，从而获得更多有机流量的方法和策略。随着互联网的不断发展，SEO 已经成为网站推广和营销的重要手段，对于提升网站的曝光度和影响力具有重要意义。

3. SEO 的原理

   > - EO 的原理主要包括以下几个方面：关键词优化、网站结构优化、内容优化和外部链接优化。首先，关键词优化是指通过研究用户的搜索习惯和需求，选择合适的关键词，并在网页的标题、内容、标签等位置合理地使用这些关键词，以提高网页在搜索结果中的排名。
   > - 其次，网站结构优化是指通过合理的网站设计和布局，使搜索引擎能够更好地理解和索引网页的内容。包括优化网页的 URL 结构、导航菜单、内部链接等方面。
   > - 再次，内容优化是指通过提供高质量、有价值的内容，吸引用户的访问和分享，从而提高网站的权威性和可信度。内容优化包括对网页内容的优化、图片和视频的优化等。
   > - 最后，外部链接优化是指通过获取高质量的外部链接，提高网站的权威性和可信度。外部链接的数量和质量对网站的排名影响很大，因此需要通过合法手段，获取其他网站的链接。
   > - 综上所述，SEO 的原理是通过优化关键词、网站结构、内容和外部链接等方面，提高网站在搜索引擎中的排名，从而增加网站的流量和曝光度。

4. Nuxt 语法

   > Nuxtpage NuxtLink

5. Nuxt3 请求方式

   > Nuxt3 提供了四种常用的请求方式，‌ 分别是 useAsyncData、‌useFetch、‌useLazyAsyncData 和 useLazyFetch。‌ 这些方法用于在 Nuxt3 应用程序中异步获取和处理数据。‌

   - useAsyncData：‌ 这个方法允许你在页面、‌ 组件和插件中访问异步解析的数据。‌ 它适用于需要在服务器端或路由更新之前获取数据的场景。‌

   ```js
   const { data, pending, error, refresh } = await useAsyncData(
     "mountains",
     () => $fetch("https://api.nuxtjs.dev/mountains")
   );
   ```

   - useFetch：‌ 这是一个可组合函数，‌ 提供了一个方便的包装器，‌ 用于 useAsyncData 和$fetch。‌ 它自动生成基于 URL 和 fetch 选项的键，‌ 并根据服务器路由提供请求 URL 的类型提示，‌ 同时推断 API 响应类型。‌useFetch 可以理解为对 useAsyncData 的一层封装，‌ 提供了更简洁的开发者语法糖。‌

   ```js
   const param1 = ref("value1");

   const { data, pending, error, refresh } = await useFetch(
     "https://api.nuxtjs.dev/mountains",
     {
       query: { param1, param2: "value2" },
     }
   );
   ```

   - useLazyAsyncData 和 useLazyFetch：‌ 这两个方法与上述方法类似，‌ 但它们将配置选项中的 Lazy 设置成了 true。‌ 这意味着在使用 useLazyAsyncData 和 useLazyFetch 时，‌ 在客户端渲染时不会阻塞导航的跳转。‌ 你需要处理数据为 null 的情况，‌ 并为 data 设置一个默认值。‌ 在服务端渲染时，‌ 效果与 useAsyncData 和 useFetch 相同，‌ 会阻塞页面渲染。‌

   > 这些方法提供了灵活的选项来处理异步数据获取，‌ 适应不同的应用场景和需求。‌ 使用这些方法，‌ 开发者可以更高效地在 Nuxt3 应用程序中获取和处理后端数据

6. Nuxt3 生命周期钩子

   - 在 nuxt.config 时---

   ```js
   export default defineNuxtConfig({
     hooks: {
       close: () => {},
     },
   });
   ```

   - 在 Nuxt 模块中

   ```js
   import { defineNuxtModule } from "@nuxt/kit";
   export default defineNuxtModule({
     setup(options, nuxt) {
       nuxt.hook("close", async () => {});
     },
   });
   ```

   - 应用程序钩子（运行时）

   ```js
   export default defineNuxtPlugin((nuxtApp) => {
     nuxtApp.hook("page:start", () => {
       /* 在这里写入你的代码 */
     });
   });
   ```

   - 服务程序钩子

   ```js
   export default defineNitroPlugin((nitroApp) => {
     nitroApp.hooks.hook("render:html", (html, { event }) => {
       console.log("render:html", html);
       html.bodyAppend.push("<hr>由自定义插件追加的内容");
     });
     nitroApp.hooks.hook("render:response", (response, { event }) => {
       console.log("render:response", response);
     });
   });
   ```

   - 附加钩子

   ```js
   import { HookResult } from "@nuxt/schema";

   declare module '#app' {
       interface RuntimeNuxtHooks {
           'your-nuxt-runtime-hook': () => HookResult
       }
       interface NuxtHooks {
           'your-nuxt-hook': () => HookResult
       }
   }

   declare module 'nitropack' {
       interface NitroRuntimeHooks {
           'your-nitro-hook': () => void;
       }
   }
   ```

7. Nuxt3 状态管理

   > Nuxt 提供了强大的状态管理库和 userState 组合函数，用于创建响应式且适用于 SSR 的共享状态

   ***

   Nuxt 提供了 useState 组合函数，用于在组件之间创建响应式且适用于 SSR 的共享状态
   useState 是一个适用于 SSR 的 ref 替代品。他的只将在服务器端渲染后保留（在客户端渲染期间进行 hydration）,并通过唯一的键在所有组件之间共享

   > 由于 useState 内部的数据将被序列化为 JSON，因此重要的是它不包含无法序列化的内容，如类，函数或符号

- 使用 shallowRef
  > 如果你不需要你的状态具有深层响应性，你可以将 useState 与 shallowRef 结合使用。当状态包含大型对象和数组时，这可以提高性能。

```js
const state = useState("my-shallow-state", () =>
  shallowRef({ deep: "not reactive" })
);
// isShallow(state) === true
```

- 基本用法

```js
<script setup lang="ts">
const counter = useState('counter', () => Math.round(Math.random() * 1000))
</script>

<template>
<div>
  计数器：{{ counter }}
  <button @click="counter++">
    +
  </button>
  <button @click="counter--">
    -
  </button>
</div>
</template>

```
