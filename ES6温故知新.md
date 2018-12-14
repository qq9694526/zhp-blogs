
## 原则
1、尽可能的精简描述
2、列举应用场景
3、优先级，常用性评价

## 一、什么是ES6？
ES6，全拼ECMAscript 6，顾明思议是版本号为6的javaScript标准，它于2015 年 6 月发布，亦称“ES2015”。   
但是由于该版本更新量较大，且随后的版本都是小幅修订。   所以ES6，一般是指2015 年 6 月发布的 ES2015 标准，但有时也是泛指5.1版本以后的“下一代 JavaScript 语言”。   
注：2016 年 6 月发布的ES6.1只新增了数组实例的includes方法和指数运算符。
## 二、变量的声明let和const
> ES5 只有两种声明变量的方法：var命令和function命令。ES6 除了添加let和const命令，后面章节还会提到，另外两种声明变量的方法：import命令和class命令。所以，ES6 一共有 6 种声明变量的方法。  

let声明一个变量只在当前作用域有效。
const声明一个变量只在当前作用域有效。
ES5 只有全局作用域和函数作用域，没有块级作用域，这里的let实际上为 JavaScript 新增了块级作用域   
ps: if判断、for循环的每一个循环都是独立的作用域 块级作用域
```javascript
//全局作用域window/global
const a="";
//函数作用域
function (){
    let a="";
}
//块级作用域
if(true){
  let a="";  
}
for (let i = 0; i < 3; i++) {//设置循环变量这部分也是一个单独的作用域
  let i = 'abc';//每一个循环 都是独立的作用域 块级作用域
}
```
与var的区别
1. 不存在变量提升（变量会提前声明随后赋值）。比如变量a，在声明之前使用，使用var声明，a===undifined，let直接报错。暂时性死区
2. 全局变量将逐步与顶层对象的属性脱钩。
3. 不允许在相同作用域内，重复声明同一个变量
const声明一个只读的常量，
const的作用域与let命令相同：只在声明所在的块级作用域内有效。
需要注意的是 这个常量并不说值不可改动。对于复合类型的数据，const 保存的只是一个指向实际数据的指针
，可以给对象添加属性，可以增删数组。
```javascript
const foo = {};
// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123
```
## 三、变量的解构赋值
ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。
## 四、字符串的扩展
> ES6 加强了对 Unicode 的支持，并且扩展了字符串对象。 

1. 新增了codePointAt和fromCodePoint方法，使支持32位字符与码点之间的转换。
* codePointAt方法返回的是码点的十进制值,如果想要十六进制的值，可以使用toString方法转换。
* String.fromCodePoint方法用于从码点返回对应字符,识别 32 位的 UTF-16 字符（Unicode 编号大于0xFFFF）.
2. ES6 为字符串添加了遍历器接口，使得字符串可以被for...of循环遍历。
3. 新增模板字符串``   
```javascript
`hello,${'world'}!`
```
4. 传统上，JavaScript只有indexOf方法，可以用来确定一个字符串是否包含在另一个字符串中。ES6 又提供了三种新方法。
* includes()：返回布尔值，表示是否找到了参数字符串。
* startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
* endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。
5. repeat方法返回一个新字符串，表示将原字符串重复n次。
6. [ES2017]字符串长度补全，padStart()用于头部补全，padEnd()用于尾部补全。
## 五、数值的扩展
> 由于 JavaScript 采用 IEEE 754 标准，数值存储为64位双精度格式，数值精度最多可以达到 53 个二进制位（1 个隐藏位与 52 个有效位）   

不知道为什么在知道“js算不准”后就对数值或者数字运算丧失了学习的欲望。我想大概和物理学家们刚知道“上帝是掷骰子”的时候是同样的感受吧？
和物理不太一样的是，现代物理学家已经“量子力学”真香了，而我怕是等不到“js运算”真香的那一天了。

关于此，我之前还特意开发了一个npm包：[accurate-math](You can now limit a repository's notifications exclusively to releases. Use it to follow your favorite repositories!) 专门用于解决js浮点数运算算不准这个问题。

数值部分就此略过，等王境泽喊我了再回来！！！


## 六、对象的扩展
1. 属性简写
```
const baz = {foo};
// 等同于
const baz = {foo: foo};
```
2. 属性名表达式
```
let propKey = 'foo';
//中括号里是表达式
let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};
```
3. 遍历对象的属性  
   for in
   Object.keys(obj)//返回键名的数组
   Object.values(obj)//返回值的数组
   Object.keys(obj)//返回键名的数组
   > 属性遍历的次序规则。
    -首先遍历所有数值键，按照数值升序排列。
    -其次遍历所有字符串键，按照加入时间升序排列。
    -最后遍历所有 Symbol 键，按照加入时间升序排列。
4. 扩展运算符   
   就像脱衣服，穿衣服。
   应用场景
   * 修改对象部分属性；合并对象
省略，方法的 name 属性，super 关键字   
5. 新增的方法
* Object.is()//判断两个值是否相等，解决==和===运算符的缺点
```
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```
* Object.assign(targetObj,...otherObjs)//合并对象的方法
* Object.keys(obj)//返回数组：对象所有键名的集合
* Object.values(obj)//返回数组：对象所有值的集合
* Object.entries(obj)//返回数组：对象键值对的集合，标准Map对象的数据结构
* Object.fromEntries(obj)//entries的已操作，常用于Map结构和对象互转


## 七、Symbol
js的第七种数据类型，表示一个独一无二的值，常被用于作为对象的键名防止属性覆盖。