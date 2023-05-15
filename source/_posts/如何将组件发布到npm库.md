---
title: 如何将组件发布到npm库
tags: Eric的博客
author: Eric
---
# 发布一个自己的组件到npm
> 每天我们都在使用组件开发，但是如何发布一个我们自己的组件呢？
> go~~~

1. 搭建发布环境

   - 下载node（<http://nodejs.cn/download/）>
   - 下载vue-cli（npm install  @vue/cli）

2. 创建一个vue项目（ vue create myui ）
3. 修改默认文件夹src为examples
4. 新建一个packages文件夹
5. 修改vue.config.js文件
![在这里插入图片描述](/images/发布npm库/1-1.png)

### 一、 编写组件代码

- 先新建一个简单的button组件，
- Packages文件夹下新建一个组件文件夹和入口文件index.js
![在这里插入图片描述](/images/发布npm库/1-2.png)

- 然后srcs文件夹下存放组件源代码
![在这里插入图片描述](/images/发布npm库/1-3.png)
- 在button下的index.js l里编写如下代码 作为vue组件的安装（详细文档可以看vue官网的组件篇）
![在这里插入图片描述](/images/发布npm库/1-4.png)

- 接下来在packages的入口文件中导入组件并安装导出
![在这里插入图片描述](/images/发布npm库/1-5.png)
- 组件编写完毕 进入packages文件夹下 实例化一个package.json（npm init -y）
![在这里插入图片描述](/images/发布npm库/1-6.png)
  - name：包名，version：版本号，description：组件描述，main:入口文件

- 登录npm官网（<https://www.npmjs.com）>
  - 没有账号新注册一个
  - 切换到npm官方源
    - 查看npm当前源（ npm config get registry 或者 npm config list ）
    - 设置回原本的源，用来发布npm包（ npm config set registry <https://registry.npmjs.org/> ）
    - 设置为淘宝镜像（npm config set registry <https://registry.npm.taobao.org）>
- 首次需要设置用户名密码绑定npm
  - npm adduser ( 输入改行密令 会让你绑定自己的账号 按提示输入 )
  - 邮箱认证（ 点击npm官网头像选择账户，点击2FA认证）
 ***注意：认证需要下载一个authenticator应用程序（问我要）***
  - 切换到要发布的包（ cd packages ）
  - 执行发布密令（ npm publish ）
- 返回npm官网查看发布是否成功

- 发布过程中容易出现的错误 npm adduser / npm publish 报错：
  >  [https://blog.csdn.net/meteorsshower2013/article/details/121957217?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1.pc_relevant_default&spm=1001.2101.3001.4242.2&utm_relevant_index=4）](%EF%81%B5%09https://blog.csdn.net/meteorsshower2013/article/details/121957217?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1.pc_relevant_default&spm=1001.2101.3001.4242.2&utm_relevant_index=4%EF%BC%89)
  >
  >  Npm双因素身份认证（2FA）：
  >  [https://segmentfault.com/a/1190000041025567](https://segmentfault.com/a/1190000041025567)
