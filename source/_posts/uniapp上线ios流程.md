---
layout:
  - post
title: uniapp上线ios流程
tags: Eric的博客
author: Eric
---

---

## 前言

使用uniapp开发多端系统，总是绕不过上线安卓和ios，安卓端的HBuilderX/HBuilder会提供我们免费的证书打包测试，但是当我们要测试ios时就比较麻烦了，踩坑之后记录下自己上线iOS的一些流程。

---

## 一、pandas是什么？

示例：pandas 是基于NumPy 的一种工具，该工具是为了解决数据分析任务而创建的。

## 二、使用步骤

### 1.引入库

代码如下（示例）：

```c++
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings('ignore')
import  ssl
ssl._create_default_https_context = ssl._create_unverified_context
```

### 2.读入数据

代码如下（示例）：

```js
data = pd.read_csv(
    'https://labfile.oss.aliyuncs.com/courses/1283/adult.data.csv')
print(data.head())
```

该处使用的url网络请求的数据。

---

## 总结

提示：这里对文章进行总结：
例如：以上就是今天要讲的内容，本文仅仅简单介绍了pandas的使用，而pandas提供了大量能使我们快速便捷地处理数据的函数和方法。
