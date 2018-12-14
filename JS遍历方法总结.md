来来来，新鲜出炉的js遍历总结，比我菜的都看一下。保你戒掉循环，告别模棱两可，达到灵活操作数据的高潮。
### for循环
```javascript
const data = ["a", "b", "c", "d"];
for (let i = 0, len = data.length; i < len; i++) {
  console.log(i);// 0 1 2 3
  console.log(data[i]);// a b c d 
}
```
适用范围：Array、String    
点评：性能最高但不够优雅
### forEach
```javascript
const data = ["a", "b", "c", "d"];
data.forEach((v, i) => {
  console.log(v);// a b c d 
  console.log(i);// 0 1 2 3
})
```
适用范围：Array、Set、Map   
点评：无法中途跳出循环，break命令或return命令都不能奏效。
### for...in
```javascript
const data = { key1: "a", key2: "b", key3: "c" };
for (let key in data) {
  console.log(key); // key1 key2 key3 
  console.log(data[key]); // a b c   
}
const data = ["a", "b", "c", "d"];
for (let index in data) {
  console.log(index); // 0 1 2 3  
}
```
适用范围：Array、Object、String   
点评：会枚举原型属性，就是说会遍历来自继承对象的可枚举属性
### for...of
```javascript
const data = ["a", "b", "c", "d"];
for (let item of data) {
  console.log(item); // a b c d  
}
```
适用范围：Array、String、Set、Map   
点评：ES6新增的作为遍历所有数据结构的统一的方法。唯一的缺点也许就是兼容性了吧。
### Array.filter()
返回一个新数组，数组中的元素为原始数组中符合条件的所有项。
```javascript
//筛选出包含_mgt的项
const limits = ["member_mgt", "member_audit", "recommend_mgt", "dictionary_mgt"]
const result = limits.filter(item => {
  return item.indexOf("_mgt") > -1
})
console.log(result)// [ 'member_mgt', 'recommend_mgt', 'dictionary_mgt' ]
```
适用于想从源数据中筛选/提取出指定数据的业务场景。
### Array.map()
返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。
```javascript
//数组中所有值转大写
const limits = ["member_mgt", "member_audit", "recommend_mgt", "dictionary_mgt"]
const result = limits.map(item => {
  return item.toLocaleUpperCase()
})
console.log(result)// [ 'MEMBER_MGT','MEMBER_AUDIT','RECOMMEND_MGT','DICTIONARY_MGT' ]
```
适用于要对源数据进行加工或修改的业务场景
### Array.some()  
检测所有元素是否满足条件，并返回一个Boolean值。 如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测。如果没有满足条件的元素，则返回false。
```javascript
//判断是否有member_mgt的权限
const limits = ["member_mgt", "member_audit", "recommend_mgt", "dictionary_mgt"]
const result = limits.some(item => {
  return item === "member_mgt"
})
console.log(result)// true
//这个效果其实等同于limits.includes("member_mgt") 这里只是为了举例
```
### Array.every() 
检测所有元素是否满足条件，并返回一个Boolean值。 如果数组中检测到有一个元素不满足，则整个表达式返回 false ，且剩余的元素不会再进行检测。如果所有元素都满足条件，则返回 true。
```javascript
//判断数组中的值是否均为字符串
const limits = ["member_mgt", "member_audit", "recommend_mgt", "dictionary_mgt"]
const result = limits.every(item => {
  return typeof item === "string"
})
console.log(result)// true
```
### 总结发言
* 其实，支持对象遍历的方法只有for...in，不过可以通过Object.keys()、Object.values()、Object.entries()等方法将对象转成数组后再操作。   
* 其实，遍历的目的无外乎是对数据进行筛选、重组抑或是判断，只要熟练使用数组的那4个方法就能满足几乎所有的应用场景。消灭循环从我做起，加油！


