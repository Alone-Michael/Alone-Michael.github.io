---
title: Js实现扁平化数据结构和tree转换
tags: Eric的博客
date: 2023-10-7
author: Eric
comments: false
cover: /img/22.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true 
position: both #封面显示的位置# 三个值可配置left , right , both 
---

## 扁平化数组转tree

1. map对象实现
> 思路：先把数据转成Map去存储，然后再遍历的同时借助对象的引用，直接从Map找对应的数据做存储。
> 
> Object.prototype.hasOwnProperty: 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性，会忽略掉那些从原型链上继承到的属性。

```js
// map对象实现
function arrayToTree(items) {
    let res = [] // 存放结果集
    let map = {}

    // 先转成map存储
    for (const i of items) {
        map[i.id] = { ...i, children: [] }
    }

    for (const i of items) {
        const newItem = map[i.id]
        if (i.pid === 0) {
            res.push(newItem)
        } else {
            if (Object.prototype.hasOwnProperty.call(map, i.pid)) {
                map[i.pid].children.push(newItem)
            }
        }
    }
    return res
}
```
2. 递归实现
> 最常用到的就是递归实现，思路也比较简单，实现一个方法，该方法传入tree父节点和父id，循环遍历数组，无脑查询，找到对应的子节点，push到父节点中，再递归查找子节点的子节点。
```js
function arrayToTree(items) {
  let res = []
  let getChildren = (res, pid) => {
      for (const i of items) {
          if (i.pid === pid) {
              const newItem = { ...i, children: [] }
              res.push(newItem)
              getChildren(newItem.children, newItem.id)
          }
      }
  }
  getChildren(res, 0)
  return res
}
```
3. 边做map存储，边找对应关系
> 思路：循环将该项的id为键，存储到map中，如果已经有该键值对了，则不用存储了，同时找该项的pid在不在map的键中，在直接对应父子关系，不在就在map中生成一个键值对，键为该pid，然后再对应父子关系。
```js
function arrayToTree(items) {
    let res = [] // 存放结果集
    let map = {}
    // 判断对象是否有某个属性
    let getHasOwnProperty = (obj, property) => Object.prototype.hasOwnProperty.call(obj, property)

    // 边做map存储，边找对应关系
    for (const i of items) {
        map[i.id] = {
            ...i,
            children: getHasOwnProperty(map, i.id) ? map[i.id].children : []
        }
        const newItem = map[i.id]
        if (i.pid === 0) {
            res.push(newItem)
        } else {
            if (!getHasOwnProperty(map, i.pid)) {
                map[i.pid] = {
                    children: []
                }
            }
            map[i.pid].children.push(newItem)
        }
    }
    return res
}
```

### tree扁平化

1. 递归实现
> 遍历tree，每一项加进结果集，如果有children且长度不为0，则递归遍历。
> 这里需要用解构赋值将每个节点的 children属性去除。
```js
function treeToArray(tree) {
  let res = []
  for (const item of tree) {
    const { children, ...i } = item
    if (children && children.length) {
      res = res.concat(treeToArray(children))
    }
    res.push(i)
  }
  return res
}
```

2. reduce实现
```js
function treeToArray(tree) {
  return tree.reduce((res, item) => {
    const { children, ...i } = item
    return res.concat(i, children && children.length ? treeToArray(children) : [])
  }, [])
}
```