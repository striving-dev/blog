---
title: JSON.stringify和JSON.parse
date: 2020-12-21 10:34:28
tags: JS基础
author: simple
index_img: assert/JSONstringify.jpeg
---

#### JSON

JSON的全称是`JavaScript Object Notation`，意思是JavaScript对象表示法，它是一种基于文本，独立于语言的轻量级数据交换格式

##### JSON 格式

JSON 有两种表示法， 对象和数组，两种表示法可以相互嵌套组合， 和 JavaScript 中的语法基本一致， 但有一些必须遵守的格式

- 关键字必须是字符串， 值可以是字符串、 数值、 Boolean、null、 对象或数组
- 字符串必须用双引号括起来
- JSON 中不可以有注释



javaScript 中提供了两个处理 JSON 的方法， 分别是

`JSON.stringify`: 将对象处理成 JSON字符串

`JSON.parse`: 将 JSON 字符串转化成对象

这两个方法平时开发中使用率还是很高的， 比如将对象转为字符串存入 `storage`,  或者深度复制一个对象

通常用法如下

```javascript
const xiaoming = {
    name: '小明',
    age: 14,
    skills: ['JavaScript', 'Java', 'Python', 'Lisp']
};
const s = JSON.stringify(xiaoming);
console.log(s)
// '{"name":"小明","age":14,"skills":["JavaScript","Java","Python","Lisp"]}'
const so = JSON.parse(s);
console.log(so);
/*
{
    name: '小明',
    age: 14,
    skills: ['JavaScript', 'Java', 'Python', 'Lisp']
}
*/
```

但是这两个方法并非这么简单， 我们平时使用过程中很容易忽略了它的高级用法， 我们一个一个的看

#### JSON.stringify

先开看看它的语法

```javascript
JSON.stringify(value[, replacer [, space]])
```

是的， 你没看错，是不是很惊讶， 它居然有三个参数，如果你知道这个知识点， 那么你可以不用往下看了， 你很棒！

那么我们看看这三个参数到底是干什么， 它到底有何神通？

- value: 这个我们一直在用， 没什么可介绍的， 它就是我们要转化成 JSON 字符串的对象
- replacer: 可选值， 函数或者数组， 用于序列化过程中， 处理每个属性
- space: 可选值 字符串或者数字， 用于美化输出



光看文字说明可能不大容易理解， 我们举几个🌰

```javascript
// 我们的目标对象
const xiaoming = {
    name: '小明',
    age: 14,
    skills: ['JavaScript', 'Java', 'Python', 'Lisp'],
	  friends: [{names: 'xiaohong'}]
};
```

```javascript
// 第二个参数是数组
const temp1 = JSON.stringify(xiaoming, ['name', 'age', 'friends']);
console.log(temp1);
// '{"name":"小明","age":14,"friends":[{}]}'
```

发现了什么， 得到的 JSON 字符串 temp1 的属性值是经过第二个参数数组过滤后的， 而且第二数组的属性平铺的， 不考虑目标对象的深度，如果第二个参数是` ['name', 'age', 'friends', 'names']`, 那么最终得到的值就是 `'{"name":"小明","age":14,"friends":[{"names": "xiaohong"}]}'`

```javascript
// 第二个参数是函数
const temp2 = JSON.stringify(xiaoming, function(k, v) { console.log(k, v); return v; });
console.log('---------------------------------------分割线')
console.log(temp2);
/**
"" '{"name":"小明","age":14,"skills":["JavaScript","Java","Python","Lisp"],"friends":[{"names":"xiaohong"}]}'
name 小明
age 14
skills ["JavaScript", "Java", "Python", "Lisp"]
0 JavaScript
1 Java
2 Python
3 Lisp
friends [{names: "xiaohong"}]
0 {names: "xiaohong"}
names xiaohong
---------------------------------------分割线
'{"name":"小明","age":14,"skills":["JavaScript","Java","Python","Lisp"],"friends":[{"names":"xiaohong"}]}'
*/
```

分割线以上即第二个参数打印出来的， 对象的每个属性都会递归的去调用这个函数， 函数有两个参数， 第一个参数是 k, 代表对象的属性， 第二参数是 v，代表相应的属性值， 通过这个函数， 我们就可以对每个属性进行操作、过滤等。

```javascript
// 第三个参数是 字符串
const temp3 = JSON.stringify(xiaoming, null, 'aaaa');
console.log(temp3);
/**
"{
aaaa"name": "小明",
aaaa"age": 14,
aaaa"skills": [
aaaaaaaa"JavaScript",
aaaaaaaa"Java",
aaaaaaaa"Python",
aaaaaaaa"Lisp"
aaaa],
aaaa"friends": [
aaaaaaaa{
aaaaaaaaaaaa"names": "xiaohong"
aaaaaaaa}
aaaa]
}"
*/

// 第三个参数是 数字
const temp4 = JSON.stringify(xiaoming, null, 4);
console.log(temp4);
/**
"{
    "name": "小明",
    "age": 14,
    "skills": [
        "JavaScript",
        "Java",
        "Python",
        "Lisp"
    ],
    "friends": [
        {
            "names": "xiaohong"
        }
    ]
}"
*/
```

`JSON.stringify` 的第三个参数用于美化输出， 如果是字符串， 那么缩进的地方都会被字符串填满， 如 temp3 打印的那样。 如果是数字（num）， 那么缩进的地方都会被 num 个空格填满。

`JSON.stringify` 的三个参数都介绍完了， 再介绍几个需要注意地方

1. `JSON.stringify` 不仅可以处理对象， 也可以处理其他类型

   ```javascript
   JSON.stringify(true);                      // "true"
   JSON.stringify("foo");                     // ""foo""
   JSON.stringify(1);                         // "1"
   JSON.stringify(null);                      // "null"
   ```

2. 某些特定于 JavaScript 的对象属性会被 `JSON.stringify` 跳过

   - 函数属性（方法）
   - Symbol 类型的属性
   - 存储 undefined 的属性
   - 不可枚举的属性

   ```javascript
   const user = {
     sayHi() { // 被忽略
       alert("Hello");
     },
     [Symbol("id")]: 123, // 被忽略
     something: undefined, // 被忽略
     ...Object.create(
           null, 
           { 
               x: { value: 'x', enumerable: false } // 被忽略
           }
       )
   };
   console.log(JSON.stringify(user)); // {}
   ```

3. `JSON.stringify`第二个参数`replacer`如果是函数的话， 会根据对象的键值对递归的去调用， 但是第一个调用很特别，  `(key, value)` 对的键是空的， 并且该值是整个目标对象。 这个理念是为了给 `replacer` 提供尽可能多的功能： 如果有必要， 它有机会分析并替换、跳过整个对象

4. `JSON.stringify` 转换的对象中不可以有循环引用



##### toJSON 方法

像`toStirng`进行字符串转换， 对象也可以提供 `toJSON` 方法来进行 JSON 转换。如果可用，`JSON.stringify` 会自动调用它

```javascript
let meetup = {
  title: "Conference",
  date: new Date(Date.UTC(2020, 0, 1)),
};

alert( JSON.stringify(meetup) );
/*
  {
    "title":"Conference",
    "date":"2020-01-01T00:00:00.000Z",  // (1)
  }
*/
```

在这儿我们可以看到 `date`  变成了一个字符串。这是因为所有日期都有一个内置的 `toJSON` 方法来返回这种类型的字符串

我们也可以为对象 xiaoming 添加一个自定义的 `toJSON`, (xiaoming 今天的出场率有点高呀， ^_^)

```javascript
var xiaoming = {
    name: '小明',
    age: 14,
    skills: ['JavaScript', 'Java', 'Python', 'Lisp'],
    toJSON() {
      return {
        ...this,
        hobby: '读书'
      };
    }
};
alert(JSON.stringify(xiaoming));
// '{"name":"小明","age":14,"skills":["JavaScript","Java","Python","Lisp"],"hobby":"读书"}'
```

这样， 在返回的字符串中 xiaoming 就会多一个 `hobby: '读书'`



#### JSON.parse

要解码 JSON 字符串， 我们需要另一个方法 `JSON.parse`

语法

```javascript
JSON.parse(str, [reviver]);
```

- str

  要解析的 JSON 字符串。

- reviver

  可选的函数 function(key,value)，该函数将为每个 `(key, value)` 对调用，并可以对值进行转换。

基本用法如下

```javascript
let xiaoming = '{"name":"小明","age":14,"friends":[{"names":"xiaohong"}]}';
xiaoming = JSON.parse(xiaoming);
alert(xiaoming.name);
```



##### 使用 reviver

假如我们拿到一个字符串， 他是这样的

```javascript
let str = '{"title":"Conference","date":"2020-11-30T12:00:00.000Z"}';
```

现在我们需要对它进行反序列， 把它转换回 JavaScript 对象。

那么我们可以使用 `JSON.parse` 来完成

```javascript
let str = '{"title":"Conference","date":"2020-11-30T12:00:00.000Z"}';
let obj = JSON.parse(str);
alert(obj.date.getDate()); // Error: obj.date.getDate is not a function
```

报错了， 转换以后的 date 字段是个字符串， 而不是 Date 对象。 那么我们要怎么做呢？

这是 `JSON.parse` 第二个参数 reviver 就派上用场了， 我们可以这么处理。

```javascript
let str = '{"title":"Conference","date":"2020-11-30T12:00:00.000Z"}';
let obj = JSON.parse(str, function(k, v) {
  if (k === 'date') {
    return new Date(v);
  }
  return v;
});
alert(obj.date.getDate()); // 30 
```

##### 注意

- `JSON.parse` 转换的字符串必须符合 JSON 格式， 否则会报错

`JSON.stringify` 和 `JSON.parse` 的用法就介绍完了， 那它们有什么用呢？ 举几个工作中用到的例子吧。

1. 将属性值中可以转成数字的字符串转成数字

   ```javascript
   JSON.stringify(
     obj, // 要转化的对象
     function replacer(key, value) { // 转换函数
       if (!isNaN(value)) { // 判断是否可以转成数字
         return +value; // 那么返回转成数字的值
       }
       return value; // 否则返回原值
     },
     4
   );
   ```

2. 利用 `toJSON` 返回自己想要的对象， 属性顺序按自己的意愿排列

   ```javascript
   const obj = {
     a: '1',
     b: '2',
     c: '3'
   }
   Object.defineProperty(obj, "toJSON", {
     value: function() {
       return {
         c: this.c,
         b: this.b,
         a: this.a
       };
     },
     configurable: false,
     enumerable: false, // 不可枚举
     writable: false
   });
   
   JSON.stringify(obj, null, 4);
   // 输出的字符串就是 '{"c": "3", "b": "2", "a": "1"}'
   ```

这里只是举了几个简单的例子，要想让这两个方法在工作中发挥出更大的作用， 还需要您自己亲自去使用， 去发掘。