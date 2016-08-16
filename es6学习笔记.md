## 一.let 和 const 命令

### let

- 所声明变量只在 `let` 命令所在代码块内有效
- 不存在变量提升
- 暂时性死区（在代码块中，使用`let` 声明变量之前，变量不可用）
- 不允许在相同作用域内重复声明同一个变量

### 块级作用域

- 变量只在当前代码块内有效
- 作用域可以任意嵌套 `{{{{{{let a = 'shabi'}}}}}}`
- 函数本身的作用域在其所在的块级作用域内

```javascript
let f;
{
  let a = 'shabi';
  f = function(){
    return a;
  }
}
f(); // shabi
```

### const

- 声明常量，一旦声明，值不会改变，必须立刻初始化，不能留到以后再赋值
- 只在声明所在的块级作用域内有效
- 常量不指向数据而指向数据所在的地址，指向的数组、对象依然可更新

### 跨模块常量

```javascript
// constants.js 模块
export const A = 1;
//test1.js 模块
import * as constatns form './constants';
constants.A; //1
//test2.js 模块
import {A} form './constants'
A; //1
```

### 全局对象的属性

- `var` `function` 声明的全局变量依旧是全局对象的属性
- `let` `const` `class` 声明的全局变量不属于全局对象

----



## 二、变量的解构赋值

> 从数组或对象中提取值，对变量进行赋值 - Destructuring

### 数组的解构赋值

```javascript
let [a,b,c] = [1,2,3]
// 从数组中提取值，按照对应关系对变量赋值

let [foo,[[bar],baz]] = [1,[[2],3]];
foo //1
bar //2
baz //3

let [,,third] = [1,2,3];
third //3
```

- 解构不成功，变量为 `undefined`
- 只要数据具有 `iterator` 接口即可解构

#### 默认值

```javascript
let [x,y='0'] = ['a'];
x //a
y //0
```

- 默认值可以引用解构赋值的其他变量，但该变量必须已经声明

```javascript
let [x=1,y=x] = [2];
x //1
y //2

let [x=y,y=1] = []; // ReferenceError
```

### 对象的解构赋值

- 对象的属性没有顺序，变量名必须与属性名相同才能正确赋值
- 如果变量名与属性名不一致，须在声明时指定一个相同的属性名
- 默认值在属性名严格等于`undefined` 时才生效

> 对象的解构赋值内部机制为，先找到同名的属性，然后再赋值给对应的变量，真正的赋值的是后者而不是前者

```javascript
let {foo,bar,baz,foo:x} = {foo:'a',bar:'b'};
foo //a
bar //b
baz //undefined
x //a 被赋值的是x，而不是foo

let obj = {
  p:[
    'hello',
    {y:'world'}
  ]
}
let {p:[x,{y}]} = obj; //p是模式，不是变量，所以不会被赋值
x //hello
y //world

let node = {
  loc:{
    start:{
      line:1,
      column:5
    }
  }
}
let {loc:{start:{line}}} = node; //loc start都是模式
loc,start //undefined
line //1

let obj = {},arr = [];
({foo:obj.prop,bar:arr[0]}={foo:123,bar:true}) // es6会将{}识别为作用域块，用()解释为代码块
obj.prop //123
arr //[true]
```

### 字符串的解构赋值

- 被解构的字符串会被转换为一个类似数组的对象
- 类似数组的对象有`length` 属性，可以对这个属性解构赋值

```javascript
const [a,b,c,d,e] = 'hello';
a //h
let {lenght:len} = 'hello';
len //5
```

### 数值和布尔值的解构赋值

- 等号右边是数值或布尔值时，先会被转换成对象

```javascript
let {toString:s} = 123;
s === Number.prototype.toString //true 将s赋值为Number的toString方法
```

### 函数参数的解构赋值

- `functoin add({x=0,y=0}={}){}` 参数中变量x和变量y的默认值
- `function add({x,y} = {x:0,y:0})` 为函数参数指定默认值，而不是为x和y提供默认值

```javascript
function add([x,y]){ //[x,y]不是一个数组，而是通过解构得到的变量x,y
  return x+y;
}

[[1,2],[3,4]].map([a,b] => a+b)
//[3,7]

[1,undefined,3].map((x='yes') => x);
//[1,yes,3]
```

### 圆括号

- 解构赋值中不能使用圆括号的情况
  - 变量声明语句中，模式不能带有圆括号
  - 函数参数中不能带有圆括号（函数参数也属于变量声明）
  - 不能将整个模式或嵌套模式中的一层放在圆括号中
- 只有赋值语句的非模式部分可以使用圆括号

### 用途

#### 交换变量的值

```javascript
[x,y] = [y,x]
```

#### 从函数返回多个值

```javascript
function add(){
  return [1,2,3,4]
}
let [a,b,c,d] = add();

function ex(){
  return {
    foo:1,
    bar:2
  }
}
let {foo,bar} = ex();
```

#### 函数参数的定义

- 解构赋值可以方便的将一组参数与变量名对应

```javascript
function f([x,y,z]){}
f([1,2,3]) //有序值
function o({x,y,z}){}
f({x:3,z:1,y:0}) //无序值
```

#### 提取JSON数据

```javascript
var jsonDATA = {
  id:1,
  name:'shabi'
}
let {id,name} = jsonDATA;
id //1
name //shabi
```

#### 函数参数的默认值

```javascript
var add = function({a=1,b=2}={}){
  return a+b
}
add() //3
add(10,10) //3
add({a:20,b:20}) //40
```

#### 遍历Map结构

```javascript
var map = new Map();
map.set('key','value');
for(let [key,value] of map){ // [key]获取键名 [,value]获取值名
  key //key
  value //value
}
```

#### 输入模块的指定方法

```javascript
const {A,B} = require(constants)
```

---



## 三、字符串的扩展

### 字符的unicode表示法

- 将码点放入大括号就能正确解读字符 `\u{20BB7} = 𠮷` `\u{41}\u{42}\u{43} = abc`

### codePointAt()

- 返回32位的 UTF-16 字符的码点
- 可以检测字符是2字节字符还是4字节字符 `str.codePointAt(0) > 0xFFFF`

### String.fromCodePoint()

- 从码点返回字符 `String.fromCOdePoint(0x20BB7) = 吉`

### 字符串遍历器接口

- 字符串可以使用 `for-of` 循环遍历

### ~~at() (ES7)~~

- `'abc'.at(0) = a`

### normalize()

- 将字符的不同表现方法统一为同样的形式 （比如带有重音符号的字符）

### includes(), startsWith(), endsWith()

- `includes()` 表示是否找到了参数字符串
- `startsWith()` 表示是否以参数字符串开头
- `endsWith()` 表示是否以参数字符串结尾
- 第二个参数表示开始搜索的位置

### repeat()

- 返回新字符串，将原字符串重复n次

### ~~padStart(), padEnd() (ES7)~~

- 字符串补全，接收最小长度和用来补全的字符串

### 模板字符串

- 使用 **`** 号标识模板字符串，是增强版的字符串

```javascript
let name = 'shabi';
`hello ${name} \`\`` //hello shabi ``

`hello ${'da ' + name}` //hello da shabi
```

### 标签模板

- 将模板字符串跟在一个函数名后面，该函数将被调用来处理这个模板字符串
- 标签是一个函数，整个表达式返回的值就是标签函数处理整个模板字符串后的解构
- 标签函数接受到的第一个参数是模板字符串中没有变量替换的部分组成的数组
- 第一个参数还有一个`raw`属性，会将`\n` 识别为 `\` 和 `n` 两个字符，这样可以获得转义之前的原始模板

```javascript
let total = 30;
let msg = passthru`the total is ${total} (${total * 1.05} with tax)`;
function passthru(values){
  let i=0,result='';
  while(i<values.length){
    result += values[i++];
    if(i<arguments.length){
      result += arguments[i]
    }
  }
  return result;
}
//传入的参数为所有字符拼接成的数组the total is,(,with tax)三个部分
//拼接total变量到result
msg; //the total is 30 (31.5 with tax)
```

### String.raw()

- 返回一个 `\`都被转义的字符串
- 作为处理模板字符串的基本方法，会替换所有的变量并对 `\` 进行转义

---



## 四、正则的扩展

### RegExp构造函数

- RegExp构造函数允许接受正则表达式作为参数，返回一个原有正则表达式的拷贝 `var regex = new RegExp(/xyz/i)` 
- 如果 RegExp 构造函数接收第二个参数指定的修饰符，则会忽略原有的正则修饰符

### 字符串的正则方法

- 所有与正则有关的方法全部调用 RegExp 的实例方法
  - String.prototype.match 调用 RegExp.prototype[Symbol.match]
  - String.prototype.replace 调用 RegExp.prototype[Symbol.replace]
  - String.prototype.search 调用 RegExp.prototype[Symbol.search]
  - String.prototype.split 调用 RegExp.prototype[Symbol.split]

### u修饰符

- u修饰符表示unicode模式，用来处理 \uFFFF的unicode字符

#### .字符

- 对于码点大于0xFFFF的字符 `.` 字符不能识别，须配合u修饰符使用

```javascript
var s = '\u{20BB7}';
/^.$/.test(s); //false
/^.$/u.test(s); //true
```

#### unicode字符表示法

- 使用{}表示的unicode字符，必须使用u修饰符

#### 量词

- {}内配合u修饰符不会被识别为量词

```javascript
/𠮷{2}/.test('𠮷𠮷') //false
/𠮷{2}/u.test('𠮷𠮷') //true
```

#### 预定义模式

```javascript
//使用u修饰符和\s 预定义模式正确返回字符串长度
function codePointLength(text){
  var result = text.match(/[\s\S]/gu);
  return result ? result.length : 0
}
```

### y修饰符

> 沾连（sticky）修饰符

- 作用为全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始

```javascript
let s = 'aaa_aa_a';
let r1 = /a+/g;
let r2 = /a+/y;

r1.exec(s); //['aaa']
r2.exec(s); //['aaa']
//剩余字符串为 _aa_a

r1.exec(s); //['aa']
r2.exec(s); //null

let r3 = /_a+/y;
r3.exec(s) //['aaa_']
r3.exec(s) //['aa_']
```

### sticky属性

- 表示正则对象中是否使用了y修饰符

### flags属性

- 表示正则表达式的修饰符

### ~~RegExp.escape() (ES7)~~

- 字符串必须转义才能作为正则表达式

---



## 五、数值的扩展

### 二进制和八进制数值表示法

- 使用 `0b` 或 `0B` 表示二进制 `Number(0b111) = 7`
- 使用 `0o` 或 `00` 表示八进制 `Number(0010) = 8`

### Number.isFinite()和Number.isNaN()

- `Number.isFinite()` 检测是否为非无穷数
- 对非数值一律返回false，不做类型转换

### Number.parseInt() 、Number.parseFloat()

- 行为与 `parseInt` 和 `parseFloat` 一致，但移植到了Number对象上

### Number.isInteger()

- 判断是否为整数，`3 === 3.0`

### Number.EPSILON

- 一个极小的常量，为浮点数计算设置一个范围误差
- 浮点误差小于 `Number.EPSILON` 即可认为得到了争取的结果

```javascript
0.1 + 0.2 // 0.30000000000000004
function withinErrorMargin(left,right){
  return Math.abs(left - right) < Number.EPSILON
}
withinErrorMargin(0.1+0.2,0.3) //true
withinErrorMargin(0.1+0.2,0.4) //false
```

### 安全整数和Number.isSafeInteger()

- `Number.MAX_SAFE_INTEGER` 和 `Number.MIN_SAFE_INTEGER` 两个常量表示整数范围上下限
- `Number.isSafeInteger()` 判断一个整数是否落在安全范围内**（使用时须验证每一个参与运算的数值）**

### Math对象的扩展

- Math.trunc() 去除一个数的小数部分
- Math.sign() 判断一个数是正数、负数、0 ，正数返回+1，负数返回-1，0返回0，-0返回-0，其他值返回NaN
- Math.cbrt() 返回一个数的立方根
- Math.clz32() 返回一个数的32位无符号整数形式有多少个前导0
- Math.imul() 返回两个数以32位带符号整数形式相乘的结果
- Math.fround() 返回一个数的单精度浮点数形式
- Math.hypot() 返回所有参数的平方和的平方根
- Math.expm1(x) 返回e^x-1
- Math.log1p(x) 返回 \ln(1+x)
- Math.log10(x) 返回以10为底x的对数
- Math.log2(x) 返回以2为低x的对数

### 指数运算符 *（ES7，babel可用）*

- `2 ** 2 = 4` `a **= 2 //a =a * a` `b **= 3 //b = b*b*b`

---



## 六、数组的扩展

### Array.from()

- 用于将类似数组的对象和可遍历的对象转为真正的数组
  - Array-like object 必须拥有 `length` 属性
- 可以将部署了 `Iterator` 接口的数据结构转为数组
- 使用扩展运算符 `…` 也可以将某些数据结构转为数组
- 第二个参数用来对每个元素进行处理，类似`map` ，可用第一个参数的 `length` 属性指定第二个参数的运行次数
- 将字符串转为数组可以精确得到字符串长度

```javascript
let arrayLike = {0:'a',1:'b',2:'c',length:3}
let arr = Array.from(arrayLike); //[a,b,c]

let ps = document.querySelectorAll('p');
Array.from(ps).forEach(function(p){
  console.log(p);
})
[...doucment.querySelectorAll('p')]

function foo(){
  var args = Array.from(arguments);
  var args = [...arguments]
}
Array.from('hello𠮷'); //[h,e,l,l,o,𠮷].length = 6
let nameSet = new Set(['a','b']);
Array.from(nameSet); //[a,b]
Array.from([1,2,3],(x) => x * x); //[1,4,9]
Array.from({length:2},() => 'jack'); //[jack,jack]
```

### Array.of()

- 用于将一组值转为数组，不存在由于参数个数不同导致的重载，可代替 `Array()` 和 `new Array()`

```javascript
Array.of(); //[]
Array.of(3); //[3]
Array.of(2,3); //[2,3]
```

### copyWithin()

- 在当前数组内部将指定位置的成员复制（并覆盖）到其他位置 
- `Array.prototype.copyWithin(target, start=0, end=this.length)`
  - target ：从该位置开始替换数序（必须）
  - start ： 从该位置开始读取数据，默认为0，负数表示倒数（可选）
  - end ： 到该位置前停止读取数据，默认等于数组长度，负数表示倒数

```javascript
[1,2,3,4,5].copyWithin(0,3); //[4,5,3,4,5]
[1,2,3,4,5].copyWithin(0,3,4); //[4,2,3,4,5]
[1,2,3,4,5].copyWithin(0,-2,-1); //[4,2,3,4,5]
[].copyWithin.call({length:5,3:1},0,3); //{0: 1, 3: 1, length: 5}
```

### find() 和 findIndex()

- `find()` 找出第一个符合条件的数组成员，参数是一个回调函数，所有数组成员依次执行该回调函数，直到第一个为true的成员并返回
- `findIndex()` 找出第一个符合条件的成员的位置，没找到返回 -1
- 两个方法都可以发现 `NaN` 

```javascript
[1,6,3,4].find((n) => n%2 == 0); //6
[1,5,10,15].find(function(value,index,arr){
  return value > 10;
}) // 15

[1,5,10,15].findIndex(function(value,index,arr){
  return value > 10;
}) // 3
```

### fill()

- 使用指定值填充数组，可用于初始化空数组，会抹去数组中已有的元素
- 第二、三个参数表示填充的起始位置和结束位置

### entries()、keys()、 values()

- 都返回一个遍历器对象，可使用 `for-of` 遍历

```javascript
for (let index of ['a','b'].keys()){
  console.log(index)//0,1
}

for (let elem of ['a','b'].values()){
  console.log(elem)//0,1 need polyfill
}

for (let [index,elem] of ['a','b'].entries()){
  console.log(index,elem)
  //0 "a"
  //1 "b
}
```

### includes() *需要babel*

- 表示数组是否包含给的值，第二个参数表示开始搜索的位置，可以找到 `NaN` 

### 数组的空位

- 指数组某一位置没有任何值，es6中的方法不会忽略空位而将空位转成undefined

> 空位不是undefined，一个位置的值等于undefined但依然是有值的，空位没有任何值，es5与es6处理空位方式不同所以不要出现空位

### 数组推导 *ES7 需要babel*

- 直接通过现有数组生成新的数组
- `for-of` 语句需要在前面，返回表达式写在后面
- 可以代替 `map` 和 `filter` 方法
- 数组推导的方括号 `[]` 构成了一个单独的作用域

```javascript
let arr = [1,2,3,4];
let arr2 = [for (i of arr) i*2]; //[2,4,6,8]

let years = [2001,2002,2003,2004];
let y = [for (year in years) if(year > 2001) year]; //2002,2003,2004

[for (i of [1,2,3]) i*2];
//等同于
[1,2,3].map(function(i){return i*2});

[for (i of [1,2,3]) if (i < 3) i];
//等同于
[1,2,3].filter(function(i){return i<3});
```

---



## 七、函数的扩展

### 函数参数的默认值

- 直接写在参数后面可以为参数设定默认值 `function (x,y='haha')`
- 参数变量是默认声明的，不能再用 `let` `const` 声明

#### 与解构赋值默认值结合

- 函数参数默认值可以与解构赋值结合使用

```javascript
function foo({x,y=5}){
  console.log(x,y);
}
foo(); //报错 参数不是变量没有触发解构赋值
foo({}); //undefined 5
foo({x:1}); //1 5

function m1({x=0,y=0}={}){};  //参数的默认值是空对象，触发了解构赋值的默认值
function m2({x,y}={x:0,y:0}){}; //参数是一个有具体属性的对象，不会触发解构赋值默认值
```

#### 参数默认值的位置

- 定义了默认值的参数应该是尾参数，比较容易看出省略了哪些参数，且参数无法省略
- 传入undefined将触发默认值，null则不行

#### 函数的length

- 函数的 `length` 属性将返回没有指定默认值的参数的个数（`...` 参数不会记入）

#### 默认参数的作用域

- 如果参数的默认值是变量，该变量所处的作用域与其他变量一致，先是当前函数的作用域之后是全局

```javascript
var x = 1;
function f1(x,y=x){
  console.log(y)
}
f1(2); //2 参数y的默认值等于x，由于函数作用域内部变量x已经生成，所以y等于参数x而不是全局变量x

let xx = 1;
function f2(yy=xx){
  let xx = 2;
  console.log(yy);
}
f2(); //1 参数yy的默认值变量xx尚未在函数内部生成所以yy指向全局变量xx

function f3(yyy = xxx){
  let xxx = 2; //全局变量xxx不存在
  console.log(yyy);
}
f3(); //ReferenceError: xxx is not defined(…)
```

- 如果参数的默认值是一个函数，由于**函数的作用域是其声明时所在的作用域**，作为参数的函数的作用域就是不是接受参数的函数，而是全局作用域

```javascript
let foo = 'outer';
function bar(func = x=> foo){ //声明的func作用域是全局
  let foo = 'inner';
  console.log(func());
};
bar(); //outer
```

> 利用函数参数默认值可以设定某个参数不得省略（默认值等于抛出错误的方法）
>
> 默认值设置为undefined可以表明参数可忽略

### rest参数

- `...variable` 用于获取函数的多余参数，不需要使用`arguments` 对象，`rest` 参数搭配的变量是一个数组，该变量将多余的参数放入其中
- 所有数组的方法都可以用于rest参数
- rest参数只能用于最后一个参数，函数的`length` 属性不包括rest参数

```javascript
function add(...values){
  let sum = 0;
  for (let val of values){
    sum += val;
  }
  return sum;
}
add(2,3,4); //9
```

### 扩展运算符

- 扩展运算符是 `…` ，类似于 rest参数的逆运算，讲一个数组转为用逗号分隔的参数序列

```javascript
console.log(...[1,2,3]); //1 2 3
```

#### 代替数组的apply

- 不需要用 `apply` 方法将数组转为参数

```javascript
Math.max(...[23,2,4,12,34,4]); //34
let arr1 = [1,2,3];
let arr2 = [5,6,7];
arr1.push(...arr2)；

let d = new Date(...[2011,3,1]); //Fri Apr 01 2011 00:00:00 GMT+0800 (CST)
```

#### 扩展运算符的应用

##### 合并数组

`[1,2,...array]`

##### 与解构赋值结合

```javascript
const [first,...rest] = [1,2,3,4,5]; //只能放在最后一位
first; //1
rest; //[2,3,4,5]
```

##### 函数的返回值

- 将函数返回的数组或对象直接传入其他函数

##### 字符串

- 将字符串转为真正的数组，而且可以识别32位的unicode字符

```javascript
function length(str){
  return [...str].length;
}
length('𠮷'); //1

let arr = ['𠮷','𠮷a','𠮷𠮷'];
[...arr].reverse().join(''); //𠮷𠮷𠮷a𠮷

let str = '𠮷a𠮷𠮷aa𠮷𠮷';
[...str].reverse().join(''); //𠮷𠮷aa𠮷𠮷a𠮷
```

##### 将类似数组的对象转为数组

`[...document.querySelectorAll('div')]`

##### Map 、Set 、Generator函数

- 扩展运算符内部调用的是数据结构的 iterator 接口

```javascript
let map = new Map([
  [1,'a'],
  [2,'b']
])
[...map.keys()]; //[1,2]

let go = function* (){
  yield 1;
  yield 2;
}
[..go()]; //[1,2] 将内部遍历得到的值转为数组
```

### name属性

- 返回函数名，将匿名函数赋值给一个变量，会返回实际函数名
- `Function` 构造函数返回的函数实例，name为anonymous
- `bind` 返回的函数，name属性前会加上 `bound` 前缀

```javascript
function foo(){};
foo.bind({}).name; //bound foo
(function(){}.bind({})).name //bound
```

### 箭头函数

#### 基本用法

- 使用 `=>` 定义函数 `let f = v => v` 等同于 `function f(v){return v}`
- 如果箭头函数不需要参数或需要多个参数，可用`()` 代表参数部分 `let sum = (x,y,z) => x+y+z`
- 如果需要返回一个对象，需要用 `()`  `let getObj = id => ({id:1})`
- 与解构赋值结合 `let full = ({a,b}) => a+b` `full({a:'a',b:'b'})`
- 与rest参数结合 `let numbers = (...nums) => nums`

#### 注意事项

- 箭头函数内部的 `this` 对象是定义时所在的对象，而不是使用时所在的对象（箭头函数内部没有this对象，内部this就是外层代码块的this）
- 不可当做构造函数，不能使用 `new` 操作符
- 不可以使用 `arguments` 对象，可用rest参数代替，在内部使用 `arguments` 对象时，其实是外层函数的 `arguments` 对象
- 不可使用 `yield` 命令，不可作为Generator函数
- `bind()` `call()` `apply()` 方法无效

```javascript
function Timer(){
  this.seconds = 0;
  setInterval(() => this.seconds++,1000);
};
let timer = new Timer();
setTimeout(() => console.log(timer.seconds),3100); //3
```

#### 嵌套箭头函数

- 箭头函数内部可以继续使用箭头函数，实现函数嵌套

```javascript
let insert = (value) => ({into:(array) => ({after:(afterValue) => {
  array.splice(array.indexOf(afterValue)+1, 0 ,value);
  return array;
}})});
insert(2).into([1,3]).after(1); //[1, 2, 3]
```

### 函数绑定 *（ES7，需要babel）*

> 用来取代 `bind()` `call()` `apply()` 调用

- 使用 `::` 左边是对象，右边是函数，该运算符会自动将左边的对象作为上下文环境绑定到右边的函数上
- 如果 `::` 左边为空，右边为一个对象的方法，则等于将该方法绑定在该对象上
- 返回的对象还是原对象

```javascript
foo:bar;
//等同于
bar.bind(foo);

foo:bar(...arguments);
//等同于
bar.apply(foo,arguments);

const hasOwnProperty = Object.hasOwnProperty;
function hasOwn(obj,key){
  return obj::hasOwnProperty(key);
}

let method = ::obj.foo; //等同于 obj::obj.foo

document.querySelectorAll('div')::find('p')::html('hahaha');
//等同于ES5中
var _context;
(_context = (_context = document.querySelectorAll('div'), find).call(_context, 'p'), html).call(_context, 'hahaha');
```

### 尾调用优化

> 指函数的**最后一步**是调用另一个函数，不一定在函数尾部

```javascript
function f(x){
  return g(x);
};
```

#### 优化

- 不再用到外层函数的内部变量，内层函数的调用帧会取代外层函数的调用帧

#### 尾递归

- 尾调用自身
- ***只有严格模式才会启用尾递归优化***

```javascript
function f(n){
  if(n===1) return 1;
  return n * f(n-1); //没有尾调用优化，复杂度为O(n)
}
//改为尾递归
function f2(n,total){
  if(n===1) return 1;
  return f2(n-1,n*total)
}
```

#### 递归函数改写

- 把所有用到的内部函数改写成参数形式，可以使用<u>柯里化</u>将原本的函数改为只接受一个参数的函数，或者使用ES6中的函数参数默认值

### ~~函数参数的尾逗号 （ES7）~~

- 函数定义和调用时都不允许参数有尾逗号

---



## 八、对象的扩展

### 属性的简洁表示法

- 直接写入变量和函数作为对象的属性和方法
- 只写属性名不写属性值时，属性值等于属性名所代表的变量
- 属性的赋值器 `getter` 和取值器 `setter` 也可简写
- 简洁写法中的属性名永远是字符串

```javascript
let foo = 'bar';
let baz = {foo};
baz; //{foo:'bar'}

function f(x,y){
  return {x,y}; //等同于 return {x:x,y:y}
}
f(1,2); //Object {x: 1, y: 2}

let obj ={
  name:'shabi',
  hello(){
    console.log(this.name) //等同于 hello = function()
  },
  age, //等同于 age:age
  * m(){
    yield 1; //m() 是一个Generator函数
  },
  get get_name(){
    return this.name + '?'
  },
  set set_name(name){
    this.name = name;
  }
}

module.exports = {a,b,c}; //等同于 module.exports = {a:a,b:b,c:c}
```

### 属性名表达式

- 字面量定义对象时，可以把表达式放在方括号 `[]` 中
- 不能与属性间接表示法混用

```javascript
let str = 'hi';
let obj = {
  [str]:'hello',
  ['h'+'ello'](){
    console.log('haha')
  }
}

let foo = 'bar';
let baz = {[foo]:'abc'}; //Object {bar: "abc"}
```

### Object.is()

- 比较两个值是否严格相等 `Object.is(+0,-0) === false` `Object.is(NaN,NaN) === true`

### Object.assign()

- 将来源对象所有的可枚举属性（包括 `Symbol` ）复制（并覆盖）到目标对象
- 合并数组时，会将数组视为属性名为0,1,2的对象 `Object.assign([1,2,3],[4,5]) == [4,5,3]`

#### 为对象添加属性

```javascript
class Point{
  constructor(x,y){
    Object.assign(this,{x,y}); //将x、y属性添加到Point类的对象实例
  }
}
```

#### 为对象添加方法

```javascript
Object.assign(someClass.prototype,{
  method(){},
  method2(){}
})
```

#### 克隆对象

```javascript
function clone(obj){
  return Object.assign({},obj); //仅克隆原始对象自身的值，不能克隆它继承的值
  //可隆obj继承的属性
  let objProto = Object.getPrototypeOf(obj);
  return Object.assign(Object.create(objProto),obj);
  //简写为
  Object.assign({ __proto__: obj.__proto__ }, obj)
}
```

#### 合并多个对象

```javascript
const merge = (target,...sources) => Object.assign(target,...sources);
const mergeAsNew = (...sources) => Object.assign({},...sources);
```

#### 为属性指定默认值

```javascript
const DEFAULTS = {a:1,b:2};
let options = Object.assign({},DEFAULTS,option)
```

### 属性的可枚举性

>  实际操作中，尽量使用 `Object.keys()` 代替 `for-in` 循环

- 对象的每一个属性都有一个描述对象，用于控制该属性的行为
- `for-in` `Object.keys()` `JSON.stringify()` `Object.assign()` `Reflect.enumerate()`  方法会忽略描述对象的 `enumerable` 属性为false的属性

### 属性的遍历

- `for-in` 遍历对象自身的和继承的可枚举属性（不包含Symbol属性）
- `Object.keys()` 返回对象自身，不包含继承和Symbol的，所有可枚举属性的数组
- `Object.getOwnPropertyNames(obj)` 返回自身所有属性和不可枚举属性的数组，不包含Symbol属性
- `Object.getOwePropertySymbols(obj)` 返回自身所有Symbol属性的数组
- `Reflect.ownKeys(obj)` 返回自身、Symbol、无论是否枚举的全部属性
- `Reflect.enumerate(obj)` 返回Iterator对象，遍历对象自身的和继承的所有可枚举属性（不含Symbol，与`for-in` 一致）

### ~~_\_ proto\_\_ 属性~~

- 用来读取当前对象的 `prototype` 对象，不要使用

### Object.setPrototypeOf() 、 Object.getPrototypeOf()

- `Object.setPrototypeOf()` 用于设置一个对象的 `prototype` 对象
- `Object.getPrototypeOf()` 用于读取一个对象的 `prototype` 对象

```javascript
Object.setPrototypeOf(object,prototype);

let proto = {};
let obj = {x:10};
Object.setPrototypeOf(obj, proto);
proto.x = 30;
proto.y = 20;
obj.x; //10
obj.y; //20

function Func(){};
let func = new Func();
Object.getPrototypeOf(func) === Func.prototype; //true
```

### 对象扩展运算符 *ES7，需要Babel*

#### rest参数

- 将所有可遍历但尚未读取的属性，分配到指定的对象上
- 浅复制，只是引用不是副本，不会赋值继承自圆形的对象

```javascript
let {x,y,...z} = {x:1,y:2,a:3,b:4};
x //1
y //2
z //{a:3,b:4}
```

#### 扩展运算符

- 取出参数对象的所有可遍历属性，复制到当前对象中
- 扩展运算符遇到 `get` 取值函数时，`get` 是会执行的

```javascript
let z = {a:3,b:5};
let n = {...z};
n; //{a:3,b:5}

{...a} === Object.assign({},a);
{...a,...b} === Object.assign(a,b);
```

---



## 九、Symbol

- ES6中新的数据类型，表示独一无二的值，通过 `Symbol` 函数生成 `let n=Symbol('n'); typeof n==='symbol'`
- Symbol是原始类型的值，不是对象，**不能与其他值做运算**
- Symbol函数的参数只表示对当前Symbol值的描述，相同参数的Symbol返回的值不相等
- Symbol值可以显式转为字符串，不能转为数值 `Symbol('MySymbol').toString(); 'Symbol(MySymbol)'`

### 作为属性名的Symbol

- Symbol可以作为标识符用于对象的属性名，保证不会出现同名的属性
- Symbol作为对象属性名的时候不能使用 `.` 点运算符
- Symbol所谓属性名时，该属性是公开的不是私有属性
- Symbol可用于定义常量，保证常量的值都是不相等的

```javascript
let mySymbol = Symbol();
let o1 = {};
o1[mySymbol] = 'hello';
let o2 = {[mySymbol]:'hello'};
let o3 = {};
Object.defineProperty(o3,mySymbol,{value:'hello'});
```

实例：消除魔术字符串

> 在代码中多次出现，与代码形成强耦合的某一个具体的字符串或数值，良好的的代码应尽量消除魔术字符串
>
> 将常出现的字符串使用Symbol定义为一个常量即可

### Symbol属性名的遍历

- `Object.getOwnPropertySymbols(obj)` 方法获取指定对象的所有Symbol属性名，返回数组

### Symbol.for()、Symbol.keyFor()

- `Symbol.for()` 可以重新使用一个Symbol值，接受一个字符串参数，搜索以该字符串作为名称的Symbol值，如果有返回这个Symbol值，没有则使用该字符串新建一个Symbol值 `Symbol.for('ha') === Symbol.for('ha')`
- `Symbol.keyFor()` 返回一个已登记的Symbol类型值的key `Symbol.keyFor(Symbol.for('ha')) === 'ha'`
- `Symbol.for()` 为Symbol值登记的名字是全局环境，可跨 Iframe 或 worker 使用

