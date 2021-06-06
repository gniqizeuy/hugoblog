+++
title = "other"

date = "2021-05-20"
categories = [
"vue"
]

menu = "main"

+++

### vue-cli3 控制台network存在 info

服务端：[sockjs-node](https://github.com/sockjs/sockjs-node)  
客户端：[sockjs-clien](https://github.com/sockjs/sockjs-client)

如果你的项目没有用到 sockjs，vuecli3 运行 npm run serve 之后 network 里面一直一个接口：http://localhost:8080/sockjs-node/info?t=1462183700002

#### 关闭info
1.找到/node_modules/sockjs-client/dist/sockjs.js

2.注释下面代码
```js
    try {
    //  self.xhr.send(payload);
    } catch (e) {
    self.emit('finish', 0, '');
    self._cleanup(false);
    }
```

