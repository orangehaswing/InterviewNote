# JavaScript

- 掌握JavaScript中是定义和初始化变量的方法
- 掌握JavaScript中的七种数据类型
- 掌握JavaScript中的作用域原则
- 理解程序中变量的概念
- 理解什么是作用域
- 理解为什么需要有作用域

# 1.变量

## 学习内容

- 简单来说，变量就是存储信息的容器。变量就相当与一个”盒子“，用来存储信息，比如：“Bob”，“true”，35等。同时，为了能够区分不同的盒子里存储的是什么信息，需要为每一个”盒子“起一个名字，称为“变量名”。当然，在“盒子”中存储的信息都有不同的数据类型，比如，若是存储了姓名“Bob”，那它就是字符串类型，若是存储了年龄`35`，那它就是数字类型，不同的语言会有不同的数据类型，JavaScript也有自己的数据类型，我们将会在下一节中讲述。变量在使用前通常有两个步骤：声明和初始化。声明就相当于拿了一个"盒子"说这个"盒子"就用来存储姓名了，即指定变量名；初始化相当于将信息放在这个"盒子"中存储，比如将“Bob”放在"盒子"中存储。

**JavaScript 变量**

- 在JavaScript中，变量是松散类型的，所谓松散类型就是可以用变量来保存任何类型的数据。换句哈说，每个变量仅仅是一个用户保存值的占位符而已。在JavaScript中定义变量使用`var`来声明一个变量，后面跟一个变量名，变量名必须是合法的标识符变量必须以字母开头变量也能以 $ 和 _ 符号开头（不过我们不推荐这么做）变量名称对大小写敏感（y 和 Y 是不同的变量）

```
  var message = 'Hello World';  // Right!
  var 2age = 30; // Wrong! 变量名不能以数字开头
```

**JavaScript 数据类型**

- 在上一面提到，JavaScript中的变量是松散类型的，但是松散类型不是说JavaScript中没有数据类型，而意思是：我们在用"盒子"来存储信息时，不必规定这个”盒子“这能存储数字类型的信息，而是数字，字符串，日期等什么类型的信息都可以存储，但是数据本身是由数据类型的。

```
var pi=3.14;
var name="Bill Gates";
var answer='Yes I am!';
```

**声明（创建） JavaScript 变量**

- 我们也可以像下面这样来声明一个变量：

```
  var information;
```

- 这行代码定义了一个名为`information`的变量，该变量可以用来存储任何值（像这样未经过初始化的变量，会被保存一个特殊的值——`undefined`）。
- 在JavaScript中当一个变量已经被声明和初始化之后，还可以对该变量的值进行修改，如下：

```
  var name = 'Bob'; 
  name = 'Mike';
```

- 可以看到在上述代码中的第二行将姓名改为`’Mike‘`。之前我们提到JavaScript是松散类型的，我们是否可以向下面这样修改变量呢？

```
  var name = 'Bob';
  name = 10; // 有效！但不推荐
```

- 在这个例子中，变量`name`一开始保存了一个字符串的值`’Bob‘`，然后该值又被一个数字值`10`取代。虽然这种方式在JavaScript中完全有效，但是我们不建议修改变量时修改所保存值的类型。
- 有时我们可能需要同时定义多个变量，比如要一起定义姓名，年龄，学号，我们可以用下面的方法来定义变量：

```
  var name = 'Bob'; 
  var age = 16; 
  var stuNo = 0012323; 
  // 也可以这样定义 
  var name = 'Bob', age = 16, stuNo = 0012323;
```

- 第一种方法我们分别定义了三个变量，通过对比上面两种方法可以发现第二种方法将分号改为逗号，只有一个`var`同时定义了三个变量。同样，由于JavaScript是松散类型的，因而使用不同类型初始化变量的操作可以放在一条语句中来完成。虽然代码里的换行和变量缩进不是必须的，但这样做可以提高代码的可读性。


- **Value 值为 undefined**

在计算机程序中，经常会声明无值的变量。未使用值来声明的变量，其值实际上是 undefined。

```
// 在执行过以下语句后，变量 name 的值将是 undefined：
var name;
```

- **重新声明 JavaScript 变量**

如果重新声明 JavaScript 变量，该变量的值不会丢失：

```
//在以下两条语句执行后，变量 name 的值依然是 "Jack"：
var name="Jack";
var name;
```

# 2.数据类型

## 学习内容

- 在上一节中我们提到每一种编程语言都有不同的数据类型，也就是我们的“盒子”里面存储的是字符串、还是数字等等。 
- JavaScript 是一种**弱类型**或者说**动态**语言。这意味着不用提前声明变量的类型，在程序运行过程中，类型会被自动确定。这也意味着可以使用同一个变量保存不同类型的数据：

```
  var foo = 42; // foo is a Number now 
  var foo = "bar"; // foo is a String now 
  var foo = true; // foo is a Boolean now
```

在 JavaScript 规范中，共定义了七种数据类型，分为 “基本类型” 和 “引用类型” 两大类，如下所示：

- 基本类型：String、Number、Boolean、Symbol、Undefined、Null
- 引用类型：Object

#### 1.字符串类型（String）

- JavaScript的字符串类型用于表示文本数据。在字符串中的每个元素占据了字符串的位置。第一个元素的索引为0，下一个是索引1，依此类推。字符串的长度是它的元素的数量。
- 在JavaScript中的字符串需要使用单引号`'**'`或双引号`"**"`括起来，表示该值是一个字符串。
- JavaScript 字符串是不可更改的。这意味着字符串一旦被创建，就不能被修改。但是，可以基于对原始字符串的操作来创建新的字符串。例如：
  1. 获取一个字符串的子串可通过选择个别字母或者使用`String.substr()`。
  2. 两个字符串的连接使用连接操作符 (`+`) 或者`String.concat()`。
- 符号类型（Symbol）：符号(Symbols)是ES6新定义的。符号类型是唯一的并且是不可修改的。

#### 2.Number

Number类型包含整数和浮点数（浮点数数值必须包含一个小数点，且小数点后面至少有一位数字）两种值

#### 3.布尔类型（Boolean）

布尔表示一个逻辑实体，意为真、假，可以有两个值：`true`和`false`。

#### 4.Symbol（了解）

Symbol 是 ES6 新增的一种原始数据类型，它的字面意思是：符号、标记。代表独一无二的值 。

#### 5.Undefined 和 Null

- Undefined 这个值表示变量不含有值。
- Null 类型只有一个值：`null`，表示空值，表示没有被呈现
- 可以通过将变量的值设置为 null 来清空变量。

```
// 值：undefined
var car="Volvo"; //值："Volvo"
var car=null; //值：null
```

#### 6.对象（Object）

- javascript 中的对象(物体)，和其它编程语言中的对象一样，可以比照现实生活中的对象(物体)来理解它。 javascript 中对象(物体)的概念可以比照着现实生活中实实在在的物体来理解。
  - 在javascript中，一个对象可以是一个单独的拥有属性和类型的实体。我们拿它和一个杯子做下类比。一个杯子是一个对象(物体)，拥有属性。杯子有颜色，图案，重量，由什么材质构成等等。同样，javascript对象也有属性来定义它的特征。
  - 对象可以通过`new`操作符后跟要创建的对象类型的名称来创建。而创建Object类型的示例并为其添加属性和（或）方法，就可以创建自定义对象，如下所示：

```
      var o = new Object();	
```

- 我们也可以通过下面的方式直接创建一个对象：

```
      var person = { name: 'Bob', age: 20, gender: 'male' };
```

- 上述对象就定义了一个名为’Bob‘，20岁，的男生。

#### 7.typeof操作符

- 由于JavaScript是松散类型的，因此需要有一种手段来检测给定变量的数据类型——`typeof`就是负责提供这方面信息的操作符。对一个值使用`typeof`操作符可能返回下列某个字符串：

```
    'undefined' —— 未定义 
    'boolean' —— 布尔值 
    'string' —— 字符串 
    'number' —— 数字值 
    'object' —— 对象或null 
    function —— 函数 
```

- 下面展示一下使用`typeof`的示例：

```
    var message = 'some string'; 
    alert(typeof message); // "string" 
    alert(typeof(message)); // "string" 
    alert(typeof 95); // number
```

- 在实际编程的过程中，可以用`typeof`判断任何一个变量的数据类型。

# 3.作用域

## 学习目标

- 理解什么是作用域
- 理解为什么需要有作用域
- 掌握JavaScript中的作用域原则

## 学习内容

- 作用域被用来描述在某个代码块可见的所有实体（或有效的所有标识符），更精准一点，叫做上下文。
- 那么为什么需要有作用域呢？
  - 最小访问原则
  - 通过限制变量的可见性，不允许代码中所有的东西在任意地方都可用的好处是什么？其中一个优势，是作用域为你的代码提供了一个安全层级。计算机安全中，有个常规的原则是：用户只能访问他们当前需要的东西。
- JavaScript的作用域链
  - 首先观看这段代码：

```
    <script type="text/javascript">
        var rain = 1;
        function rainman(){
            var man = 2;
            function inner(){
                var innerVar = 4;
                alert(rain);
            }
            inner();    //调用inner函数
        }
        rainman();    //调用rainman函数
    </script>
```

- 观察alert(rain);这句代码。JavaScript首先在inner函数中查找是否定义了变量rain，如果定义了则使用inner函数中的rain变量；如果inner函数中没有定义rain变量，JavaScript则会继续在rainman函数中查找是否定义了rain变量，在这段代码中rainman函数体内没有定义rain变量，则JavaScript引擎会继续向上（全局对象）查找是否定义了rain；在全局对象中我们定义了rain = 1，因此最终结果会弹出'1'。


- 作用域链：JavaScript需要查询一个变量x时，首先会查找作用域链的第一个对象，如果以第一个对象没有定义x变量，JavaScript会继续查找有没有定义x变量，如果第二个对象没有定义则会继续查找，以此类推。
  - 上面的代码涉及到了三个作用域链对象，依次是：inner、rainman、window。


- 函数体内部，局部变量的优先级比同名的全局变量高
  - 首先看如下代码示例：

```
    <script type="text/javascript">
        var rain = 1;    //定义全局变量 rain
        function check(){
            var rain = 100;    //定义局部变量rain
            alert( rain );       //这里会弹出 100
        }
        check();
        alert( rain );    //这里会弹出1
    </script>
```

- 没有块级作用域，只有函数作用域
  - 这一点也是JavaScript相比其它语言较灵活的部分。
  - 仔细观察下面的代码，你会发现变量i、j、k作用域是相同的，他们在整个rain函数体内都是全局的。

```
    <script type="text/javascript">
        function rainman(){
            // rainman函数体内存在三个局部变量 i j k
            var i = 0;
            if ( 1 ) {
                var j = 0;
                for(var k = 0; k < 3; k++) {
                    alert( k );    //分别弹出 0 1 2
                }
                alert( k );        //弹出3
            }
            alert( j );            //弹出0
        }
    </script>
```

- 函数中声明的变量在整个函数中都有定义
  - 观察一下代码示例

```
    <script type="text/javascript">
        function rain(){
            var x = 1;
            function man(){
                x = 100;
            }
            man();        //调用man
            alert( x );    //这里会弹出 100
        }
        rain();    //调用rain
    </script>
```

- 上面的代码说明了，变量x在整个rain函数体内都可以使用，并可以重新赋值。由于这条规则，会产生“匪夷所思”的结果，观察下面的代码。

```
    <script type="text/javascript">
        var x = 1;
        function rain(){
            alert( x );        //弹出 'undefined'，而不是1
            var x = 'rain-man';
            alert( x );        //弹出 'rain-man'
        }
        rain();
    </script>	
```

- 是由于在函数rain内局部变量x在整个函数体内都有定义（ var x= 'rain-man'，进行了声明），所以在整个rain函数体内隐藏了同名的全局变量x。这里之所以会弹出'undefined'是因为，第一个执行alert(x)时，局部变量x仍未被初始化。所以上面的rain函数等同于下面的函数：

```
    function rain(){
        var x;
        alert( x );
        x = 'rain-man';
        alert( x );
    }
```

- 未使用var、let、const关键字声明的变量都是全局变量

```
    <script type="text/javascript">
        function rain(){
            x = 100;    //声明了全局变量x并进行赋值
        }
        rain();
        alert( x );    //会弹出100
    </script>
```

- 这也是JavaScript新手常见的错误，无意之中留下的许多全局变量。
- 全局变量都是window对象的属性

```
    <script type="text/javascript">
        var x = 100 ;
        alert( window.x );//弹出100
        alert(x);
    </script>
```

- 等同于下面的代码

```
    <script type="text/javascript">
        window.x = 100;
        alert( window.x );
        alert(x)
    </script>	
```











