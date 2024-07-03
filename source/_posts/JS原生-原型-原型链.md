---
title: JS原生-原型-原型链
tags:
  - js
date: 2023-6-30
author: Eric
comments: false
cover: https://pic.20988.xyz/2024-05-29/1716947219-264223-4.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

### 前言

原型和原型链基本上是基础前端面试必问的问题之一，虽然这是一个非常基础的知识点，但是往往工作了好几年的前端程序员都理不清楚。或许你大概知道在 Javascript 中有原型的概念，也知道有原型链的存在，但是如果面试官让你说出来，你可能会把自己都说蒙圈！

**总体原因：对原型和原型链理解得不够透彻！**

### 1.为什么要学会它们？

如果你学过后端语言比如 Java 等等，那么你应该知道它们都是面向对象的开发方式。面向对象有许多特点，其中继承就是其中一个。在 Java 中通常通过类 class 的方式来实现继承。

而我们的 JavaScript 语言是一门基于对象的语言，它不是一门真正的面向对象编程的语言。虽然 ES6 提出了 class 编程的方式，但它终究只是一个语法糖，class 编译之后其实就是一个函数。

那么在 JavaScript 中如何实现继承呢？这个时候就用到了原型和原型链，它们非常巧妙地解决了**在 JavaScript 中实现继承的问题！**

### 2.原型和原型对象

在 JS 中，我们所说的原型通常是针对于函数而言的，当然构造函数也是一个函数。

我们都知道函数也是一个对象，是对象那么它就有属性，在 JS 中，我们所创建的每一个函数自带一个属性 prototype，我们就把 prototype 称为原型，有些小伙伴也把它称之为“显示原型”，反正就一个意思。

这个 prototype 它指向了一个对象，你可以把 prototype 想象成一个指针，或者更简单的理解为 prototype 的属性值（键值对）。prototype 指向的这个对象我们就称之为原型对象，通常大家就直接将 prototype 理解为原型对象。

为了让大家更好理解，我们可以在浏览器控制台简单查看一下函数的 prototype，如下图：

![](../images/JS远程/1-3.png)

上图中我们声明了一个 Person 函数，既然函数是对象，那么我们就可以使用“.”来查看它的属性，可以看到有一个 prototype 属性，这个是每一个函数都有的。

我们在代码里面打印出来看看，示例代码如下：

```js
<script>
  function Person() {}
  console.log(Person.prototype)
</script>
```

**输出结果：**

![](../images/JS远程/1-4.png)

我们可以看到原型对象 prototype 里面有一个 constructor 属性，它指向了 Person 构造函数，我们可以画一张图来理解：

![](../images/JS远程/1-5.png)

**总结**

其实原型或者原型对象没有那么复杂，总结下来就下面几点：

每个函数都有 prototype 属性，被称作原型。
prototype 原型指向一个对象，故也称作原型对象。

### 3.prototype 和**ptoto**

很多小伙伴把 prototype 和**proto**混为一潭，其实这是两个维度的东西。prototype 的维度是函数，而**proto**的维度是对象。**proto**是每个对象都有的属性，我们通常把它称为"隐式原型"，把 prototype 称为"显式原型"。

有些小伙伴可能有疑惑，函数也是一个对象，那它是不是也有**proto**属性呢？答案是肯定的，我们可以通过浏览器控制台验证一下。

#### **Function：**

![](../images/JS远程/1-6.jpg)

#### **对象：**

![](../images/JS远程/1-7.png)

我们可以看到函数有 prototype 和**proto**两个属性，而对象只有**proto**属性。接下来我们再来看看**proto**属性有什么呢？

在浏览器控制台进行测试：
![](/images/JS远程/1-8.png)

上图中我们发现了一个新的属性：[[Prototype]]。很多小伙伴会误认为这个就是我们所说的显式原型 prototype，其实不是的，官方对于这个属性其实有解释，我们这里通俗的给大家解释一下：

> [[prototype]]其实就是隐式原型**proto**，因为各大浏览器厂家不同，所以取了别名罢了，大家只需记住这个和**proto**一样即可。
> 上段代码中我们定义了一个空对象，它有一个隐式原型[[prototype]]，隐式原型的 constructor 指向了构造函数 Object。

**总结**

> **proto**和 prototype 不太一样，一个是对象拥有的隐式原型，一个是函数拥有的显式原型，这里我们简单总结一下**proto**：
>
> - 通常被称作隐式原型，每个对象都拥有该属性。
> - [[prototype]]其实就是**proto**。

### 4.原型链

前面两节我们主要介绍了 prototype 和**proto**，那么它们之间有什么关系呢？

为了理清楚之间它们的关系，我们拿出一段示例代码：

```js
<script>
  function Person(name) {
  }
  // 在函数的原型上添加变量和方法
  Person.prototype.name = "小猪课堂";
  Person.prototype.say = function () {
    console.log("你好小猪课堂");
  }

  let obj = new Person();
  console.log(obj.name); // 小猪课堂
  obj.say(); // 你好小猪课堂
</script>
```

上段代码大家应该都很熟悉，我们声明了一个构造函数 Person，其实就是一个函数。我们知道函数的 prototype 是一个对象，我们就可以往这个对象上添加东西，所以我们就直接往函数的原型上添加了变量和方法。

接着我们使用 new 关键词创建一个 Person 构造函数的实例对象，分别打印 name 和调用 say 方法，大家会发现输出结果其实都是 Person 原型上的东西。

这是为什么呢？这其实就和我们的原型链有关了，我们把 obj 打印出来看看。

![](../images/JS远程/1-9.png)

我们会发现 obj 对象上面其实并没有 name 属性和 say 方法，但是在它的隐式原型[[prototype]]上有 name 和 say，而且我们会发现 obj 的[[prototype]]中的 constructor 指向的式它的构造函数 Person。

所以我们大胆的做一个猜想：obj 的隐式原型**proto**是否和构造函数 Person 的显式原型 prototype 相等呢？我们用代码证实一下：

```js
console.log(obj.**proto** === Person.prototype) // true
```

我们发现它们两个果然相等！

接下来我们修改一下我们的代码，我们在 obj 对象上添加一个 name 属性，看看会输出什么。

代码如下：

```js
<script>
  function Person(name) {
    this.name = name;
  }
  // 在函数的原型上添加变量和方法
  Person.prototype.name = "小猪课堂";
  Person.prototype.say = function () {
    console.log("你好小猪课堂");
  }

  let obj = new Person("张三");
  console.log(obj.name); // 张三
  obj.say(); // 你好小猪课堂

  console.log(obj)
</script>
```

**输出结果：**

![](../images/JS远程/2-1.png)

上段代码中我们 obj 上面有自己的 name 属性了，这个时候输出的就是 obj 自带的 name 属性。到这里我们又可以做一个大胆的猜想：**obj 对象想要获取 name 或者 say，首先判断自己的属性当中有没有，如果没有找到，那么就在**proto**属性中去找，而这个时候**proto**与 Person 的 prototype 是相等的，也就是**proto**指向 Person，那么便可以找到 name 和 say。**

上面的猜想非常的正确！我们还可以在上面的猜想上扩展一下，既然函数的 prototype 是一个对象，那么它必然有**proto**，当我们在函数的原型上没有找到的时候，我们又继续查找 prototype 的**proto**，以此下去，直到找到或者**proto**没有指向某个构造函数为止。

上面的查找过程是不是很像链式查找啊！而这就是我们所说的原型链，而且我们发现查找的过程主要是通过**proto**原型来进行的，所以**proto**就是我们原型链中的连接点。

**总结：**

上面的查找的过程形成的一条线索就叫做原型链，大家可以把原型链拆开来理解：原型和链。

- 原型就是我们的 prototype
- 链就是**proto**，它让整个链路连接起来。
  想要理解原型链，我们还得理解**proto**指向哪儿，也就是说它指向那个构造函数，比如上面的 obj 对象的**proto**指向的就是 Person 构造函数，所以我们继续往 Person 上查找。

最后我们上一张经典的图，相信大家能看懂了：

![](../images/JS远程/2-2.png)

上面这张图看似很复杂，但是我们理解了 prototype 和**proto**之后很简单，大家按照下面的思路去看上面这张图会很简单的：

1. 上面很多虚线，我们发现虚线上都有**proto**属性，所以可以看出来**proto**就是一个连接的作用。
2. 上图中无非有三个构造函数：Foo、Object、Function，我们都知道每个函数都有一个 prototype 显示原型，而且这个显示原型指向了自身这个构造函数。
3. 接着我们在看图中的 new 关键字，我们知道 new 创建的对象都有一个**proto**隐式原型，而且这个隐式原型执行了它的构造函数，也就是**proto** === prototype。
4. Foo 的隐式原型**proto**指向的是 Function 的 prototype，因为函数是属于 Function 这个构造函数的。所以上图中的 Foo 和 Object 的**proto**都指向了 Function 的 prototype。

**总结**

利用原型链这种链式查找的方式，我们就巧妙地实现了继承！要理解原型和原型链其实不难，主要是大家还是要有面向对象的思想，比如通过 new 关键词创建实例，构造函数是什么？
