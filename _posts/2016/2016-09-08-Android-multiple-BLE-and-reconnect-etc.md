---
layout: "post"
title: "Android BLE 的使用:多设备连接和断线重连等"
date: "2016-08-09 22:38"
category: Android
tags: [BLE, 断线重连]
---

一直想写一篇BLE的使用总结,正好昨天翻笔记翻到了BLE多设备这个,为了防止忘掉(iOS端已然忘掉),赶紧整理一下.

这里的几个问题大概是老鸟觉得简单懒得总结,新人又很难有思路的问题,
下边写一下具体解决方法,当然未必是最优解仅供参考.

### BLE多设备连接

刚开始完全不知道该怎么搞,官方文档里也没提多设备,

最初得到答案的时候我是拒绝的,因为我已经想了辣么久，答案竟然是直接连接就可以~

#### 方法一 直接连接

就跟单个连接一样,继续在`connect`函数里填入要连接的BLE外设.
然后需要在 对应的 `GattCallback` 回调里区分设备,
比如在`onCharateristicChanged`这个里边通过物理地址来判断是从哪个设备得到的数据.

```
// 此处为了节省空间,部分定义和非空判断略去了
public boolean connect(final String address){
  final BluetoothDevice mDevice = mBluetoothAdapter.getRemoteDevice(address);
  mBluetoothGatt = mDevice.connectGatt(this, false, mGattCallback);
  // 此处自动重连速度比iOS端慢很多,建议false,在 onConnectionStateChange 处理
  return true;
}
```


#### 方法二 每一个设备对应一个`BluetoothGattCallback`回调

就是每个设备都新建一个`BluetoothGattCallback`的回调,像这样:

```
private final BluetoothGattCallback mGattcallbackOne = new BluetoothGattCallback();
private final BluetoothGattCallback mGattcallbackTwo = new BluetoothGattCallback();
```

根据实际使用中的复杂度和可靠度这里推荐第二种.

### 断线重连

断线重连这里,官方提供了方法(方法一中的注释),然而重连的速度跟iOS相比实在太慢了,所以这里选择自己处理,

就是在`GattCallback`中的`onConnectionStateChange` 这里发送一个`已断开`的广播到处理蓝牙操作的后台服务,在服务里判断 disconnect 后使用 close 方法 释放资源,然后再connect方法连接(我把蓝牙相关的操作和数据处理独立到一个单独的服务里).

```
mBLEService.close();
mBLEService.connect(deviceAddress);

/** close函数主要内容是这两句
  mBluetoothGatt.close();
  mBluetoothGatt = null;
**/
```


### 绑定

大概思路就是展示一个过滤后的设备列表,使用`SharedPreferences`把要绑定MAC地址存储起来,启动的时候去读取,如果存在执行标准的BLE连接操作,省略了扫描这一步.

### 判断蓝牙开启状态

为了快速的连接设备,这里一启动就尝试开启蓝牙,至于开启结果可以在`onActivityResult()`里判断,

具体方法就是先判断requestCode是否为`2`,如果是再判断resultCode是否为`－1`

```
// public static final int REQUEST_ENABLE_BT = 2;
// public static final int RESULT_OK = -1;
//  0 开启成功, -1 开启失败
```
