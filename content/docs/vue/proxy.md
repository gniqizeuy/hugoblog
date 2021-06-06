+++
title = "proxy"

date = "2021-05-20"
categories = [
"vue"
]

menu = "main"

+++

### vue 查看代理请求的真实地址
在文件 vue.config.js 文件里面找到 proxyTable 添加配置。

```js
'/api': {
    traget: url,
    changeOrigin: true
    loglevel: 'debug'
}
```