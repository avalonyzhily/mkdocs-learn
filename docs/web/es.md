#### 扩展运算符 '...'
1. 扩展运算符（ spread ）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。
2. 替换apply方法(貌似需要首参数为null)

#### 扩展运算符的应用
1. 合并数组:
```
var arr1 = ['a', 'b'];  
var arr2 = ['c'];  
var arr3 = ['d', 'e'];  
// ES5 的合并数组  
arr1.concat(arr2, arr3);  
// ES6 的合并数组  
[...arr1, ...arr2, ...arr3]  
```
2. 与解构赋值结合：
```
// ES5  
a = list[0], rest = list.slice(1)  
// ES6  
[a, ...rest] = list

ps:
const [first, ...rest] = [];  
first // undefined  
rest // []:
```
3. 函数的返回值,JavaScript 的函数只能返回一个值，如果需要返回多个值，只能返回数组或对象。扩展运算符提供了解决这个问题的一种变通方法。
```
var dateFields = readDateFields(database);  
//dateFields是一个数组
var d = new Date(...dateFields);  
```
4. 实现了 Iterator 接口的对象
任何 Iterator 接口的对象，都可以用扩展运算符转为真正的数组。
5. Map 和 Set 结构， Generator 函数
扩展运算符内部调用的是数据结构的 Iterator 接口，因此只要具有 Iterator 接口的对象，都可以使用扩展运算符，比如 Map 结构。
```
let map = new Map([  
[1, 'one'],  
[2, 'two'],  
[3, 'three'],  
]);  
let arr = [...map.keys()]; // [1, 2, 3]  

var go = function*(){  
yield 1;  
yield 2;  
yield 3;  
};  
[...go()] // [1, 2, 3] 
```
6. 注意没有实现iterator接口的对象不能使用扩展运算符。

#### Immutable Data
Immutable.js库,Immutable Data 就是一旦创建，就不能再更改的数据。对Immutable 对象进行修改、添加或删除操作，都会返回一个新的 Immutable 对象。