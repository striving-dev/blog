---
title: TS中不识别自己封装的类型判断
date: 2020-12-11 15:04:28
tags: 踩坑记录
author: 李京阳
index_img: assert/ts.png
---
在大型项目中我们更愿意使用`ts`取代`js`，其中一个主要原因是`ts`能够进行静态类型检查，提高代码质量。

<!-- more -->

比如当我们对一个对象进行`push`操作时，`ts`会先判断当前对象是否是一个数组。就像这样：

![image-20201211141434434](https://cdn.ljybill.com/uPic/image-20201211141434434.png)

这样能够比较安全的操作一些参数，然而我们可能进行一些类型校验的封装，比如下面的代码：

```typescript
function isArray(arr): boolean {
    return Array.isArray(arr);
}

function isFunction(fn: any): boolean {
    return typeof fn === 'function';
}
```

然而，如果这样封装之后，`ts`并不买账你的类型判断：

![image-20201211141925858](https://cdn.ljybill.com/uPic/image-20201211141925858.png)

这种情况下怎么办呢？

---

## 解决方案

有两种解决方案：

1. 一种是强行断言

```typescript
function foo(arg: Array<number> | undefined) {
    arg.push(1);
    if (isArray(arg)) {
        (arg as Array<number>).push(1);
    }
    if (Array.isArray(arg)) {
        arg.push(1);
    }
}
```

这并不是一种优雅的选择。

2. **修改`isFunction`等判断类型函数的返回值**

```typescript
function isArray(arr): arr is Array<any> {
    return Array.isArray(arr);
}

function isFunction(fn: any): fn is Function {
    return typeof fn === 'function';
}

function foo(arg: Array<number> | undefined) {
    arg.push(1);         // TS2532: Object is possibly 'undefined'.
    if (isArray(arg)) {
        arg.push(1);
    }
    if (Array.isArray(arg)) {
        arg.push(1);
    }
}
```

至此，问题解决。

## 备注

但是ts默认是不校验`null`和`undefined`的。除非我们修改`ts.config.json`中的`strictNullChecks`字段。`"strictNullChecks": true`。建议开启此配置项，能够进一步提高代码的严谨性。