new Promise((resolve, reject) => {
  console.log("log: 外部promise");
  resolve();
})
  .then(() => {
    console.log("log: 外部第一个then");
    new Promise((resolve, reject) => {
      console.log("log: 内部promise");
      resolve();
    })
      .then(() => {
        console.log("log: 内部第一个then");
      })
      .then(() => {
        console.log("log: 内部第二个then");
      });
  })
  .then(() => {
    console.log("log: 外部第二个then");
  });

// log: 外部promise
// log: 外部第一个then
// log: 内部promise
// log: 内部第一个then
// log: 外部第二个then
// log: 内部第二个then



！！！执行 then 方法是同步的，而 then 中的回调是异步的

new Promise((resolve, reject) => {
  resolve();
}).then(() => {
  console.log("log: 外部第一个then");
});
实例化 Promise 传入的函数是同步执行的，then 方法本身其实也是同步执行的，
但 then 中的回调会先放入微任务队列，等同步任务执行完毕后，再依次取出执行，换句话说只有回调是异步的

同时在同步执行 then 方法时，会进行判断：

！！！如果前面的 promise 已经是 resolved 状态，则会立即将回调推入微任务队列（但是执行回调还是要等到所有同步任务都结束后）

！！！如果前面的 promise 是 pending 状态则会将回调存储在 promise 的内部，一直等到 promise 被 resolve 才将回调推入微任务队列





当一个 promise 被 resolve 时，会遍历之前通过 then 给这个 promise 注册的所有回调，将它们依次放入微任务队列中

如何理解通过 then 给这个 promise 注册的所有回调，考虑以下案例

let p = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000);
});
p.then(() => {
  console.log("log: 外部第一个then");
});
p.then(() => {
  console.log("log: 外部第二个then");
});
p.then(() => {
  console.log("log: 外部第三个then");
});
1 秒后变量 p 才会被 resolve，但是在 resolve 前通过 then 方法给它注册了 3 个回调，
此时这 3 个回调不会被执行，也不会被放入微任务队列中，它们会被 p 内部储存起来（在手写 promise 时，这些回调会放在 promise 内部保存的数组中），
等到 p 被 resolve 后，依次将这 3 个回调推入微任务队列，此时如果没有同步任务就会逐个取出再执行

另外还有几点需要注意:

对于普通的 promise 来说，当执行完 resolve 函数时，promise 状态就为 resolved

resolve 函数就是在实例化 Promise 时，传入函数的第一个参数

new Promise(resolve => {
  resolve();
});
！！！它的作用除了将当前的 promise 由 pending 变为 resolved，
！！！还会遍历之前通过 then 给这个 promise 注册的所有回调，将它们依次放入微任务队列中，
！！！很多人以为是由 then 方法来触发它保存回调，而事实上 then 方法即不会触发回调，也不会将它放到微任务，
！！！then 只负责注册回调，由 resolve 将注册的回调放入微任务队列，由事件循环将其取出并执行

具体的行为可以参考底部链接

对于 then 方法返回的 promise 它是没有 resolve 函数的，取而代之只要 then 中回调的代码执行完毕并获得同步返回值，这个 then 返回的 promise 就算被 resolve

同步返回值的意思换句话说，如果 then 中的回调返回了一个 promise，那么 then 返回的 promise 会等待这个 promise 被 resolve 后再 resolve（这句话有点像绕口令哈哈哈～）

