---
title: uniapp上线ios流程
tags:
  - uniapp
  - ios
date: 2023-5-9
author: Eric
comments: false
cover: /img/22.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

---

# 前言

使用 uniapp 开发多端系统，总是绕不过上线安卓和 ios，安卓端的 HBuilderX/HBuilder 会提供我们免费的证书打包测试，但是当我们要测试 ios 时就比较麻烦了，踩坑之后记录下自己上线 iOS 的一些流程。

---

# 一、pandas 是什么？

示例：pandas 是基于 NumPy 的一种工具，该工具是为了解决数据分析任务而创建的。

# 二、使用步骤

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
  "https://labfile.oss.aliyuncs.com/courses/1283/adult.data.csv"
);
print(data.head());
```

该处使用的 url 网络请求的数据。

---

# 总结

提示：这里对文章进行总结：
例如：以上就是今天要讲的内容，本文仅仅简单介绍了 pandas 的使用，而 pandas 提供了大量能使我们快速便捷地处理数据的函数和方法。
