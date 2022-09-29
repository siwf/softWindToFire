# 如何优雅处理 async await 错误

> 发现团队里目前项目中在使用 async await 的时候，是通过 try catch 来进行错误处理的；而 try catch 这个东西本身的设计是处理未知错误的；并且大量的 try catch 使我们从promse的链式地狱转变成了 try catch 地狱

## 总结一下目前处理 async await 错误的两种方式

### 一、 promise.then.catch 的方式

如果`await`后面的异步操作出错，那么等同于`async`函数返回的 Promise 对象被`reject`

```javascript
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出错了
复制代码
```

或者

```javascript
async function f() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
复制代码
```

### 二、try catch 的 方式

```javascript
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出错了');
    });
  } catch(e) {
  }
  return await('hello world');
}
复制代码
```

## 解决方案

在 await 处理 promise 返回的时候 增加一个数据的拦截机制

![img](../image/e1e472aec3894e3ea0628e8f74df289d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

改造后

![img](../image/9acb96a4d7254c36bf2e4dbabc69392e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

```javascript
async function f() {
	const [err, data] = toAwait(Promise.reject('wrong'))

}
```

