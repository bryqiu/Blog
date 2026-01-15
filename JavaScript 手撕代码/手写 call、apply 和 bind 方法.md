## this 指向

翻开《你不知道的 JavaScript》上篇关于 this 指向的章节，将 this 分为这些规则：

- 默认绑定，不带任何修饰的函数引用调用的，this 绑定到全局对象 `window`
- 隐式绑定，由上下文对象来调用函数，函数内的 this 绑定在上下文对象中
- 显示绑定，函数实例上的方法 `call`、`apply`、`bind` 强制绑定 this
- new 绑定，使用 `new` 关键字调用函数，`new` 内部操作会改变 this，参考之前文章：[new 操作符实现](https://github.com/bryqiu/Blog/blob/main/JavaScript%20%E6%89%8B%E6%92%95%E4%BB%A3%E7%A0%81/new%20%E6%93%8D%E4%BD%9C%E7%AC%A6%E5%AE%9E%E7%8E%B0.md)
- 箭头函数的 this，箭头函数没有自己的 this，而是根据外层作用域来决定 this

## 手写实现 `call`

`call()` 方法会以给定的 this 值和逐个提供的参数调用该函数，返回函数调用后的结果

手写 `call` 要明确一条核心思路：**在对象上下文里调用函数**，也就是隐式绑定，改变函数内的 this 指向

```js
/**
 * 手写实现 call
 * @params context - this 指向
 * @params args - 函数参数
 * @returns - 返回函数调用后的结果
 */
function myCall(context, ...args) {
  // 如果函数不在严格模式下，null 和 undefined 将被替换为全局对象，并且原始值将被转换为对象
  context =
    context === null || context === undefined
      ? window
      : Object(context);

  // Symbol 生成一个独一无二的 key，避免覆盖原有属性
  const key = Symbol("fn");

  // 这里的 this 指向调用 myCall 的函数，隐式上下文绑定
  context[key] = this;

  // 执行并获取结果
  const result = context[key](...args);

  // 删除在context生成的key，避免污染全局对象
  Reflect.deleteProperty(context, key);

  // 和原生 call 一样，返回函数调用后的结果
  return result;
}

Function.prototype.myCall = myCall;
```

使用下面这个例子测试：

```js
const obj = {
  name: "bryan",
};

function getName(text) {
  console.log(`${text} ${this.name}`);
}

getName.myCall(obj, "hey"); // hey bryan
```

## 手写实现 `apply`

`apply()` 方法会以给定的 this 值和一个数组（或类数组对象）作为参数调用该函数，返回函数调用后的结果

理解和能手写出上面的 `myCall`，其实 `apply` 就没什么好讲的了，不同点只是 `apply` 接收一个类数组作为参数

```js
/**
 * 手写实现 apply
 * @params context - this 指向
 * @params args - 函数参数
 * @returns - 返回函数调用后的结果
 */
function myApply(context, args) {
  context =
    context === null || context === undefined
      ? window
      : Object(context);

  const key = Symbol("fn");
  context[key] = this;
  const result = context[key](...args);
  Reflect.deleteProperty(context, key);
  return result;
}

Function.prototype.myApply = myApply;

const obj = {
  name: "bryan",
};

function getName(text) {
  console.log(`${text} ${this.name}`);
}

getName.myApply(obj, ["hey"]); // hey bryan
```

## 手写实现 `bind`

`bind()` 方法会返回一个函数，当调用这个函数时，会将 this 指向 `bind` 方法的第一个参数，后续参数会作为新函数的参数

- 要返回一个函数，供调用
- 传入两个参数：`context` 改变 this 指向的值，`args` 函数参数
- 偏函数应用，`args` 接收一部分参数，`innerArgs` 是返回函数传入的参数，固定住 `args` 参数，再接上 `innerArgs` 参数

```js
/**
 * 手写实现 bind
 * @params context - this 指向
 * @params args - 函数参数
 * @returns - 返回新函数
 */
function myBind(context, ...args) {
  context =
    context === null || context === undefined ? window : Object(context);

  // 这里的 this 指向调用 myBind 的函数本身，在下面的例子中是 getName 函数
  const self = this;

  return function (...innerArgs) {
    return self.apply(context, args.concat(innerArgs));
  };
}

Function.prototype.myBind = myBind;
```

用下面的例子测试一下：

```js
const obj = {
  name: "bryan",
};

function getName(text, text2) {
  console.log(`${text} ${this.name},${text2}`);
}
const bindFn = getName.myBind(obj, "hey");

console.log(bindFn("good morning")); // hey bryan,good morning
```

- `MDN - bind` 的描述中，如果使用 `new` 运算符构造绑定函数时，`bind` 方法的第一个参数会被忽略
- 此外，还要注意的是，构造函数生成的实例对象，应该继承 `bind` 方法的原型

> 在上面的例子是使用 `bindFn` 构造出的实例对象，要继承 `getName` 函数的原型对象

根据描述，我们实现第二版 `myBind`

```js
/**
 * 手写实现 bind
 * @params context - this 指向
 * @params args - 函数参数
 * @returns - 返回新函数
 */
function myBind(context, ...args) {
  if (typeof this !== "function") {
    throw new TypeError(
      "Function.prototype.bind - what is trying to be bound is not callable"
    );
  }

  context =
    context === null || context === undefined ? window : Object(context);

  // 这里的 this 指向调用 myBind 的函数本身，在上面的例子中是 getName 函数
  const self = this;

  const fBound = function (...innerArgs) {
    // fBound 函数作为构造函数时，这时候 this 指向实例对象
    // 如果 this 原型链上有 fBound 函数，说明是 `new fBound` 构造函数(通过 this instanceof fBound 来判断)
    // 这时候，我们需要将 this 指向新创建的实例对象，否则 this 指向 context
    return self.apply(
      this instanceof fBound ? this : context,
      args.concat(innerArgs)
    );
  };

  // 为了保持原型链的继承关系，我们需要将 fBound 函数的原型指向 self 函数的原型
  // tip：使用 Object.create 方法而不是直接 fBound.prototype = self.prototype，因为直接赋值会改变 fBound 函数的原型
  fBound.prototype = Object.create(self.prototype);

  return fBound;
}

Function.prototype.myBind = myBind;
```

使用一个例子来测试：

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
  console.log(`姓名 ${this.name}, 年龄 ${this.age}`);
}

Person.prototype.sayHi = function () {
  console.log(`Hi, I am ${this.name}`);
};

const obj = { name: "ContextObj" };

// 普通函数调用
const boundFn = Person.myBind(obj, "Alice");
console.log(boundFn(25)); // 姓名 Alice, 年龄 25

// 作为构造函数调用
const boundFn2 = new boundFn("Bob", 30);
boundFn2.sayHi(); // Hi, I am Bob
```

## 参考资料

- [MDN-Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
- [JavaScript 深入之 call 和 apply 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
- [MDN-Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [MDN-Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)