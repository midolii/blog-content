---
title: JavaScript的原型
published: 2026-07-14
description: "什么是JavaScript的原型"
image: ""
tags: [JavaScript]
category: "学习"
draft: false
---

## \_\_proto__ 和 prototype 的关系

```js
function Person() {}
const p1 = new Person();

console.log(Person.prototype.constructor === Person); // true
console.log(p1.__proto__ === Person.prototype); // true
```

## 我继承谁？- \_\_proto__ ?

1. **\_\_proto__** 是对象实例上的一个属性，指向了对象的 **构造函数.prototype**
2. 注意：`function Person() {}` 中，Person 既是函数又是对象，所以它既有 **\_\_proto__**（指向 Function.prototype），
   也有 prototype（留给它的实例继承）。

## 谁继承我？- prototype ?

1. 构造函数上的一个特殊属性，引擎会自动为构造函数添加 **prototype** 和 **\_\_proto__** 的逻辑

```js
function Person() {}
Person.prototype = { constructor: Person };
Person.__proto__ = Function.prototype;
```

2. 所有函数的 **\_\_proto__** 都指向 **Function.prototype**，
3. **Function** 本身也不例外。
4. **Function.prototype** 是普通对象，它的 **\_\_proto__** 才指向 **Object.prototype**。

## 速记口诀

1. **\_\_proto__** 是 “我继承谁”，**prototype** 是 “我的实例从我这里继承什么”。
2. 每个对象都有 **\_\_proto__**（指向它的原型）
3. 只有函数才有 **prototype**（将来被它的实例继承）

| 类型     | \_\_proto__ | prototype | 举例                   |
| -------- | ----------- | --------- | ---------------------- |
| 普通对象 | ✅          | ❌        | `const obj = {}`       |
| 普通函数 | ✅          | ✅        | `function Person() {}` |

### 代码示例

```js
// 定义一个构造函数
function Dog(name) {
  this.name = name;
}
Dog.prototype.bark = function () {
  console.log(this.name + " 汪汪!");
};

// 创建一个实例
const wangcai = new Dog("旺财");

// ----------------------------------------------
// === Dog 是函数，函数既有 __proto__ 也有 prototype ===

// Dog.__proto__ — "Dog 这个函数自己继承谁？"
// 答：Dog 是函数，所以它继承 Function.prototype
console.log(Dog.__proto__ === Function.prototype); // true

// Dog.prototype — "将来 new Dog() 创建的实例，继承谁？"
// 答：Dog.prototype 这个对象
console.log(typeof Dog.prototype); // 'object'
console.log(Dog.prototype.bark); // [Function: bark]

// ----------------------------------------------
// === wangcai 是普通对象，只有 __proto__，没有 prototype ===

// wangcai.__proto__ — "wangcai 继承谁？"
console.log(wangcai.__proto__ === Dog.prototype); // true

// wangcai.prototype — "wangcai 有 prototype 吗？"
console.log(wangcai.prototype); // undefined  ← 普通对象没有！
```

## new 到底做了什么

1. 创建了一个空对象
2. 将空对象的 **\_\_proto__** 指向构造函数的prototype
3. 执行构造函数方法，同时重新将this指定为第一步创建的空对象，传入参数
4. 构造函数内部执行方法，会把内部的属性 `this.xxxxxx = "xxxxxx"` 改为 `第一步创建的空对象.xxxxxx = "xxxxxx"`
5. 如果构造函数返回了一个对象，则 new 返回那个对象；否则返回第一步创建的对象。

### 代码示例

```js
function _new(Func, ...args) {
  const obj = {};
  obj.__proto__ = Func.prototype; // 必须，new的精髓，如果不执行，Func上面的共享方法丢失，无法访问
  const newObj = Func.apply(obj, args);
  return newObj instanceof Object ? newObj : obj;
}
```

## 手写instanceof

```js
function _instanceof(obj, Func) {
  // 右侧必须是可调用的函数
  if (typeof Func !== "function") {
    throw new TypeError(`Right-hand side of 'instanceof' is not callable`);
  }
  // 原始类型直接返回 false（instanceof 对原始类型也返回 false）
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) {
    return false;
  }
  let proto = Object.getPrototypeOf(obj);
  while (proto !== null) {
    if (proto === Func.prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
  return false;
}
```

## 复习

### 问题1

```js
function Parent() {}
function Child() {}
Child.prototype = new Parent();

const c = new Child();
```

**提问：** `c.constructor === Parent` 是 true 还是 false？为什么？如果想让 `c.constructor === Child`，怎么修复？

**我的回答：** true;因为Child的constructor在Child的prototype上，他的prototype被修改为了new Parent()的一个实例对象，所以constructor得继续往上找，一直找到 `new Parent()` 实例对象链上的constructor，即 `Parent.prototype.constructor`

> 回答**正确**，`new Parent()` 实例自身没有 constructor 属性，所以沿 **\_\_proto__** 找到 `Parent.prototype.constructor`，即 `Parent`

### 问题2

```js
const obj = Object.create(null);
```

**提问：** `obj instanceof Object` 返回什么？`obj.__proto__` 是什么？`Object.getPrototypeOf(obj)` 返回什么？——说说为什么。`

**我的回答：** `false;null;null`

> 回答**错误**，正确答案应该为false;undefined;null，因为__proto__是Object.prototype上的一个getter，Object.create(null)创建的就是一个null，null不继承Object.prototype，所以没有这个getter

### 问题3

用时 3 分钟，手写一个 _instanceof。要求：

- 右侧不是函数时抛 TypeError
- 左侧是 null 或原始类型时返回 false
- 用 Object.getPrototypeOf 而不是 **\_\_proto__**

```js
function _instanceof(obj, Func) {
  if (typeof Func !== "function") {
    throw new TypeError("Right-hand side of instanceof is not callable");
  }
  // 注意 "hello"、123、true、Symbol() 这些原始类型在 instanceof 下也返回 false，所以需要添加非object的判断
  if (obj === null || (typeof obj !== "object" && typeof obj !== "function")) {
    return false;
  }
  let proto = Object.getPrototypeOf(obj);
  while (proto !== null) {
    if (proto === Func.prototype) return true;
    proto = Object.getPrototypeOf(proto); // ← 注意不要用错变量，不是参数的obj，否则会死循环
  }
  return false;
}
```
