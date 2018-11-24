## koajs简介

###### koajs是实现了中间件的很小的精简的node服务器。 koa.js本身只自带中间件与请求响应的一些帮助方法， 甚至是路由、试图渲染、数据库都没有。 只能通过相关中间件实现。

```
// 编程风格实例代码如下

async function a(ctx, next) {
    console.log(1);
    const hello = await Promise.resolve("hello nodejs");
    console.log(hello);
    await next();
    console.log(" a end);
}

async function b(ctx, next) {
    console.log(2);
    const hello = await Promise.resolve("hello nodejs");
    console.log(hello);
    await next();
    console.log("b end");
}

compose([a, b])({});
```

###### 创建一个中间件
```
// 每一个中间件都会接受ctx、next作为参数， next是下一个回调的Promise， 而ctx是Koa封装的上下文， 这个对象包含响应和请求的所有方法。 当我们想添加一些方法的时刻可以直接顾阿斋在ctx上。
// 创建log.js

module.exports = options => {
    if (!options.format) {
        console.log("需要传递format 函数");
    }

    return async (ctx, next) => {
        console.log(options.format(ctx.url));
        await next();
    }
}
```

#### Egg.js基础知识
* app/router.js, 路由映射配置；
* app/controller, 存放控制器的目录， 并用来处理跳转相关的逻辑；
* app/service, 用来存放业务逻辑；
* app/middleware, 存放中间件的目录；
* app/public, 存放静态文件的目录；
* app/extends, 扩展框架的目录， 比如在ctx之上添加一些变量和方法等；
* config, 配置目录、中间件的配置项、环境配置变量等；
* test, 测试文件目录；
* app.js， 可以在该文件添加启动钩子；
