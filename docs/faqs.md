# FAQs

## columns.js 增加自定义事件
columns里的`formItem`是直接用的antd里的对应组件 比如默认 `formItem: {}` 这样写用的就是antd的`Input`组件`formItem: { type: 'select' }`就是antd的Select下拉组件，要写事件的话直接在formItem里定义就行了,之后会传给对应组件,例：
```js
...
{
  title: '角色名',
  name: 'roleName',
  formItem: {
    onKeyDown: e => console.log(e)
  }
},
...
```

## 如何配代理 proxy （开发环境下生效）

当请求后端接口时，我们使用反向代理方式，在`package.json`中进行设置，更多设置看[create-react-app](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#proxying-api-requests-in-development)
```json
"proxy": {
  "/api": {
    "target": "http://192.168.202.12:8391",
    "changeOrigin": true,
    "pathRewrite": {
      "^/api": ""
    }
  }
},

// 例 fetch('/api/user/getDetail') -> 'http://192.168.202.12:8391/user/getDetail'

// 可配置多个, 注意顺序，最大范围的要放到最下面，
"proxy": {
  "/api/v1": {
    "target": "http://192.168.202.11",
    "changeOrigin": true,
    "pathRewrite": {
      "^/api": ""
    }
  },
  "/api/v2": {
    "target": "http://192.168.202.12",
    "changeOrigin": true,
    "pathRewrite": {
      "^/api": ""
    }
  },
  "/api": {
    "target": "http://192.168.202.13",
    "changeOrigin": true,
    "pathRewrite": {
      "^/api": ""
    }
  }
},
```

## 如何配反向代理 nginx （生产环境下）

1. 打开nginx配置文件`nginx.conf`，与上面效果一样，在`server`下增加
```js
location /api/v1/ {
  proxy_pass http://192.168.202.11/v1/;
}

location /api/v2/ {
  proxy_pass http://192.168.202.12/v2/;
}

location /api/ {
  proxy_pass http://192.168.202.13/;
}
```
更多配置自行查找...

2. 重启nginx

## 不配代理，后台配跨域（不推荐）

场景一：如果后台设置如下 （推荐）
```js
Access-Control-Allow-Origin *;
Access-Control-Allow-Credentials true // 重要
```
则前台可以直接发请求（可能会有一条options请求），不用改配置

场景二：如果后台设置如下
```js
Access-Control-Allow-Origin *;
```

如果前台不改配置，会报错<font color="red">device:1 Failed to load http://localhost:8080/test: Response to preflight request doesn't pass access control check: The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'. Origin 'http://localhost:3000' is therefore not allowed access.</font>


需要前台需改配置`config.js`中的`request`
```js
  request: {
    prefix: '/api',
    credentials: 'omit', // credentials改为omit，fetch的默认值
```
在发请求应该就ok了，因为可能不是简单请求同样会有一条options

## 发布路径

build项目的时候注意在`config-overrides.js`中配置正确的`publicPath`,例如放到`demo`文件夹下为
```js
config.output.publicPath = '/demo/'; // 跟据实际项目设置
```
若发布到服务器的跟目录下为
```js
config.output.publicPath = '/'; // 跟据实际项目设置
```
配置错则有可能加载不到相关资源

## 使用`$$.post, $$.get`等发送请求时，注意处理反回异常
例：
```js
$$.get('url').then(resp => {
  // ...more      
}).catch(e => console.err(e)); // 最好处理下错误
```
因为：在`config.js`中的`afterResponse`会再次抛出错误，需要我们自已处理
