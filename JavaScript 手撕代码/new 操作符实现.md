## new 操作符实现

> 前置知识：理解原型与原型链、构造函数...

翻开《JavaScript 高级程序设计》第四版中关于原型的章节，

文中写到当以 `new` 关键字调用函数时，会执行以下操作：

1. 在内存中创建一个新对象
2. 这个新对象内部的 `[[Prototype]]` 特性被赋值为构造函数的 prototyoe 属性
3. 构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）
4. 执行构造函数内部的代码（给新对象添加属性）
5. 如果构造函数返回非空对象，返回这个对象；否则，返回新创建的对象


按照上面的思路，我们可以模拟写一个自己的 `new` 操作符 

```js
function myNew(constructor, ...args) {
  // 1.在内存中创建一个新对象
  // 2.这个新对象内部的 `[[Prototype]]` 特性被赋值为构造函数的 prototype 属性
  const obj = Object.create(constructor.prototype);

  // 3. 构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）
  // 4. 执行构造函数内部的代码（给新对象添加属性）
  const result = constructor.apply(obj, args);

  // 5. 如果构造函数返回非空对象，返回这个对象；否则，返回新创建的对象
  return (result !== null && typeof result === 'object') ? result : obj;
}
```

解析：

使用 `Object.create` 来实现第一步、第二步，这个方法会创建一个新对象，并且将新对象的原型指针指向传入的对象的原型

![image](https://github.com/user-attachments/assets/d8f5871d-0763-4129-99de-a923172523c2)


在第三步和第四步，使用了函数上的实例方法 `Function.protytype.apply`

这个方法会改变函数内部的 `this` 指向，并且将剩余的参数(`arguments`)作为数组传入，然后调用该函数



## 手写的效果

在谷歌浏览器 - F12 开发者工具上测试效果

![image](https://github.com/user-attachments/assets/b8987251-e9c3-4d56-8e9a-a11dd42f811b)


## 总结

实例对象拥有构造函数的全部属性和方法，并且原型指针指向构造函数的原型，都是 new 操作符在背后做的工作：

1. 在内存中创建一个新对象
2. 这个新对象内部的 `[[Prototype]]` 特性被赋值为构造函数的 prototyoe 属性
3. 构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）
4. 执行构造函数内部的代码（给新对象添加属性）
5. 如果构造函数返回非空对象，返回这个对象；否则，返回新创建的对象

记住，手写实现 new 操作符，本质是为了实践理解 new 操作符的背后原理

## 参考资料
- [MDN - Object.create](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN - new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
- 《JavaScript 高级程序设计》第四版