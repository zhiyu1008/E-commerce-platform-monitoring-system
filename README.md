

## 一. 项目介绍

### 1. 项目截图

![](https://shizhiyuya.oss-cn-beijing.aliyuncs.com/images/image-20220802162403658.png)

https://www.bilibili.com/video/BV1bh41197p8?p=1&vd_source=3828dbbb5c12e12caae05cbc226df429

### 3. 项目所用到的知识点

1. 前端用到了 vue2+echarts
2. 服务端用到了 koa2+websocket

## 二. 运行项目

### 1. 下载依赖

```
npm install
```

### 2. 运行服务端项目

```
node app.js
```

> 服务端主动向客户端推送数据

### 3. 运行前端项目

```
npm run serve
```

>如果执行 npm run serve 时报错，可尝试执行以下命令

```
set NODE_OPTIONS=--openssl-legacy-provider
```

