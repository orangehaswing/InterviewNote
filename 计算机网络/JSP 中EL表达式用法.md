# JSP 中EL表达式用法 #
# 与 [ ] 运算符

EL 提供 . 和 [ ] 两种运算符来导航数据。下列两者所代表的意思是一样的：

```
{sessionScope.user.sex}等于{sessionScope.user["sex"]}
```

. 和 [ ] 也可以同时混合使用，如下：

```
${sessionScope.shoppingCart[0].price}
```

回传结果为shoppingCart中第一项物品的价格。

以下两种情况，两者会有差异：

1. 当要存取的属性名称中包含一些特殊字符，如. 或 – 等并非字母或数字的符号，就一定要使用 [ ]，

   例如：

   ```
   ${user.My-Name}
   ```

   $上述是不正确的方式，应当改为：

   ```
   ${user["My-Name"]}
   ```

2. 我们来考虑下列情况：

   ```
   ${sessionScope.user[data]}
   ```

   此时，data 是一个变量，假若data的值为"sex"时，那上述的例子等于 

   ```
   ${sessionScope.user.sex}
   ```

   假若data 的值为"name"时，它就等于

   ```
   ${sessionScope.user.name}
   ```

   因此，如果要动态取值时，就可以用上述的方法来做，但 .  无法做到动态取值。

# 自动转变类型

EL 除了提供方便存取变量的语法之外，它另外一个方便的功能就是：自动转变类型，我们来看下面这个范例：

```
${param.count + 20}
```

假若窗体传来count的值为10时，那么上面的结果为30。之前没接触过JSP 的读者可能会认为上面的例子是理所当然的，
但是在JSP 1.2 之中不能这样做，原因是从窗体所传来的值，它们的类型一律是String，所以当你接收之后，必须再将它转为其他类型，
如：int、float 等等，然后才能执行一些数学运算，下面是之前的做法：

```
String str_count =request.getParameter("count");
int count =Integer.parseInt(str_count);
count = count + 20;
```

# EL 隐含对象

JSP有9个隐含对象，而EL也有自己的隐含对象。EL隐含对象总共有11 个

隐含对象                    类型                    说明
PageContext         javax.servlet.ServletContext    表示此JSP的PageContext
PageScope           java.util.Map               取得Page范围的属性名称所对应的值
RequestScope        java.util.Map            取得Request范围的属性名称所对应的值
sessionScope        java.util.Map            取得Session范围的属性名称所对应的值
applicationScope    java.util.Map        取得Application范围的属性名称所对应的值
param               java.util.Map       如同ServletRequest.getParameter(String name)。回传String类型的值

paramValues         java.util.Map   如同ServletRequest.getParameterValues(String name)。回传String[]类型的值

header              java.util.Map           如同ServletRequest.getHeader(String name)。回传String类型的值

headerValues        java.util.Map           如同ServletRequest.getHeaders(String name)。回传String[]类型的值

cookie              java.util.Map            如同HttpServletRequest.getCookies()
initParam           java.util.Map     如同ServletContext.getInitParameter(String name)。回传String类型的值

不过有一点要注意的是如果你要用EL输出一个常量的话，字符串要加双引号，不然的话EL会默认把你认为的常量当做一个变量来处理，

这时如果这个变量在4个声明范围不存在的话会输出空，如果存在则输出该变量的值。

# EL算术运算

    <%@ page contentType="text/html; charset=gb2312"%>
    <html>
    <head>
    <title>表达式语言 - 算术运算符</title>
    </head>
    <body>
    <h2>表达式语言 - 算术运算符</h2>
    <hr>
    <table border="1" bgcolor="aaaadd">
    <tr>
    <td><b>表达式语言</b></td>
    <td><b>计算结果</b></td>
    </tr>
    <!-- 直接输出常量 -->
    <tr>
    <td>\${1}</td>
    <td>${1}</td>
    </tr>
    <!-- 计算加法 -->
    <tr>
    <td>\${1.2 + 2.3}</td>
    <td>${1.2 + 2.3}</td>
    </tr>
    <!-- 计算加法 -->
    <tr>
    <td>\${1.2E4 + 1.4}</td>
    <td>${1.2E4 + 1.4}</td>
    </tr>
    <!-- 计算减法 -->
    <tr>
    <td>\${-4 - 2}</td>
    <td>${-4 - 2}</td>
    </tr>
    <!-- 计算乘法 -->
    <tr>
    <td>\${21 * 2}</td>
    <td>${21 * 2}</td>
    </tr>
    <!-- 计算除法 -->
    <tr>
    <td>\${3/4}</td>
    <td>${3/4}</td>
    </tr>
    <!-- 计算除法 -->
    <tr>
    <td>\${3 div 4}</td>
    <td>${3 div 4}</td>
    </tr>
    <!-- 计算除法 -->
    <tr>
    <td>\${3/0}</td>
    <td>${3/0}</td>
    </tr>
    <!-- 计算求余 -->
    <tr>
    <td>\${10%4}</td>
    <td>${10%4}</td>
    </tr>
    <!-- 计算求余 -->
    <tr>
    <td>\${10 mod 4}</td>
    <td>${10 mod 4}</td>
    </tr>
    <!-- 计算三目运算符 -->
    <tr>
    <td>\${(1==2) ? 3 : 4}</td>
    <td>${(1==2) ? 3 : 4}</td>
    </tr>
    </table>
    </body>
    </html>
 上面页面中示范了表达式语言所支持的加、减、乘、除、求余等算术运算符的功能，读者可能也发现了表达式语言还支持div、mod等运算符。而且表达式语言把所有数值都当成浮点数处理，所以3/0的实质是3.0/0.0，得到结果应该是Infinity。

如果需要在支持表达式语言的页面中正常输出“$”符号，则在“$”符号前加转义字符“\”，否则系统以为“$”是表达式语言的特殊标记。

# EL关系运算符

关系运算符  说明    范例                结果

== 或 eq    等于    ${5==5}或${5eq5}    true
!= 或 ne    不等于  ${5!=5}或${5ne5}    false
< 或 lt     小于    ${3<5}或${3lt5}     true
> 或 gt     大于    ${3>5}或{3gt5}      false
> <= 或 le    小于等于${3<=5}或${3le5}    true
> = 或 ge    大于等于5}或${3ge5}         false

表达式语言不仅可在数字与数字之间比较，还可在字符与字符之间比较，字符串的比较是根据其对应UNICODE值来比较大小的。



# EL逻辑运算符

逻辑运算符     范例                         结果
&&或and        交集${A && B}或${A and B}    true/false
||或or         并集${A || B}或${A or B}     true/false
!或not         非${! A }或${not A}          true/false
Empty          运算符

Empty 运算符主要用来判断值是否为空（NULL,空字符串，空集合）。

# 条件运算符

${ A ? B : C}


tags：HTML