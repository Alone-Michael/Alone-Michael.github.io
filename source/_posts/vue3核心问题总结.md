---
title: vue3核心问题总结
tags:
  - 问题
  - vue3
  - vue2
date: 2024-6-19
author: Eric
comments: false
cover: /img/23.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

### 前言

> 使用 vue3 中总结的一些问题

### 1. Vue3 相比 Vue2 有那些主要改进？

> - 使用了 Composition API，提高了代码逻辑的可复用性。
> - 引入了 Fragment，允许组件有多个根节点。
> - 更好的 TypeScript 支持。
> - 使用了更小的包体积和更高效的运行时性能。
> - 提供了 Suspense 组件，用于处理异步组件的加载状态。
> - Teleport 组件允许将子组件渲染到 DOM 中的任何位置。

### 2. Vue 3 的 Composition API 是什么？

> Composition API 是一组新的、基于函数的 API，它允许你以更灵活和可复用的方式组织组件逻辑。它主要包括 ref、reactive、computed、watch、setup 等函数和钩子。

### 3. Vue 3 的 Composition API 是什么？

> setup() 是 Vue 3 组件选项 API 中的一个新选项。它是 Composition API 的入口点，在组件被创建之前执行，用于初始化状态、计算属性和方法，并返回在模板中使用的响应式引用。

### 4. 请解释 ref 和 reactive 的区别？？

> - ref 用于创建简单的响应式引用，通常用于基本数据类型。
> - reactive 用于创建响应式对象，通常用于复杂数据类型如数组和对象。

### 5. Vue 3 中的 watch 和 watchEffect 有何不同？

> - watch 允许你监听特定的响应式引用或计算属性，并在它们改变时执行回调函数。
> - watchEffect 会立即执行一个函数，并自动追踪其依赖的响应式引用，当依赖改变时重新执行。

### 6. Vue 3 中的 Suspense 组件是如何工作的？

> Suspense 组件允许你指定一个加载中的状态（fallback）和一个加载失败的状态（fallback slot），用于处理异步组件的加载状态。当异步组件加载时，会先显示加载中的状态；加载完成后，会显示异步组件；如果加载失败，会显示加载失败的状态。

### 7. Vue 3 中的 Teleport 组件有什么作用？

> Teleport 组件允许你将子组件渲染到 DOM 树中的任何位置，而不仅仅是其父组件的模板内。这在处理模态框、下拉菜单等需要渲染到特定位置的组件时非常有用。

### 8. Vue 3 如何优化性能？

> - 使用 v-show 代替 v-if 来频繁切换元素，因为 v-show 只是切换元素的 CSS 属性。
> - 使用 key 来优化列表渲染的性能。
> - 使用 computed 和 watch 来减少不必要的计算和渲染。
> - 使用 v-memo 来缓存组件的渲染结果。

### 9. Vue 3 的 provide 和 inject 是如何工作的？

> provide 和 inject 允许你在祖先组件和后代组件之间传递数据，而不需要通过 props 和 events 进行逐层传递。provide 在祖先组件中定义要传递的数据，inject 在后代组件中接收数据。

### 10. Vue 3 如何处理组件的异步加载？

> Vue 3 使用 defineAsyncComponent 函数来定义异步组件。异步组件允许你延迟加载组件，直到需要时才加载，从而优化性能。

### 11. Vue 3 中的响应式系统是如何工作的？

> Vue 3 的响应式系统通过 ES6 的 Proxy 对象来实现。当使用 reactive 或 ref 函数时，Vue 会创建一个 Proxy 对象来包裹原始数据。这个 Proxy 对象会拦截对原始数据的访问和修改，从而可以追踪到数据的变化。当数据发生变化时，Vue 会触发相应的依赖更新，重新渲染视图。

### 12. Vue 3 中的自定义指令是如何定义的？

> Vue 3 中的自定义指令可以使用 app.directive()方法或组件的 directives 选项来定义。自定义指令包含一些钩子函数，如 bind、inserted、update、componentUpdated 和 unbind，这些钩子函数会在指令的不同生命周期阶段被调用。

### 13. Vue 3 中的路由和 Vue Router 4 有哪些变化？

> Vue Router 4 与 Vue 3 一起发布，并做了一些改进和优化。其中，最大的变化是支持 Vue 3 的 Composition API，使得路由逻辑更加可复用和灵活。此外，Vue Router 4 还引入了一些新的功能和 API，如滚动行为、路由守卫的改进等。

### 14. Vue 3 如何与 Vuex 4 一起使用？

> Vuex 4 是 Vue 3 的官方状态管理库。在 Vue 3 中，你可以使用 Vuex 4 来管理全局状态。你需要创建一个 Vuex store，并在 Vue 应用中使用 createStore 函数将其与 Vue 实例关联起来。然后，你可以在组件中使用 computed 或 mapState 等辅助函数来访问和修改 Vuex 中的状态。

### 15. Vue 3 的插槽（Slots）和 Vue 2 有何不同？

> Vue 3 的插槽（Slots）与 Vue 2 相比有一些改进和变化。首先，Vue 3 支持具名插槽和默认插槽的混合使用，这使得插槽的使用更加灵活。其次，Vue 3 引入了作用域插槽（Scoped Slots），允许你在插槽中访问子组件的数据和方法。此外，Vue 3 还改进了插槽的渲染和更新性能。

### 16. Vue 3 中的过渡和动画效果是如何实现的？

> Vue 3 中的过渡和动画效果可以通过 CSS 或 JavaScript 来实现。Vue 提供了<transition>和<transition-group>组件来封装要过渡的元素，并通过 CSS 类名来控制过渡效果。同时，Vue 还支持 JavaScript 钩子函数来在过渡的不同阶段执行自定义逻辑。

### 17. Vue 3 的服务器端渲染（SSR）是如何工作的？

> Vue 3 的服务器端渲染（SSR）是通过将 Vue 组件渲染为服务器端的 HTML 字符串来实现的。在服务器端，你可以使用 Vue 的 SSR 库（如 Nuxt.js）来构建 Vue 应用，并将组件渲染为 HTML 字符串发送给客户端。客户端在接收到 HTML 字符串后，可以直接将其插入 DOM 中，从而避免了在客户端进行首次渲染的开销。

### 18. Vue 3 中的虚拟 DOM 是如何工作的？

> Vue 3 中的虚拟 DOM（Virtual DOM）是一个编程概念，用于在内存中模拟 DOM 树的结构和状态。当 Vue 组件的状态发生变化时，Vue 会创建一个新的虚拟 DOM 树，并与旧的虚拟 DOM 树进行比较。通过比较两个虚拟 DOM 树的差异，Vue 可以计算出最小的 DOM 更新操作，并将这些操作应用到实际的 DOM 树上，从而提高了页面的渲染性能。

### 19. Vue 3 如何处理全局状态？

> 在 Vue 3 中，你可以使用 Vuex 4 来处理全局状态。Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式和库。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。此外，你还可以使用 Vue 3 的 provide 和 inject API 在祖先组件和后代组件之间传递全局状态。

### 20. Vue 3 的生命周期钩子有哪些变化？

> Vue 3 对生命周期钩子进行了一些调整和优化。首先，Vue 3 将 beforeDestroy 和 destroyed 钩子重命名为 beforeUnmount 和 unmounted，以更好地反映组件的卸载过程。其次，Vue 3 新增了 setup 钩子函数，用于在组件创建之前执行初始化逻辑。此外，Vue 3 还提供了 onBeforeMount、onMounted、onBeforeUpdate、onUpdated、onBeforeUnmount 和 onUnmounted 等生命周期钩子函数，用于在组件的不同生命周期阶段执行自定义逻辑。

### 21. Vue 3 如何与 TypeScript 一起使用？

> Vue 3 内部已经使用了 TypeScript 进行编写，因此与 TypeScript 的集成非常顺畅。你可以直接在 Vue 组件中使用 TypeScript 的类型声明和接口，以提供更严格的类型检查和更好的代码提示。此外，Vue 3 的官方文档也提供了 TypeScript 的指南和示例，帮助你更好地使用 TypeScript 来编写 Vue 应用。

### 22. Vue 3 中的 Teleport 是什么？

> Teleport 是 Vue 3 中引入的一个新特性，它允许你将组件的 DOM 结构渲染到 DOM 树中的任何位置，而不仅仅是组件的根元素内部。这使得你可以更灵活地控制组件的渲染位置，实现一些复杂的布局和交互效果。例如，你可以将模态框（Modal）的 DOM 结构渲染到<body>元素的末尾，以确保其始终位于页面的最顶层。

### 23. Vue 3 中的 Suspense 是什么？

> Suspense 是 Vue 3 中引入的另一个新特性，用于处理异步组件的加载过程。通过在 Suspense 组件中包裹异步组件，并提供一个 fallback 选项，你可以在异步组件加载过程中展示一个占位符或加载指示器，直到异步组件加载完成并成功渲染。这使得你可以更优雅地处理异步组件的加载和错误情况，提升用户体验。

### 24. Vue 3 中的 Composition API 是什么？

> Composition API 是 Vue 3 中引入的一套新的 API，用于组织和重用 Vue 组件的逻辑。与 Vue 2 中的 Options API 相比，Composition API 提供了更灵活、更可复用的代码组织方式。它允许你将组件的逻辑拆分成多个独立的函数（称为“组合函数”或“composables”），每个函数都负责处理特定的功能或状态。然后，你可以将这些组合函数组合起来，形成一个完整的 Vue 组件。这种方式使得代码更易于测试、维护和复用，特别适用于大型和复杂的 Vue 应用。

### 25. Vue 3 的响应式系统是如何追踪数据变化的？

> Vue 3 的响应式系统使用了 ES6 的 Proxy 对象来追踪数据的变化。当数据对象被 Proxy 包裹后，任何对数据的访问或修改都会触发相应的 getter 或 setter 函数，从而可以追踪到数据的变化。当数据变化时，Vue 会触发依赖的更新，重新渲染视图。

### 26. Vue 3 的事件修饰符有哪些？

Vue 3 的事件修饰符与 Vue 2 保持一致，主要包括：

> - .stop：阻止事件冒泡
> - .prevent：阻止默认事件行为
> - .capture：使用事件捕获模式
> - .self：只有当事件在元素本身（而不是子元素）触发时触发回调
> - .once：事件只触发一次
> - .left：只当鼠标左键被点击时触发
> - .right：只当鼠标右键被点击时触发
> - .middle：只当鼠标中键被点击时触发
> - .passive：以 { passive: true } 模式调用 eventListener

### 27. Vue 3 中的 Fragment 是什么？

> 在 Vue 3 中，Fragment 允许组件有多个根节点。在 Vue 2 中，组件只能有一个根节点，但在 Vue 3 中，组件可以有多个根节点，这使得组件的结构更加灵活。Fragment 在渲染时会被视为一个虚拟节点，并不会实际创建额外的 DOM 元素。

### 28. Vue 3 如何实现组件的懒加载？

> Vue 3 使用动态导入（import()）语法来实现组件的懒加载。动态导入允许你按需加载 JavaScript 模块，从而实现组件的延迟加载。在 Vue 中，你可以使用 defineAsyncComponent 函数来定义异步组件，该函数接受一个返回 Promise 的函数作为参数，该 Promise 在解析时应返回一个组件定义。

### 29. Vue 3 中的自定义指令如何定义和使用？

> 在 Vue 3 中，你可以使用 app.directive() 方法来全局注册自定义指令，或者使用组件选项 directives 来局部注册。自定义指令的定义包括一个名字、一个或多个钩子函数（如 bind、inserted、update、componentUpdated 和 unbind）以及一些可选的选项。在模板中，你可以使用 v- 前缀加上指令名来使用自定义指令

### 30. Vue 3 如何与 Web Components 集成？

> Vue 3 提供了对 Web Components 的原生支持，允许你在 Vue 应用中直接使用 Web Components。你可以将 Web Components 作为自定义元素在 Vue 模板中使用，也可以使用 Vue 的 h 函数来动态创建和渲染 Web Components。此外，Vue 3 还提供了一些 API 来与 Web Components 进行交互，如 ref、slots 和 events 等。
