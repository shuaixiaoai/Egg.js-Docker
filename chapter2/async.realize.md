## 异步操作的几种实现

#### 利用setTimeout模拟实现异步任务
```
function my_async_function (name, fn) {
    setTimeout(() => {
        fn(null, "_" + name + "_");
    }, 3000);
}

my_async_function("hello.js", (err, name) => {
    if (err) {
        console.log(err);
    } else {
        console.log(name);
    }
})
```

#### 事件实现异步
```
class Event {
    constructor() {
        this.map = {};
    }
    add (name, fn) {
        if (this.map[name]) {
            this.map[name].push(fn);
            return;
        }

        this.map[name] = [fn];
        return;
    }
    emit(name, ...args) {
        this.map[name].forEach(fn => {
            fn(...args);
        });
    }
}

let e = new Event();
e.add("hello", (err, name) => {
    if (err) {
        console.error(err);
        return;
    }
    console.log(name);
});

e.emit("hello", "发生了错误");
e.emit("hello", null, "hello nodejs");

// 对于事件，通常会有一个名字，比如单击事件， 所以要有一个name参数， 然后把name参数作为key传到map里面， 当触发emit的时候， 获得对应的回调列表进行调用即可；
// 优化上述代码，支持链式调用。

class ChainEvente {
    constructor () {
        this.map = {};
    }

    add (name, fn) {
        if (this.map[name]) {
            this.map[name].push(fn);
            return;
        }

        this.map[name] = [fn];
        return this;
    }

    emit(name, ...args) {
        this.map[name].forEach(fn => {
            fn(...args);
        });
        return this;
    }
}

let e2 = new ChainEvente();
e2.add("hello", (err, name) => {
    if (err) {
        console.error(err);
        return;
    }
    console.log(name);
});

e2.emit("hello", "发生了错误");
e2.emit("hello", null, "hello nodejs");

// 把事件加入异步编程里面， 只需要在异步的方法里面触发这个事件即可， 其本质还是回调， 只是换一种形式而已， 并没有从根本上解决异步回调的问题。
```

#### 观察者模式实现异步
```
function create(fn) {
    let ret = false;
    return ({ next, complete, error }) => {
        function nextFn(...args) {
            if (ret) return;
            next(...args);
        }

        function completeFn(...args) {
            complete(...args);
            ret = true;
        }

        function errorFn(...args) {
            error(...args);
        }

        fn({
            next: nextFn,
            complete: completeFn,
            error: errorFn
        });

        return () => { ret = true; };
    };
}

let observable = create(observer => {
    setTimeout(() => {
        observer.next(1);
    }, 1000);

    observer.next(2);
    observer.complete(3);
});

const subject = {
    next: value => {
        console.log(value);
    },
    complete: console.log,
    error: console.log
};

let unsubscribe = observerable(subject);

// 范例借鉴Rx.js(相当于增强版Promise)
```

#### Promise异步
```
// promise本质上还是回调， 只是其特有书写格式以及状态规范使其成为javascript标准
// 有点低版本浏览器以及低版本nodejs不支持Promise规范， 需要添加兼容实现库： promise-ployfill/babel-ployfill等

const getName = new Promise(( resolve, reject ) => {
    setTimeout(() => {
        resolve("nodejs);
    }, 50);                             // 其中50ms为js监听时间最小值
});

const getNumber = Promise.resolve(1);
const getError = promise.reject("出错啦。。。");

getError.catch(console.log);

Promise.all([getName, getNumber])
    .then(console.log)
    .catch(console.log)
Promise.race([getName, getNumber])
    .then(console.log)
    .catch(console.log)

getName()
    .then( name => {
        console.log(name);
        return 20;
    })
    .then(number => {
        console.log(number);
    })

// 输出结果如下： 
// 出错啦。。。
// nodejs
// ['nodejs', 'nodejs']
// nodejs
// 20
```

#### async/await 异步
```
// node8.9之后原生支持了async/await关键字咯。 前端js可以通过babel转义则可。
// 异步的本质还是回调， 而async/await可以在语法层面上规避回调

async function func() {
    return 2;
}

func().then(console.log);

const getPosts = () => {
    new Promise(( resolve, reject ) => {
        resolve([
            {
                name: "a"
            },
            {
                name: "b"
            },
            {
                name: "c"
            },
            {
                name: "d"
            }
        ]);
    });
};

async function func2() {
    try {
        const number = await func();
        const posts = await getPosts();
        console.log(number);
        console.log(posts);
    } catch (e) {
        console.log(e);
    }
}

func2();

//  运行结果如下： 
//  2
//  2
// 【 { name: "a" }, { name: "b" }, { name: "c" }, { name: "d" }  】

// async修饰function， 说明这是一个异步的方法， 当调用的时候， 它会放回一个Promise， 只有通过then方法才能获取返回值
// 在调用async修饰的function时， 返回的是Promise， 而把await放在Promise前， 就可以获取被Promise包裹的值。
// 通常async与await是成对出现的。 await后面可以跟Promise和其他的async函数， 也可以是一般的同步函数。 使用async/await之后可以使用try {} catch (e) {}捕获异常。
```