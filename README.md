分布式服务框架
==============

+ 基于Redis来传递消息
+ 无管理主机，由节点自主维护
+ 服务器任意加入
+ 简单出错处理机制：调用超时自动重新调用，超过一定次数时返回调用失败错误
+ 目前没有做任何性能测试
+ 可以随意启动多个节点进程，相同的服务也可以派发到多个不同节点来处理


## 服务器端

```javascript
var clouds = require('clouds');

// 创建服务器
var server = new clouds.Server({
  // redis连接配置
  redis: {
    host: '127.0.0.1',
    port: 6379,
    db: 3
  },
  // 心跳周期，如果服务器端异常下线，超过指定时间将自动从服务器端删除，单位：秒
  heartbeat: 2
});

// 注册服务处理程序
server.register('test.hello', function (name, msg, callback) {
  // 函数的最后一个参数表示回调函数，客户端在调用的时候必须保证参数数量是一致的
  // 回调函数第一个参数表示是否出错，第二个参数起表示返回的结果
  callback(null, 'Hello ' + name + ', ' + msg);
});
```

## 客户端

```javascript
var clouds = require('clouds');

var client = new clouds.Client({
  // redis连接配置
  redis: {
    host: '127.0.0.1',
    port: 6379,
    db: 3
  },
  // 调用超时时间，如果服务器超过指定时间没有响应结果，则认为调用失败，单位：秒
  timeout: 2
});

// 返回一个函数，用于直接调用远程服务
var testHello = client.bind('test.hello');

// 调用远程服务，跟使用本地普通函数差不多
testHello('Glen', 'timestamp is ' + Date.now(), function (err, ret) {
  console.log(err, ret);
});

// 也可以这样直接调用，第一个参数是服务名，第二个参数是调用参数数组，第三个参数是回调函数
client.call('test.hello', ['Glen', 'timestamp is ' + Date.now()], function (err, ret) {
  console.log(err, ret);
});
```

## 出错处理

客户端在初始化时可设置一个超时时间，如果调用的服务超过该时间没有返回结果，将返回一个服务超时的错误。

在`Client.bind()`时可以指定自动重试的次数（仅当重试次数超过指定值时才放弃，并执行回调函数），比如：

```javascript
// 返回一个函数，用于直接调用远程服务，第一个参数是服务名，第二个参数最大重试次数
var testHello = client.bind('test.hello', 5);

// 调用远程服务，跟使用本地普通函数差不多
testHello('Glen', 'timestamp is ' + Date.now(), function (err, ret) {
  console.log(err, ret);
});
```

## 消息

`Client`和`Server`均可互相发送消息：

```javascript
// 接收消息
client.on('message', function (sender, msg) {
  // sender表示消息发送者的ID
  // msg为消息内容
});

// 发送消息
client.send('receiver', 'msg');
```


授权
===========

基于MIT协议发布

