---
title: Vue3.0学习笔记（持续更新）
tags: Eric的博客
author: Eric
---

### Vue 之 getCurrentInstance

1. 概述：该方法可以获取当前组件的实例、上下文来操作router和vuex等

2. 使用：由vue 提供，按需引入

```js
import { getCurrentInstance } from 'vue'
```

```js
import { getCurrentInstance } from 'vue';
// 获取当前组件实例
const instance = getCurrentInstance();

// 获取当前组件的上下文，下面两种方式都能获取到组件的上下文。
const { ctx }  = getCurrentInstance();  //  方式一，这种方式只能在开发环境下使用，生产环境下的ctx将访问不到
const { proxy }  = getCurrentInstance();  //  方式二，此方法在开发环境以及生产环境下都能放到组件上下文对象（推荐）
// ctx 中包含了组件中由ref和reactive创建的响应式数据对象,以及以下对象及方法;
proxy.$attrs
proxy.$data
proxy.$el
proxy.$emit
proxy.$forceUpdate
proxy.$nextTick
proxy.$options
proxy.$parent
proxy.$props
proxy.$refs
proxy.$root
proxy.$slots
proxy.$watch

```

### Vue 之 ref与shallowRef、reactive与shallowReactive、triggerRef

1. shallowRef与reactive

- 使用reactive生命的变量为递归监听，使用shallowReactive声明的变量为非递归监听（通俗的讲就是reactive创建的对象会被vue内部进行递归监听，可以监听到对象中的对象是否发生了改变从而改变视图，而shallowReactive创建的对象只能监听到首层对象的变化）简而言之，shallowReactive声明的变量不具有响应性，而reactiv而具有

```javascript
<template>
  <!-- reactive -->
  <div>{{ reactiveState }}</div>
  <button @click="reactiveStateChange">改变reactiveState.b.c</button>

  <!-- shallowReactive -->
  <div>{{ shallowReactiveState }}</div>
  <button @click="shallowReactiveChange">改变shallowReactiveState.b.c</button>
</template>

<script>

import { shallowReactive, reactive } from 'vue'

export default {
  name: 'App',
  setup() {
    const reactiveState = reactive({
      a: 'a',
      b: {
        c: 'c',
        d: {
          e: 'e',
        },
      },
    })
    function reactiveStateChange() {
      reactiveState.b.c = 99999
      console.log(reactiveState, 'reactiveState发生了变化且视图更新了')
    }
    const shallowReactiveState = shallowReactive({
      a: 'a',
      b: {
        c: 'c',
        d: {
          e: 'e',
        },
      },
    })
    function shallowReactiveChange() {
      shallowReactiveState.b.c = 99999
      console.log(shallowReactiveState, 'shallowReactiveState发生了变化但视图未更新')
      // 打开下方这行注释视图将进行更新（因为首层对象发生了改变）
      // shallowReactiveState.a = 99999
    }
    return {
      reactiveState,
      reactiveStateChange,
      shallowReactiveState,
      shallowReactiveChange,
    }
  },
}
</script>
```

2. shallowRef对比ref

- ref底层相当于多用了一次reactive,并且使用value包裹了创建的变量,所以在使用过程中用ref声明的变量我们需要通过var.value的方式去使用而reactive 声明的不需要

```js
const status = ref(false)
// 等价于
cosnt status = reactive({
  value: false
})
```

- 其中shallowRef为非递归监听，ref为递归监听，与shallowReactive和reactive不同的是shallowRef和ref监听的对象首层是value这一层，只有当value发生变化时shallowRef声明的变量才会在视图上进行更新

```js
<template>
  <div>{{ shallowRefState }}</div>
  <button @click="shallowRefChange">改变state.b.c</button>
</template>

<script>
import { shallowRef, ref, triggerRef } from 'vue'
export default {
  name: 'App',
  setup() {
    const shallowRefState = shallowRef({
      a: 'a',
      b: {
        c: 'c',
        d: {
          e: 'e',
        },
      },
    })
    function shallowRefChange() {
      shallowRefState.value.b.c = 99999
      console.log(shallowRefState, 'shallowRefChange发生了变化但视图未更新')
      // 打开下方代码视图才会进行更新（原因是shallowRefState首层value发生了改变）
      // shallowRefState.value = {
      //   a: 'a',
      //   b: {
      //     c: '变化的数据',
      //     d: {
      //       e: 'e',
      //     },
      //   },
      // }
    }
    return {
      shallowRefState,
      shallowRefChange,
    }
  },
}
</script>

```

3.triggerRef的作用

- 上面第二点我们知道了只有当value发生改变时shallowRef声明的变量才会在视图上进行更新，而triggerRef的作用则是手动执行与shallowRef关联的任何副作用，强制更新视图  

```js
<template>
  <div>{{ shallowRefState }}</div>
  <button @click="shallowRefChange">改变state.b.c</button>
</template>

<script>
import { shallowRef, ref, triggerRef } from 'vue'
export default {
  name: 'App',
  setup() {
    const shallowRefState = shallowRef({
      a: 'a',
      b: {
        c: 'c',
        d: {
          e: 'e',
        },
      },
    })
    function shallowRefChange() {
      shallowRefState.value.b.c = 99999
      // 强制更新视图
      triggerRef(shallowRefState)
    }
    return {
      shallowRefState,
      shallowRefChange,
    }
  },
}
</script>

```

4. 没有triggerReactive

### Vue3.0之 使用provide/inject 代替vuex的实现

1. 在vue2中我们在实现全局状态管理以及部分全局变量的使用中会使用vuex 或者bus中央事件总线来解决或处理类似的问题，但是在使用vuex中我们往往会将某一业务实现复杂化。
2. 概述：在vue3中我们将使用vue自带的provide和inject来实现全局状态管理
3. 实现思路：首先我们在src下创建一个store的模块，来管理全局的状态、api的调用、表单模块配置；然后将全局的状态进行合并并导出；再再App.vue下使用provide来全局注入；最后在各个组件中使用inject来调用对应的状态即可。
4. 实现步骤一：目录结构

```js
src
--store
----modules
------user // 例如为用户的状态管理
--------api 
----------index.js // 用户的api接口配置
--------index.js // 用户的状态管理配置
--------model.js // 用户的表单模块配置
----index.js // 合并所有状态
```

5. 实现步骤二：具体代码

```js
// src/store/index.js
import _ from 'lodash';

const contexts = require.context('./modules', true);
let modules = {};
contexts.keys().forEach(key => {
  if (_.includes(key, 'index.js') && !_.includes(key, 'api')) {
    const module = contexts(key).default;
    Object.assign(modules, module);
  }
});
export default modules;
```

```js
// src/store/modules/api/index.js
// 主要是存放对应模块的api接口配置
```

```js
// src/store/modules/user/index.js
import { ref } from 'vue';
import api from './api';

const namespace = 'user';
const r = ac => `store/${namespace}/${ac}`;

export const GETTER_USER = r('GETTER_USER');
export const GET_USER = r('GET_USER');
export const CHANGE_USER = r('CHANGE_USER');

const storeModel = api => {
  const user = ref({});

  const getUser = () => {
    setTimeout(() => {
      user.value = {
        name: 'zhang_san',
        age: 18,
        address: '成都市',
        phone: '18888888888'
      };
    }, 1000);
  };

  const changeUser = name => {
    user.value.name = name;
  };

  return {
    [GETTER_USER]: user,
    [GET_USER]: getUser,
    [CHANGE_USER]: changeUser
  };
};

export default storeModel(api);
```

```js
// src/store/modules/user/model.js
// 主要是存放对应模块的表单模板配置
```

6. 实现步骤三：App.vue

```js
import store from '@/store';
import _ from 'lodash';

// ...
setup() {
    _.each(store, (item, index) => {
      provide(index, item);
    });
}
// ...
```

7. 实现步骤四：组件中使用

```js
<template>
  <div class="first-children-comp">
    <p @click="getUser">获取用户信息</p>
    <p @click="changeUser">修改用户信息</p>
    <span>用户信息：{{user.name}}</span>
  </div>
</template>

import { GETTER_USER, GET_USER, CHANGE_USER } from 'store/modules/user';

// ...
setup(props, context) {
    const user = inject(GETTER_USER);
    const storeGetUSer = inject(GET_USER);
    const storeChangeUser = inject(CHANGE_USER);
    // 获取用户
    const getUser = () => {
      storeGetUSer();
    };
    // 修改用户
    const changeUser = () => {
      storeChangeUser('li_si');
    };
    return { getUser, changeUser };
}
// ...
```

### Vue3之 watch  watchEffect 监听事件

1. 概述：vue2中的响应式变量监听，在vue3中以方法的形式使用
2. 使用：由vue提供，按需引入：

```js
import { watch } from 'vue'
```

3. 监听单一数据

```js
 import { ref, reactive, computed, watch } from 'vue';
setup(props) {
    // ref
    const age = ref(20);
    watch(() => age.value, (nv, ov) => { ... });
    
   // reactive
   const product = reactive({ name: '饮料', count: 1 });
   watch(() => product.count, (nv, ov) => { ... }); 
   
   // props
   watch(() => props.msg, (nv, ov) => { ... }); 
   
   // computed
   const userAge = computed(() => `今年${age.value}岁了！`);
   watch(() => userAge.value, (nv, ov) => { ... }); 
}
```

4. 监听对象

```js
import { ref, reactive, watch } from 'vue';
setup(props) {
    // ref
    const user = ref({ name: 'zhang_san', age: 20 });
    // 字面量引发的监听触发: user.value = { ... };
    watch(() => user.value, (nv, ov) => { ... });
    // 如果使用 user.value.age = 30这种方式去修改user的age值; 将不会触发上面的监听，需要使用watch的第三个参数（深度监听），且触发监听后的nv===ov true
    watch(() => user.value, (nv, ov) => { ... }, { deep: true });
    // 如果我们只需要监听name的值，那么
    watch(() => user.value.name, (nv, ov) => { ... });
    
   // reactive
   const reactiveData = reactive({ user: { name: 'zhang_san', age: 20 } });
    // 字面量引发的监听触发: reactiveData.user = { ... };
    watch(() => reactiveData.user, (nv, ov) => { ... });
    // 如果使用 user.user.age = 30这种方式去修改user的age值，将不会触发监听，需要使用watch的第三个参数（深度监听），且触发监听后的nv===ov true
    watch(() => reactiveData.user, (nv, ov) => { ... }, { deep: true });
    // 如果我们只需要监听name的值，那么
    watch(() => reactiveData.user.name, (nv, ov) => { ... });
}
```

5. 监听数组

```js
import { ref, reactive, watch } from 'vue';
setup(props) {
   // ref
   const user = ref([
      { name: 'zhang_san', age: 10 },
      { name: 'li_si', age: 10 }
   ]);
   // 字面量引发的监听触发: user.value = [ ... ];
   watch(() => user.value, (nv, ov) => { ... });
  // 如果使用数组的操作方法（如：push()）或者user.value[0].age = 20这类操作去修改数组某项的属性值，将不会触发监听，也需要使用深度监听模式，且触发监听后的nv===ov true
  watch(() => user.value, (nv, ov) => { ... }, { deep: true });
  
  // reactive
  const reactiveData = reactive({
    user: [
      { name: 'zhang_san', age: 10 },
      { name: 'li_si', age: 10 }
   ]
  });
  // 字面量引发的监听触发: user.user = [ ... ];
  watch(() => reactiveData.user, (nv, ov) => { ... });
  // 如果使用数组的操作方法（如：push()）或者user.value[0].age = 20这类操作去修改数组某项的属性值，将不会触发监听，也需要使用深度监听模式，且触发监听后的nv===ov true
  watch(() => reactiveData.user, (nv, ov) => { ... }, { deep: true });
}
```

6. 监听多个数据

```js
import { ref, reactive, computed, watch } from 'vue';
setup(props) {
    const age = ref(20);
    const user = ref({ name: 'zhang_san', age: 20 });
    
    watch([() => age.value, () => user.name], ([newAge, newName], [oldAge, oldName]) => { ... });
}

```

7. 终止监听

```js
import { ref, watch } from 'vue';
const age = ref(20);
// watch监听会返回一个方法
const stop = watch(age, (nv, ov) => { ... });
// 当调用此方法后，该监听就会被移除
stop();
```

8. 清除watch中无效的异步任务

```js
<template>
  <div>
    <input type="text" v-model="keywords" />
  </div>
</template>

<script>
import { watch, ref, reactive } from "@vue/composition-api";
export default {
  setup() {
    const keywords = ref("");
    // 异步任务：打印用户输入的关键词
    const asyncPrint = val => {
      return setTimeout(() => {
        console.log(val);
      }, 1000);
    };

    watch(
      keywords,
      (keywords, prevKeywords, onCleanup) => {
        // 执行异步任务，并得到关闭异步任务的 timerId
        const timerId = asyncPrint(keywords);
        // 如果 watch 监听被重复执行了，则会先清除上次未完成的异步任务
        onCleanup(() => clearTimeout(timerId));
      },
      { lazy: true }
    );
    return {
      keywords
    };
  }
};
</script>
```

9. 深度监听（deep）、默认执行（immediate）
  
> deep: 深度监听需要便利被侦听对象中的所有嵌套的属性，当用于大型数据结构时，开销很淡，因此在必要时使用，避免影响性能
>
> immediate: 进入该页面首次就触发监听动作

10. watchEffect也是一个监听器，是一个副作用函数，他会监听引用数据类型的所有属性，不需要具体到某个属性，一旦运行就会立即监听，组件卸载的时候也会停止监听

> watchEffect 仅会在其同步执行期间，才追踪依赖。在使用异步回调时，只有在第一个 await 正常工作前访问到的属性才会被追踪

```js
<template>
  <div>
    <input type="text" v-model="obj.name"> 
  </div>
</template>

<script>
import { reactive, watchEffect } from 'vue';
export default {
  setup(){
    let obj = reactive({
      name:'zs'
    });
    watchEffect(() => {
      console.log('name:',obj.name)
    })

    return {
      obj
    }
  }
}
</script>
```

11. watch vs. watchEffect

> watch 和 watchEffect 都能响应式地执行有副作用的回调。它们之间的主要区别是追踪响应式依赖的方式

- watch 只追踪明确侦听的数据源。它不会追踪任何在回调中访问到的东西。另外，仅在数据源确实改变时才会触发回调。watch 会避免在发生副作用时追踪依赖，因此，我们能更加精确地控制回调函数的触发时机。
- watchEffect，则会在副作用发生期间追踪依赖。它会在同步执行过程中，自动追踪所有能访问到的响应式属性。这更方便，而且代码往往更简洁，但有时其响应性依赖关系会不那么明确。

12. 回调的触发时机

> 当你更改了响应式状态，他可能会同时触发Vue组件更新和侦听器回调
>
> 默认情况下，用户创建的侦听器回调，都会在Vue组件更新**之前**被调用。这意味着你在侦听器回调中访问的DOM将是被Vue更新之前的状态
>
> 如果想在侦听器回调中能访问被Vue更新**之后**的DOM，你需要指明``` flush:'post' ```选项：
>
```js
  watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
```

> 后置刷新的watchEffect()有个更方便的别名```watchPostEffect()```:
>
```js
import { watchPostEffect } from 'vue'

watchPostEffect(() => {
  /* 在 Vue 更新后执行 */
})
```