---
title: uniapp开发安卓ios实时保证应用常活
tags:
  - uniapp
date: 2023-3-10
author: Eric
comments: false
cover: /img/19.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true
position: both #封面显示的位置# 三个值可配置left , right , both
---

# 前言

> [uniapp 开发 app](https://so.csdn.net/so/search?q=uniapp%E5%BC%80%E5%8F%91app&spm=1001.2101.3001.7020)端时候，某些业务场景需求：在后台不间断（间隔一定时间）向服务器发送用户定位信息，不管页面是否关闭，app 是否处于前后台，发送位置信息功能必须持续，直到某一个页面操作行为触发事件才停下来。

---

# 一、核心 api

> 开启定位 plus.geolocation.watchPosition(fn)  
> 关闭定位 plus.geolocation.clearWatch(id)

# 二、代码实现

### 1.locationWatcher.js

> 定位工具类 locationWatcher.js：

```js
export default {
  //检测是否开启系统定位权限
  hasLocationPermission() {
    let system = uni.getSystemInfoSync();
    if (system.platform === "android") {
      //安卓
      let context = plus.android.importClass("android.content.Context");
      let locationManager = plus.android.importClass(
        "android.location.LocationManager"
      );
      let main = plus.android.runtimeMainActivity();
      let service = main.getSystemService(context.LOCATION_SERVICE);
      //已开启系统定位服务功能
      if (service.isProviderEnabled(locationManager.GPS_PROVIDER)) return true;
      else {
        //未开启引导开启
        uni.showModal({
          title: "友情提示",
          content: "请开启位置服务功能",
          success: (e) => {
            if (e.confirm) {
              //打开手机系统gps定位设置页面
              let Intent = plus.android.importClass("android.content.Intent");
              let Settings = plus.android.importClass(
                "android.provider.Settings"
              );
              let intent = new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS);
              main.startActivity(intent);
            }
          },
        });
      }
    } else if (system.platform === "ios") {
      //ios
      let cllocationManger = plus.ios.import("CLLocationManager");
      let enable = cllocationManger.locationServicesEnabled();
      let status = cllocationManger.authorizationStatus();
      plus.ios.deleteObject(cllocationManger);
      if (enable && status != 2) return true; //已开启定位功能
      else {
        uni.showModal({
          title: "友情提示",
          content: "请前往设置-定位服务打开定位服务功能",
          success: (e) => {
            if (e.confirm) {
              let UIApplication = plus.ios.import("UIApplication");
              let application = UIApplication.sharedApplication();
              let NSURL = plus.ios.import("NSURL");
              let setting = NSURL.URLWithString("app-settings:");
              application.openURL(setting);
              plus.ios.deleteObject(setting);
              plus.ios.deleteObject(NSURL);
              plus.ios.deleteObject(application);
            }
          },
        });
      }
    }
    return false;
  },
  /**开启后台持续获取定位功能
   * successCallBack:成功回调函数
   *failCallBack:失败回调函数
   *maximumAge：获取定位间隔时间
   */
  startLocationService(
    successCallBack = () => {},
    failCallBack = () => {},
    maximumAge = 60 * 1000
  ) {
    if (this.hasLocationPermission()) {
      //有定位权限
      let locationWatcherId = plus.geolocation.watchPosition(
        (position) => {
          successCallBack({
            locationWatcherId,
            position: position.coords,
          });
        },
        function (e) {
          console.log(e, "定位失败");
          failCallBack(e);
        },
        {
          maximumAge, //获取位置间隔时间
        }
      );
    }
  },
  //关闭定位功能
  closeLocationService(locationWatcherId) {
    //locationWatcherId:开启步骤生成的监听器id
    plus.geolocation.clearWatch(locationWatcherId);
  },
};
```

### 2.页面引用

代码如下（示例）：

```js
<template>
 <view>
  <button @click="open">开启</button>
  <button @click="close">关闭</button>
 </view>
</template>

<script>
 import locationWatcher from './locationWatcher.js'
 export default {
  data() {
   return {
    locationWatcherId:'',//监听器id
    maximumAge: 10 * 1000//间隔时间10s
   }
  },
  methods: {
   //开启定位功能
         open(){
    locationWatcher.startLocationService(e=>{
     let {latitude,longitude}=e.position
     console.log(`当前位置，经度${longitude},纬度${latitude}`)
     if(!this.locationWatcherId){
      this.locationWatcherId=e.locationWatcherId//举例保存到data，实际可以缓存到全局
     }
    },(e)=>{
     console.log(e,'定位失败')
    },this.maximumAge)
   },

   //关闭定位
   close(){
    this.locationWatcherId&&locationWatcher.closeLocationService(this.locationWatcherId)
   }
  }
 }
</script>
```

### 运行结果

![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57bd5762162c452b89ef816e8c3742e7~tplv-k3u1fbpfcp-zoom-1.image)

---

# 三、注意事项

> 1.定位功能需要引入地图 appkey，申请百度或高德地图 key,在 manifest.json-APP 模块-Maps 配置  
> 2.locationWatcherId 实际开发需要全局缓存，在需要关闭定位功能地方调用  
> 3.定位功能跟 app 生命周期一致，后台 app 进程被杀死或完全退出 app 功能停止  
> 4.定位功能授权弹窗必须选择允许
