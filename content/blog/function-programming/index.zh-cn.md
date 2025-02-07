---
title: '函数式编程'
date: 2024-11-02
description: " "
---

## 函数式编程两大基石
### 纯函数(Pure Function)

- 给定**相同输入**，永远返回**相同输出**
- **没有副作用**(不修改外部状态)
- 不依赖外部状态

> [!tip] 副作用 Side effect
> 在执行一个函数时，除了返回函数值外，还对调用函数产生了附加的影响，比如修改了全局变量、修改参数或改变外部的存储。

### 不可变性(Immutability)

- 这是一个数据处理的概念
- 确保数据创建后不被修改
- 任何修改都返回新的数据副本

---

让我们来看个具体的例子加深理解：

```js
// 原始数据
let users = [
	{ name: 'alice', age: 19 },
	{ name: 'bob', age: 17 },
	{ name: 'charlie', age: 20 },
]

// 非函数式写法
let result = []
for(let user of users) {
	if(user.age >= 18) {
		result.push({
			...user,
			name: user.name.charAt(0).toUpperCase() + user.name.slice(1)
		})
	}
}
```

现在用函数式的方式改写：
```js
// 1. 纯函数：检查是否成年
const isAduit = user => user.age >= 18

// 2. 纯函数：格式化名字（保持不可变性）
const formatName = user => ({ ...user, // 创建新对象而不是修改原对象
	name: user.name.charAt(0).toUpperCase() + user.name.slice(1)
});

// 3. 最终处理流程
const processUser = users => users.filter(isAduit).map(formatName)

// 使用
const result = processUsers(users);
console.log(result);
// 输出：
// [
//   { name: 'Alice', age: 19 },
//   { name: 'Charlie', age: 20 }
// ]
```

## 高阶函数(Higher-order Functions)

- 定义：**接收或返回函数**的函数
- 作用：提供了处理函数的抽象方法

### 函数组合(Function Composition)

- 定义：将多个函数连接起来，前一个函数的输出作为后一个函数的输入
- 本质：是高阶函数的一种应用

---

对于组合函数，同样也举个例子：

比如我们有一个处理字符串的任务：
1. 去除空格
2. 转为小写
3. 获取前5个字符

```js
// 首先我们有三个独立的纯函数
const removeSpaces = str => str.replace(/\s/g, '');
const toLowerCase = str => str.toLowerCase();
const take5 = str => str.slice(0, 5);

// 不用组合函数，我们需要这样嵌套调用
const input = "Hello World";
const result = take5(toLowerCase(removeSpaces(input)));
// "hello"
```

这种嵌套调用会有几个问题：
1. 可读性差
2. 不够灵活
3. 不容易复用

使用函数组合来改进：
```js
// 1. 创建一个函数组合
const compose = (...fns) => x => fns.reduceRight((val, fn) => fn(val), x)
// 或者从左到右的 pipe
const pipe = (...fns) => x => fns.reduce((val, fn) => fn(val), x)

// 2. 组合这些函数
const processString = compose(take5, toLowerCase, removeSpaces);
// 或用 pipe
const processString2 = pipe(removeSpaces, toLowerCase, take5);

// 3. 使用
const result = processString("Hello World")  // hello
```

#### 函数组合的优势
- 代码更清晰，从左到右或从右到左顺序执行
- 可以轻松调整函数顺序
- 组合后的函数可以复用
- 每个步骤都是独立的，容易测试和维护

### 柯里化(Currying)

- 定义：将多参数函数转换为单参数函数序列
- 本质：也是高阶函数的一种应用

---
下面我们通过对比来说明函数组合和柯里化的区别：
```js
// 1. 函数组合：多个函数组合成一个新函数
const compose = (...fns) => x => fns.reduceRight((val, fn) => fn(val), x);
const add1 = x => x + 1;
const multiply2 = x => x * 2;
const subtract3 = x => x - 3;
const composed = compose(subtract3, multiply2, add1);
composed(5); // ((5 + 1) * 2) - 3 = 9

// 2. 柯里化：将多参数函数转换为单参数函数序列
const curry = (fn) => {
	return function curried(...args) {
		if(args.length >= fn.length) {
			return fn.applay(this, args)
		}
		return (...args2) => curried.apply(this, args.concat(args2))
	}
}
// 使用示例
function add(a, b, c) {  // add.length 为 3
	return a + b + c
}
const curriedAdd = curry(add)
// 执行过程分析
curriedAdd(1)(2)(3)
// 第一次调用: args=[1], 1 < 3, 返回新函数
// 第二次调用: args=[1,2], 2 < 3, 返回新函数
// 第三次调用: args=[1,2,3], 3 >= 3, 执行 add(1,2,3)

curriedAdd(1, 2)(3)
// 第一次调用: args=[1,2], 2 < 3, 返回新函数
// 第二次调用: args=[1,2,3], 3 >= 3, 执行 add(1,2,3)

curriedAdd(1, 2, 3)
// 第一次调用: args=[1,2,3], 3 >= 3, 直接执行 add(1,2,3)
```

---

### `splice` 和 `toSpliced`

`splice` 是一个具有副作用的方法，因为它会直接修改原始数组。我们来详细分析：

```js
const arr = [1, 2, 3, 4];
arr.splice(1, 2); // 返回被删除的元素 [2, 3]
console.log(arr); // [1, 4] - 原数组被修改了！
```

从函数式编程的角度来看：

1. 违反了纯函数原则：
    - 修改了外部状态（原数组）
    - 有副作用
    - 相同输入可能产生不同输出（取决于数组当前状态）
2. 违反了不可变性原则：
    - 直接修改原始数据结构
    - 不返回新的数组副本

所以，**`splice` 是典型的命令式编程**，而不是函数式编程。

`toSpliced()` 是 JavaScript 数组的一个新方法，它是 `splice()` 的不可变版本。

```js
// 1. splice() - 有副作用，会修改原数组
const array = [1, 2, 3, 4];
array.splice(1, 2);         // 返回被删除的元素 [2, 3]
console.log(array);         // [1, 4] - 原数组被修改！

// 2. toSpliced() - 无副作用，返回新数组
const array2 = [1, 2, 3, 4];
const newArray = array2.toSpliced(1, 2);  // 返回新数组 [1, 4]
console.log(array2);        // [1, 2, 3, 4] - 原数组不变
console.log(newArray);      // [1, 4]
```

`toSpliced()` 符合函数式编程理念，因为：

- 不修改原数组（不可变性）
- 相同输入总是返回相同输出（纯函数）
- 没有副作用

`toSpliced()` 是 JS 数组新增的一系列不可变方法之一，其他还包括：

- `toReversed()`
- `toSorted()`
- `with()`

这些方法都遵循了函数式编程的原则