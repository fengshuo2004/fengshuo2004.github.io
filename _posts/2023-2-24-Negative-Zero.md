---
layout: post
title: 为什么JavaScript有两种零：+0 和 -0
---

你知道在JavaScript中有两个有效的值可以代表零吗？

```js
positiveZero = +0;
negativeZero = -0;
```

0 在数学上的意义是某种量的缺失，因此它带不带正负号也是没有意义的。+0 = -0 = 0。但依靠 [IEEE 754 标准](https://zhuanlan.zhihu.com/p/353013671)表达数值的计算机在这方面却有短板。

## 多数编程语言都有两种零！

IEEE 754 浮点数标准允许 0 带有正负号，也使得 +0 和 -0 可以同时存在。对应的，1 / +0 = +∞ 而 1 / -0 = -∞。这里的正无限和负无限可以理解为数轴两侧的“尽头”。

- +0 和 -0 可以看作两个大小为零但方向相反的向量
- 求一个函数于 +0 或 -0 的极限就是求这个函数从指定方向接近 0 的值（右极限、左极限）

这两种“零”也导致我们在上面的例子中得到了两种不同的“无限”。

## IEEE 754 为什么有两种零

无论一个数的大小是否为零，都有一比特表示它的正负。因此当一个负数的大小被设为零，又没有特意变号时，这个数就变成了 -0。

这又有什么关系呢？好吧，JavaScript实现的就是 IEEE 754 标准，本篇文章亦深挖其中的细节。

请注意，JavaScript（和大多数编程语言）的正零（+0）是默认的零值。

## JavaScript 中的零

### 表达

```js
let a = -0;
a; // -0

let b = +0;
b; // 0
```

### 如何得到 +0 和 -0

任何数学运算都可能根据操作数的值得到带符号的零（+0 或 -0）。

唯一的例外是 +0 和 -0 互相加减时，

- 两个 `-0` 相加，结果永远是 `-0`
- `-0` 减去 `0` 结果还是 `-0`

零的其他排列组合都会得到正零。另外需要注意的是，两个非零数相加减永远不会产生负零，即 -3 + 3 = 3 - 3 = +0。

更多例子参考以下代码：

```js
/*  加减运算  */
3 - 3;         //  0
-3 + 3;        //  0

/*  零零相加  */
-0 + -0;       // -0
-0 -  0;       // -0
0 -  0;        //  0
0 + -0;        //  0

/*  乘法运算  */
3 *  0;        //  0
3 * -0;        // -0

/*  除法运算  */
3 / Infinity;  //  0
-3 / Infinity; // -0

/*  取模运算  */
6 % 2;         //  0
-6 % 2;        // -0
```

### 字符串转换问题

JavaScript 有这么个瑕疵：它没有内置的方法可以把数型变量 `-0` 转换成字符串 `'-0'`。调用 [toString](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString) 只会得到 `'0'`。反过来，[parseInt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt) 和 [parseFloat](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseFloat) 却都能解析负零字符串。

其后果就是，除非通过特殊处理，数字 — 字符串这个转换过程是有信息丢失的。比如你（用 [JSON.stringify](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) 之类的方法）把一些数据序列化成字符串，POST 到服务器，过会儿再取回来，原先所有的负零都变成了正零。

```js
/* Number -> String */
let a = -0;
a.toString();      // '0'
String(a);         // '0'
JSON.stringify(a); // '0'
'' + a;            // '0'

/* String -> Number */
parseInt('-0');    // -0
parseFloat('-0');  // -0
Number('-0');      // -0
JSON.parse('-0');  // -0
+'-0';             // -0
```

### 区分 +0 和 -0

怎样才能区分这两种零呢？我们试试比大小：

```js
-0 === 0;  // true
-0 < 0;    // false
0 < -0;    // false
```

ES6 的 [Math.sign](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math/sign) 方法用来获取参数的符号，但是它帮不上咱们，因为传入 +0 或 -0 它会对应地返回 `0` 或 `-0`。搁这 原 地 T P 呢...

ES6 的 [Object.is](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 方法能做到：

```js
Object.is(0, -0); // false
```

ES5 就没有这样的“辅助函数”可以帮我们区分 +0 和 -0 了，需要自己写一个：

```js
function isNegativeZero(value) {
value = +value; // 强制类型转换至Number
if (value) {
    return false;
}
let infValue = 1 / value;
return infValue < 0;
}

isNegativeZero(0);    // false
isNegativeZero(-0);   // true
isNegativeZero('-0'); // true
```

## 两种零的用途

知道这些又有什么用呢？

1. 例一，假如你在搞机器学习，需要区分正负值来分叉。如果一个 -0 的结果被“纠正”成了一个正零，可能导致难以解决的分叉 bug。

2. 另一用例是开发编译器的场景。不懂行的程序员可能会让他的编译器优化掉如 `x * 0` 这种看上去“必定得0”的表达式。实际上它不应该被优化，因为 x 的值还是会影响结果的。将这种表达式直接替换为 0 将导致 bug。

3. 别忘了好多语言都支持 IEEE 754。我们以 Java 和 C# 为例：

    ```java
    System.out.print(1.0 / 0.0);  // Infinity
    System.out.print(1.0 / -0.0); // -Infinity
    ```

    ```c#
    Console.WriteLine(1.0 / 0.0); // Infinity
    Console.WriteLine(1.0 / -0.0); // -Infinity;
    ```

    试试在你熟悉的语言里复现一下！

## IEEE 和零有关的规范

IEEE 规范还导致了以下结果：

```js
Math.round(-0.4); // -0
Math.round(0.4);  //  0
 
Math.sqrt(-0);    // -0
Math.sqrt(0);     //  0
 
1 / -Infinity;    // -0
1 /  Infinity;    //  0
```

四舍五入 -0.4 得到 -0，是因为 -0 可以看作“一个数值从负方向接近 0 的极限”。

我倒是觉得平方根规则很奇怪。规范述：“除了 -0 的平方根为 -0，任何有效数值的平方根都应该带有正号”。顺便一说，IEEE 754 也是大多数语言中 `0.1 + 0.2 != 0.3` 的原因，这又是另一个故事了...