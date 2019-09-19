---
title: '异步之回调,promise,async/await'
tags:
  - JavaScript
categories:
  - 前端
date: 2019-03-02 16:19:52
updated: 2019-03-02 16:19:52
---
# 前言
场景：点击按钮下载商品数据。涉及到异步账户检查，首先有没有登录，如果登录了，再检查是否是合法用户，如果是合法的那么继续下面操作。
# 概述
在JavaScript中处理异步请求是用的回调函数，对于需要顺序处理异步请求的，就得一层层嵌套回调，层数多了就很难维护了，就是大家说的“回调地狱”，怎么才能像同步代码那样优雅的写呢？这就是ES6的promise及ES7的async/await要解决的问题。
<!-- more -->
# 详述

## 示例一：回调函数
```js
// 执行下载数据
function doExportData(data) {
    console.log('[export data]param:' + JSON.stringify(data));
}

function exportData() {
    // 模拟ajax检查是否登录
    setTimeout(() => {
        let response = {
            code: '10001',
            data: {
                param1: '12345'
            }
        };
        console.log('检查登录请求完成：' + response);
        // 模拟若已经登录了，再检查是否为合法的用户
        if (response.code === '10001') {
            setTimeout(() => {
                let response2 = {
                    code: '10001',
                    data: {
                        param2: 'abcde'
                    }
                };
                console.log('检查合法请求完成：' + response2);
                if (response2.code === '10001') {
                    console.log('callback');
                    doExportData(response.data.param1, response2.data.param2);
                }

            }, 500)
        }

    }, 500)
}
exportData();
// 执行结果
// 检查登录请求完成：[object Object]
// 检查合法请求完成：[object Object]
// callback
// [export data]param:"12345"
```
## 示例二：Promise
```js
// 执行下载数据
function doExportData(data) {
    console.log('[export data]param:' + JSON.stringify(data));
}


let myPromise = new Promise((resolve, reject) => {
    // 模拟异步请求返回数据
    setTimeout(() => {
        let response = {
            code: '10001',
            data: {
                param1: '12345'
            }
        };
        if (response.code === '10001') {
            resolve(response.data.param1);
        }

    }, 500)
});
myPromise.then((param1) => {
    return new Promise((resolve, reject) => {
        // 模拟异步请求返回数据
        setTimeout(() => {
            let response2 = {
                code: '10001',
                data: {
                    param2: 'abcde'
                }
            };
            if (response2.code === '10001') {
                resolve({ param1: param1, param2: response2.data.param2 });
            }

        }, 500)
    })
}).then((data) => {
    console.log('promise');
    doExportData(data.param1, data.param2);
})


// promise
// [export data]param:"12345"
```

## 示例三：async await

```js
// 执行下载数据
function doExportData(data) {
    console.log('[export data]param:' + JSON.stringify(data));
}

function ajax1() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            let response = {
                code: '10001',
                data: {
                    param: '12345'
                }
            };
            // let response = {
            //     code: '10002',
            //     data: {
            //         param: 'ajax1 error'
            //     }
            // };
            if (response.code === '10001') {
                resolve(response.data);
            } else {
                reject(response.data);
            }

        }, 500)
    })

}

function ajax2(data1) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            let response = {
                code: '10001',
                data: {
                    param: 'abcde'
                }
            };
            // let response = {
            //     code: '10002',
            //     data: {
            //         param: 'ajax2 error'
            //     }
            // };
            if (response.code === '10001') {
                resolve({ param1: data1.param, param2: response.data.param });
            } else {
                reject(response.data);
            }
        }, 500)
    })

}

async function exportData(params) {
    try {
        let data1 = await ajax1();
        let data = await ajax2(data1);
        console.log('async await');
        doExportData(data);
    } catch (error) {
        console.log(error);
    }

}
exportData();

// async await
// [export data]param:{"param1":"12345","param2":"abcde"}
```

其实async await只是promise的魔法糖，例如示例三可调整为如下：
```js
let myPromise = ajax1();
myPromise.then((data1) => {
    return ajax2(data1);
}).then((data) => {
    console.log('promise');
    doExportData(data);
})

// promise
// [export data]param:{"param1":"12345","param2":"abcde"}
```

# 后记
其实promise也只是对回调函数进行的包装，通过观察者模式实现形式上的同步代码，需要深入研究的请参阅[Promise原理与实现](https://www.jianshu.com/p/b4f0425b22a1)。