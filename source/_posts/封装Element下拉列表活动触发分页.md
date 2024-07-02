---
title: Element下拉列表滚动触发分页
tags: Eric的博客
date: 2023-5-17
author: Eric
comments: false
cover: /img/7.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true 
position: both #封面显示的位置# 三个值可配置left , right , both 
---

# 需求分析

> 因为公司整体 form 是单独封装的，所以关于 select 下拉的修改最后还要整合到 form 组件中，所以使用指令的解决办法

# 实现

> 实现过程中了解到 vue3 跟 vue2 的指令声明周期函数有所不同，所以单独了解下 官方文档：[vue3.0](https://cn.vuejs.org/guide/reusability/custom-directives.html#directive-hook), [vue2.0](https://v2.cn.vuejs.org/v2/guide/custom-directive.html#%E9%92%A9%E5%AD%90%E5%87%BD%E6%95%B0)

***vue3.0***

```js
// vue3的钩子函数
const myDirective = {
 // 在绑定元素的 attribute 前
 // 或事件监听器应用前调用
 created(el, binding, vnode, prevVnode) {
  // 下面会介绍各个参数的细节
 },
 // 在元素被插入到 DOM 前调用
 beforeMount(el, binding, vnode, prevVnode) {},
 // 在绑定元素的父组件
 // 及他自己的所有子节点都挂载完成后调用
 mounted(el, binding, vnode, prevVnode) {},
 // 绑定元素的父组件更新前调用
 beforeUpdate(el, binding, vnode, prevVnode) {},
 // 在绑定元素的父组件
 // 及他自己的所有子节点都更新后调用
 updated(el, binding, vnode, prevVnode) {},
 // 绑定元素的父组件卸载前调用
 beforeUnmount(el, binding, vnode, prevVnode) {},
 // 绑定元素的父组件卸载后调用
 unmounted(el, binding, vnode, prevVnode) {},
};
```

***vue2.0***

```js
// vue2的钩子函数
const myDirective = {
 bind(el, binding, vnode, oldVnode) {}, //只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。

 inserted(el, binding, vnode, oldVnode) {}, //被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。

 update(el, binding, vnode, oldVnode) {}, //所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。

 componentUpdated(el, binding, vnode, oldVnode) {}, //指令所在组件的 VNode 及其子 VNode 全部更新后调用。

 unbind(el, binding, vnode, oldVnode) {}, //只调用一次，指令与元素解绑时调用。
};
```

***代码***

```javaScript
// 使用
import { uniqBy } from "lodash-es"
import {
 deepClone,
} from '@/common/utils/index';//深拷贝方法。
import { debounce } from 'throttle-debounce';//限制回掉函数的执行频率，但是不同于 throttle 的是，debounce 能保证在一系列调用的时间内，回调函数只执行一次

export default {
 mounted(el, binding) {
  console.log('binding', binding);
  // 如果有method由调用方实现，没有则在这里实现加载和远程搜索的功能
  if (binding.modifiers.method) {
   // 节流
   let timer;
   // 滚动监听
   el.querySelector('.el-select-dropdown .el-select-dropdown__wrap').addEventListener(
    'scroll',
    function () {
     const condition = this.scrollHeight - this.scrollTop <= this.clientHeight + 100;
     if (!timer && condition) {
      // 滚动加载（调用自定义的加载方法）
      binding.value();
      timer = setTimeout(() => {
       clearTimeout(timer);
       timer = null;
      }, 500);
     }
    }
   );
  } else {
   // 实现
   // 传入的对象
   let value = binding.value;
   // 节流
   let timer;
   // 无搜索内容变量
   let pageNum = 1;
   let pages = 1;
   // 远程搜索内容变量
   let searchPageNo = 1;
   let searchPages = 1;
   // 每次加载的条数
   let pageSize = isNaN(value.pageSize) ? 10 : parseInt(value.pageSize);

   console.log('value.modelOptions', value.modelOptions);
   let modelOptions = value?.modelOptions;
   // 唯一的字段名 用于过滤数组中重复值
   let onlyKey = value?.onlyKey ?? value.modelField;

   // 远程搜索字段
   let searchField = value.searchField;
   // TODO 接口名称 使用时补充
   let methodName = value.methodName;
   const methodNameData = {
    // 接口名对应的方法
    // methodName
   };
   const searchMethodName = methodNameData[methodName];
   // 下拉数组，这个options在本方法中必须永远指向value.options，否则整个功能都将失效
   let options = value.options;
   // 无搜索拷贝数组，此处是为了在加载的基础上加一些默认的下拉项
   let optionsCopy = deepClone(value.options);
   // 远程搜索拷贝数组
   let optionsSearch = [];
   // 远程搜索内容
   let searchValue = '';
   // 加载逻辑
   const loadOptions = (searchField, search) => {
    let params = {
     pageSize: pageSize,
     ...value.otherParams,
    };
    console.log('value?.otherParams, ', value.otherParams);
    console.log('params, ', params);
    if (value.otherParams) {
     const isOtherParamsHave = Object.values(value.otherParams).every(
      item => item !== ''
     );
     if (params?.mustHaveValue && !isOtherParamsHave) {
      return;
     }
    }
    // ! ？？ 为什么要置空
    // 这里不能改变options的指向，否则会使整个功能失效（不能用options = []）
    // options.length = 0

    // 判断是否为远程搜索，true-是
    if (searchField && search) {
     // 当到最大页数时不再查询
     if (searchPages >= searchPageNo) {
      params.pageNum = searchPageNo++;
      params[searchField] = search;
      // todo 请求方法是我们自己封装的，需要换成自己项目的方法
      searchMethodName(params).then(res => {
       if (res) {
        searchPages = Math.ceil(
         (res?.data?.total ?? res?.total ?? 0) / pageSize
        );
        const resData = res?.data?.data ?? res?.data ?? [];
        optionsSearch = optionsSearch.concat(resData);
        dataProcessing(optionsSearch);
       }
      });
     }
    } else {
     // 当到最大页数时不再查询
     console.log('pageNum1111111111', pageNum, pages);
     if (pages >= pageNum) {
      params.pageNum = pageNum++;
      console.log('', params);
      searchMethodName(params).then(res => {
       if (res) {
        // debugger
        pages = Math.ceil((res?.data?.total ?? res?.total ?? 0) / pageSize);
        const resData = res?.data?.data ?? res?.data ?? [];
        optionsCopy = optionsCopy.concat(resData);
        console.log('optionsCopy', optionsCopy);
        console.log('resData', resData);
        dataProcessing(optionsCopy);
        console.log('value', value);
        console.log('options', options);
        const map = {};
        // 1、把数组元素作为对象的键存起来（这样就算有重复的元素，也会相互替换掉）
        // options.forEach(item => map[JSON.stringify(item)] = item);

        // 2、再把对象的值抽成一个数组返回即为不重复的集合
        // Object.keys(map).map(key => console.log(map[key]))
        // options.length == 0;
        // newOptions.forEach(item => {
        //  options.push(item)
        // })
       }
      });
     } else {
      dataProcessing(optionsCopy);
     }
    }
   };

   // 返回数据处理
   let dataProcessing = (optionsCopy, isBackToShow = false) => {
    // 这里不能改变options的指向，否则会使整个功能失效
    const optionsMap = new Map();
    console.log('copy的options', optionsCopy, options);
    // 为optionsMap 设置键，值
    options.forEach(item => optionsMap.set(item[onlyKey], 1));
    optionsCopy.forEach(item => {
     let itemData = {};
    // 我们自己内部的判断/如果你不需要不用加这些判断直接走eles
    // 主要为了前端展示不同的显示有需求要拼接的话 需要单独判断
     if (methodName == 'getExpenseSmallList') {
        itemData = {
          value: item[value.valueName],
          label: item.costCode + '  ' + item[value.labelName],
          labelOtherName:
          item.parentCostCode + '  ' + item.costCode + '  ' + item.costName,
          ...item,
        };
     } else if (methodName == 'getMoneyPageList') {
        itemData = {
          value: item[value.valueName],
          label: item.moneyModelCode + '  ' + item[value.labelName],
          ...item,
        };
     } else if (methodName == 'getBoxListPage') {
        itemData = {
          value: item[value.valueName],
          label: item.containerTypeCode + '-' + item[value.labelName],
          ...item,
        };
     } else if (value?.showValue) {
        itemData = {
          value: item[value.valueName],
          label: item[value.valueName],
          labelOtherName: item[value.labelName],
          ...item,
        };
     } else {
        itemData = {
          value: item[value.valueName],
          label: item[value.labelName],
          ...item,
        };
     }
    //  判断某个键是否存在与当前Map对象中 返回布尔值
     const hasItemVal = optionsMap.has(item[onlyKey]);
     if (isBackToShow) {
        let check = options.find(t => {
         return t[value.modelField] === item[value.modelField];
        });
        if (!check && !hasItemVal) {
         options.push(itemData);
         optionsMap.set(item[onlyKey]);
        }
     } else {
        options.push(itemData)
        !hasItemVal && options.push(itemData) && optionsMap.set(item[onlyKey]);
     }
    });
    // _.uniqBy([array], [iteratee=_.identity])
    // 参数：该方法接受上述和以下所述的两个参数：
    // [array]：此参数保存要检查的阵列。
    // [iteratee = _。identity](函数)：此参数保存每个元素调用的iteratee。
    // 去重 第一个参数为数组 第二个参数为执行的迭代函数 条件/去重的字段
    const uniqByOptions = uniqBy(options, function (element) {
     return element.value
    })
    options.splice(0);
    uniqByOptions.forEach(item => {
     options.push(item);
    })
    console.log('uniqByOptions', uniqByOptions)
   };

   // 判断是否需要回显
   if (value.model && value.modelField) {
    options.splice(0);
    options.length = 0;
    optionsCopy.length = 0;
    if (modelOptions) {
     console.log(value.otherLabel);
     if (methodName == 'getExpenseSmallList') {
      modelOptions.value = modelOptions[value.valueName];
      modelOptions.label =
       modelOptions[value.valueName] + ' ' + modelOptions[value.labelName];
      modelOptions.labelOtherName = value.otherLabel + ' ' + modelOptions.label;
      options.push(modelOptions);
     } else if (value?.showValue) {
      modelOptions.value = modelOptions[value.valueName];
      modelOptions.label = modelOptions[value.valueName];
      modelOptions.labelOtherName = item[value.labelName];
      options.push(modelOptions);
     } else {
      modelOptions.value = modelOptions[value.valueName];
      modelOptions.label = modelOptions[value.labelName];
      options.push(modelOptions);
     }
    }
    // 回显方法
    /* 
    let echo = (model, modelField) => {
      let params = {
        ...value.otherParams,
        pageSize: 1,
        pageNum: 1
      }
      params[modelField] = model;
      searchMethodName(params).then(res => {
        if (res) {
          const resData = res?.data?.data ?? res?.data ?? [];
          optionsCopy = optionsCopy.concat(resData)
          dataProcessing(optionsCopy, true)
        }
      })
    }   
    if (optionsCopy.length > 0) {
      let check = optionsCopy.find((item) => {
        return item[value.modelField] === value.model
      })
      if (!check) {
        echo(value.model, value.modelField)
      }
    } else {
       echo(value.model, value.modelField)
    }*/
   } else {
    // 首次加载
    options.splice(0);
    options.length = 0;
    optionsCopy.length = 0;
    console.log(';加载首次', options);
    loadOptions();
   }
   let SELECTWRAP_DOM = null;
   // 滚动监听（无限滚动）
   if (value?.popperClass) {
    SELECTWRAP_DOM = document.querySelector(
     `.${value.popperClass} .el-select-dropdown .el-select-dropdown__wrap`
    );
   }
   if (!SELECTWRAP_DOM) {
    return;
   }
   SELECTWRAP_DOM.addEventListener('scroll', function () {
    console.log('this.scrollHeight', this.scrollHeight);
    const condition = this.scrollHeight - this.scrollTop <= this.clientHeight + 70;
    if (!timer && condition && options.length > 1) {
     // 滚动加载 非回显时 可滚动
     if (!(value.model && value.modelField)) {
      loadOptions(searchField, searchValue);
     }
     timer = setTimeout(() => {
      clearTimeout(timer);
      timer = null;
     }, 600);
    }
   });
   // 输入搜索
   function getSearchList() {
    console.log('this.value', this.value);
    if (this.value) {
     searchPageNo = 1;
     searchPages = 1;
     optionsSearch = [];
     searchValue = this.value;
     optionsCopy.length = 0;
     loadOptions(searchField, searchValue);
    } else {
     searchValue = '';
     dataProcessing(optionsCopy);
    }
   }
   // 清空所选值时的操作
   /*
      清空搜索值
      清空数组
      重新请求
   */
   function getChangeList() {
    console.log('getChangeList');
    value.model = '';
    searchPageNo = 1;
    searchPages = 1;
    optionsSearch = [];
    searchValue = '';
    options.splice(0);
    options.length = 0;
    optionsCopy.length = 0;
    pageNum = 1
    loadOptions();
   }
   const debounceGetSearchList = debounce(600, getSearchList);
   const debounceGetChangeList = debounce(600, getChangeList);

   // 输入监听（远程搜索）
   if (searchField) {
    el.getElementsByTagName('input')[0].addEventListener(
     'input',
     debounceGetSearchList
    );
   }
   // 编辑下点击清空按钮
   // el-input__suffix 增加点击事件
   el.getElementsByClassName('el-input__suffix')[0].addEventListener(
    'click',
    () => {
     const isBackToShow = Object.values(modelOptions).every(item => !!item);
     const isClear = el.getElementsByClassName('el-icon')[0].style.display;
     // 回显情况下 点了清空按钮
     if (isClear && isBackToShow) {
      console.log('清空了');
      getChangeList();
     } else if (isClear && options.length === 1) {
      // 选中情况下清空 options长度只为1 需要重新请求列表
      console.log('选中后清空了');
      getChangeList();
     } else {
      console.log('选中后清空了');
      getChangeList();
     }
    },
    true
   );
  }
 },
};

```
