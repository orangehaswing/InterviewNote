# **Matlab基本语法**



## 常用的工具箱

------

|                                          |                                          |
| ---------------------------------------- | ---------------------------------------- |
| Matlab Main Toolbox——matlab主工具箱          | Control System Toolbox——控制系统工具箱          |
| Communication Toolbox——通讯工具箱             | Financial Toolbox——财政金融工具箱               |
| System Identification Toolbox——系统辨识工具箱   | Fuzzy Logic Toolbox——模糊逻辑工具箱             |
| Higher-Order Spectral Analysis Toolbox——高阶谱分析工具箱 | Image Processing Toolbox——图象处理工具箱        |
| computer vision system toolbox——计算机视觉工具箱 | LMI Control Toolbox——线性矩阵不等式工具箱          |
| Model predictive Control Toolbox——模型预测控制工具箱 | μ-Analysis and Synthesis Toolbox——μ分析工具箱 |
| Neural Network Toolbox——神经网络工具箱          | Optimization Toolbox——优化工具箱              |
| Partial Differential Toolbox——偏微分方程工具箱   | Robust Control Toolbox——鲁棒控制工具箱          |
| Signal Processing Toolbox——信号处理工具箱       | Spline Toolbox——样条工具箱                    |
| Statistics Toolbox——统计工具箱                | Symbolic Math Toolbox——符号数学工具箱           |
| Simulink Toolbox——动态仿真工具箱                | Wavelet Toolbox——小波工具箱                   |
| DSP system toolbox——DSP处理工具箱             |                                          |



## 常用的运算符和特殊字符

MATLAB常用的运算符和特殊字符如下表所示：

| 运算符     | 目的               |
| ------- | ---------------- |
| **+**   | 加；加法运算符          |
| **-**   | 减；减法运算符          |
| *****   | 标量和矩阵乘法运算符       |
| **.\*** | 数组乘法运算符          |
| **^**   | 标量和矩阵求幂运算符       |
| **.^**  | 数组求幂运算符          |
| **\**   | 矩阵左除             |
| **/**   | 矩阵右除             |
| **.\**  | 阵列左除             |
| **./**  | 阵列右除             |
| **:**   | 向量生成；子阵提取        |
| **( )** | 下标运算；参数定义        |
| **[ ]** | 矩阵生成             |
| **.**   | 点乘运算，常与其他运算符联合使用 |
| **…**   | 续行标志；行连续运算符      |
| **,**   | 分行符（该行结果不显示）     |
| **;**   | 语句结束；分行符（该行结果显示） |
| **%**   | 注释标志             |
| **_**   | 引用符号和转置运算符       |
| **._**  | 非共轭转置运算符         |
| **=**   | 赋值运算符            |



## 创建向量

创建行向量括在方括号中的元素的集合，用空格或逗号分隔的元素。

例如，

```
r = [7 8 9 10 11]
```

创建列向量通过内附组方括号中的元素，使用分号（;)分隔的元素。

```
c = [7;  8;  9;  10; 11]
```



## 运算符

| 运算符     | 描述                                       |
| ------- | ---------------------------------------- |
| **+**   | 加法或一元加号。A + B将A和B。 A和B必须具有相同的尺寸，除非一个人是一个标量。一个标量，可以被添加到任何大小的矩阵。 |
| **-**   | 减法或一元减号。A - B，减去B从A和B必须具有相同的大小，除非是一个标量。可以从任意大小的矩阵中减去一个标量。 |
| *****   | 矩阵乘法；是一个更精确的矩阵A和B的线性代数积，矩阵乘法对于非纯量A和B，列一个数必须等于B.标量可以乘以一个任意大小的矩阵的行数。 |
| **.\*** | 数组的乘法；A.*B是数组A和B的元素积，A和B必须具有相同的大小，除非A、B中有一个是标量。 |
| **/**   | 斜线或矩阵右除法；B/A与B * inv（A）大致相同。更确切地说： B/A = (A'B')' |
| **./**  | 矩阵右除法；矩阵A与矩阵B相应元素相除（A、B为同纬度的矩阵）          |
| **.\**  | 反斜杠或矩阵左除；如果A是一个方阵，AB是大致相同的INV（A）* B，除非它是以不同的方式计算。如果A是一个n*n的矩阵，B是一个n组成的列向量，或是由若干这样的列的矩阵，则X = AB 是方程 AX = B ，如果A严重缩小或者几乎为单数，则显示警告消息。 |
| **.**   | 数组左除法；A. B是元素B（i，j）/A（i，j）的矩阵。A和B必须具有相同的大小，除非其中一个是标量。 |
| **^**   | 矩阵的幂。X^P是X到幂P，如果p是标量；如果p是一个整数，则通过重复平方计算功率。如果整数为负数，X首先反转。对P值的计算，涉及到特征值和特征向量，即如果[ D ] = V，EIG（x），那么X^P = V * D.^P / V。 |
| **.^**  | A.^B：A的每个元素的B次幂（A、B为同纬度的矩阵）              |
| **'**   | 矩阵的转置；A'是复数矩阵A的线性代数转置，这是复共轭转置。           |
| **.'**  | 数组的转置；A'是数组A的转置，对于复数矩阵，这不涉及共轭。           |



## 绘图命令

MATLAB提供了大量的命令绘制图表。下表列出了一些常用的命令绘制：

| 命令        | 作用/目的         |
| --------- | ------------- |
| axis      | 人功选择坐标轴尺寸     |
| fplot     | 智能绘图功能        |
| grid      | 显示网格线         |
| plot      | 生成XY图         |
| print     | 打印或绘图到文件      |
| title     | 把文字置于顶部       |
| xlabel    | 将文本标签添加到x轴    |
| ylabel    | 将文本标签添加到y轴    |
| axes      | 创建轴对象         |
| close     | 关闭当前的绘图       |
| close all | 关闭所有绘图        |
| figure    | 打开一个新的图形窗口    |
| gtext     | 通过鼠标在指定位置放注文  |
| hold      | 保持当前图形        |
| legend    | 鼠标放置图例        |
| refresh   | 重新绘制当前图形窗口    |
| set       | 指定对象的属性，如轴    |
| subplot   | 在子窗口中创建图      |
| text      | 在图上做标记        |
| bar       | 创建条形图         |
| loglog    | 创建双对数图        |
| polar     | 创建极坐标图像       |
| semilogx  | 创建半对数图（对数横坐标） |
| semilogy  | 创建半对数图（对数纵坐标） |
| stairs    | 创建阶梯图         |
| stem      | 创建针状图         |



## 数据类型转换

------

MATLAB提供了各种用于将一种数据类型转换为另一种数据类型的函数。 下表显示了数据类型转换函数：

| 函数               | 描述说明                    |
| ---------------- | ----------------------- |
| `char`           | 转换为字符数组(字符串)            |
| `int2str`        | 将整数数据转换为字符串             |
| `mat2str`        | 将矩阵转换为字符串               |
| `num2str`        | 将数字转换为字符串               |
| `str2double`     | 将字符串转换为双精度值             |
| `str2num`        | 将字符串转换为数字               |
| `native2unicode` | 将数字字节转换为Unicode字符       |
| `unicode2native` | 将Unicode字符转换为数字字节       |
| `base2dec`       | 将基数N字符串转换为十进制数          |
| `bin2dec`        | 将二进制数字串转换为十进制数          |
| `dec2base`       | 将十进制转换为字符串中的N数字         |
| `dec2bin`        | 将十进制转换为字符串中的二进制数        |
| `dec2hex`        | 将十进制转换为十六进制数字           |
| `hex2dec`        | 将十六进制数字字符串转换为十进制数       |
| `hex2num`        | 将十六进制数字字符串转换为双精度数字      |
| `num2hex`        | 将单数转换为IEEE十六进制字符串       |
| `cell2mat`       | 将单元格数组转换为数组             |
| `cell2struct`    | 将单元格数组转换为结构数组           |
| `cellstr`        | 从字符数组创建字符串数组            |
| `mat2cell`       | 将数组转换为具有潜在不同大小的单元格的单元阵列 |
| `num2cell`       | 将数组转换为具有一致大小的单元格的单元阵列   |
| `struct2cell`    | 将结构转换为单元格数组             |



**if... elseif...else...end **语法：

详细例子如下：

在MATLAB中建立一个脚本文件，并输入下述代码：

```
a = 100;
   if a == 10 
       fprintf('Value of a is 10' );
    elseif( a == 20 )
       fprintf('Value of a is 20' );
    elseif a == 30 
       fprintf('Value of a is 30' );
   else
       fprintf('None of the values are matching');
   fprintf('Exact value of a is: %d', a );
   end
```



switch 语句的语法如下：

```
grade = 'B';
   switch(grade)
   case 'A' 
      fprintf('Excellent!' );
   case 'B' 
       fprintf('Well done' );
   case 'C' 
      fprintf('Well done' );
   case 'D'
      fprintf('You passed' );
   case 'F' 
     fprintf('Better try again' );
     
   otherwise
     fprintf('Invalid grade' );
   end
```



while循环的语法如下：

```
a = 10;
while( a < 20 )
  fprintf('value of a: %d', a);
  a = a + 1;
end
```



 for循环的语法如下：

for 循环的值有下述三种形式之一：

| 格式                    | 描述                                       |
| --------------------- | ---------------------------------------- |
| *initval:endval*      | 将索引变量从初始到终值递增1，并重复执行程序语句，直到索引值大于终值。      |
| *initval:step:endval* | 按每次迭代中的值步骤递增索引, 或在步骤为负值时递减。              |
| *valArray*            | 在每个迭代* valArrayon *数组的后续列中创建列向量索引。例如, 在第一次迭代中, index = valArray (:, 1)，循环执行最大 n 次, 其中 n 是 *valArray *的列数，由 numel (valArray, 1,:) 给出。输入 *valArray* 可以是任何 MATLAB 数据类型, 包括字符串、单元格数组或结构。 |

```
for a = 10:20 
  fprintf('value of a: %d', a);
end
```

```
for a = 1.0: -0.1: 0.0
   disp(a)
end
```

```
for a = [24,18,17,23,28]
   disp(a)
end
```



## 向量

当引用一个冒号，一个向量，其例如为v（:)，该载体上的所有组件的被列出。

例如：

```
v = [ 1; 2; 3; 4; 5; 6];	% creating a column vector of 6 elements
v(:)
```



### 向量的模

向量 v 中的元素 v1, v2, v3, …, vn，下式给出其幅度：

|v| = √(v12 + v22 + v32 + … + vn2)

MATLAB中需要采按照下述步骤进行向量的模的计算：

1. 采取的矢量及自身的积，使用数组相乘（*）。这将产生一个向量sv，其元素是向量的元素的平方和V.

   sv = v.*v;

2. 使用求和函数得到 v。这也被称为矢量的点积向量的元素的平方的总和V.

   dp= sum(sv);

3. 使用sqrt函数得到的总和的平方根，这也是该矢量的大小V.

   mag = sqrt(s);

详细例子

在MATLAB中建立一个脚本文件，代码如下：

```
v = [1: 2: 20];
sv = v.* v;     %the vector with elements 
                % as square of v's elements
dp = sum(sv);    % sum of squares -- the dot product
mag = sqrt(dp);  % magnitude
disp('Magnitude:'); 
disp(mag);
```



### 等差元素向量

要建立一个矢量 v 带的第一个元素 f，最后一个元素 l 和元素之间的区别是任何真正的数 n，可以这样写：

```
v = [f : n : l]
```

详细例子

在MATLAB中建立一个脚本文件，代码如下：

```
v = [1: 2: 20];
sqv = v.^2;
disp(v);disp(sqv);
```



### 引用一个矩阵的元素

如果要引用 mth 行和 nth 列的一个元素，写法如下：

```
mx(m, n);
```

### 引用m列中的所有元素，我们A型（m）

接下来我们要从矩阵 a 的第4行的元素开始建立一个列向量 v ：

```
a = [ 1 2 3 4 5; 2 3 4 5 6; 3 4 5 6 7; 4 5 6 7 8];
v = a(:,4)
```

当然也可以选择第 n 列的 m 个元素，对于这一点，写法如下：

```
a(:,m:n)
```



## 单元阵列

单元阵列的阵列中每个单元格可以存储不同的维度和数据类型的数组的索引单元格。

单元格函数用于建立一个单元阵列。

单元格函数的语法如下：

```
C = cell(dim)
C = cell(dim1,...,dimN)
D = cell(obj)
```

注意

- C 是单元阵列；
- dim 是一个标量整数或整数向量，指定单元格阵列C的尺寸；
- dim1, ... , dimN 是标量整数指定尺寸的C；
- obj 是以下内容之一
  - Java 数组或对象
  - .NET阵列 System.String 类型或 System.Object

详细例子

在MATLAB中建立一个脚本文件，输入下述代码：

```
c = cell(2, 5);
c = {'Red', 'Blue', 'Green', 'Yellow', 'White'; 1 2 3 4 5}
```

运行该文件，显示以下结果：

```
c = 
    'Red'    'Blue'    'Green'    'Yellow'    'White'
    [  1]    [   2]    [    3]    [     4]    [    5]
```



## 单元格引用

```
c = {'Red', 'Blue', 'Green', 'Yellow', 'White'; 1 2 3 4 5};
c(1:2,1:2)
```

```
c = {'Red', 'Blue', 'Green', 'Yellow', 'White'; 1 2 3 4 5};
c{1, 2:4}
```



## 冒号符号

MATLAB 中可以使用 “**:**” 来建立矢量、下标数组和指定的迭代，是最有用的MATLAB运算符之一。

下述例子建立了一个包括 1~10 的一个行向量：

```
1:10
```

如果想指定以外的一个增量值，例如：

```
100: -5: 50
```



可以使用冒号 “:” 运算符建立矢量指数来选择行、列或数组中的元素。

下表描述了其用于此目的（让我们有一个矩阵A）：

| 格式             | 目的                                       |
| -------------- | ---------------------------------------- |
| **A(:,j)**     | 是A的第j列                                   |
| **A(i,:)**     | 是A的第j行                                   |
| **A(:,:)**     | 是等效的二维数组；对于矩阵，这与A相同                      |
| **A(j:k)**     | 是A（j），A（j + 1），...，A（k）                  |
| **A(:,j:k)**   | 是 A(:,j)， A(:,j+1)，...，A(:,k)            |
| **A(:,:,k)**   | 是三维数组A的第k页                               |
| **A(i,j,k,:)** | 是四维数组A中的矢量；矢量包括A（i，j，k，1），A（i，j，k，2），A（i，j，k，3）等 |
| **A(:)**       | 是 A 的所有要素，被视为单列；在赋值语句的左侧，A（:) 填充A，保留以前的形状；在这种情况下，右侧必须包含与A相同数量的元素。 |

详细例子

在MATLAB中建立一个脚本文件，并输入下述代码：

```
A = [1 2 3 4; 4 5 6 7; 7 8 9 10]
A(:,2)      % second column of A
A(:,2:3)    % second and third column of A
A(2:3,2:3)  % second and third rows and second and third columns
```



## 创建矩阵

矩阵是一个二维数字阵列。

在MATLAB中，创建一个矩阵每行输入空格或逗号分隔的元素序列，最后一排被划定一个分号。

例如，下面创建了一个3×3的矩阵：

```
m = [1 2 3; 4 5 6; 7 8 9]
```



## 系统命令

使用MATLAB的时候有一些系统命令可以方便我们的操作，如在当前的工作区中可以使用系统命令保存为一个文件、加载文件、显示日期、列出目录中的文件和显示当前目录等。

下表列举了一些MATLAB常用的系统相关的命令：

| 命令      | 目的/作用               |
| ------- | ------------------- |
| cd      | 改变当前目录。             |
| date    | 显示当前日期。             |
| delete  | 删除一个文件。             |
| diary   | 日记文件记录开/关切换。        |
| dir     | 列出当前目录中的所有文件。       |
| load    | 负载工作区从一个文件中的变量。     |
| path    | 显示搜索路径。             |
| pwd     | 显示当前目录。             |
| save    | 保存在一个文件中的工作区变量。     |
| type    | 显示一个文件的​​内容。        |
| what    | 列出所有MATLAB文件在当前目录中。 |
| wklread | 读取.wk1电子表格文件。       |



## 输入和输出命令

MATLAB提供了以下输入和输出相关的命令：

| 命令      | 作用/目的          |
| ------- | -------------- |
| disp    | 显示一个数组或字符串的内容。 |
| fscanf  | 阅读从文件格式的数据。    |
| format  | 控制屏幕显示的格式。     |
| fprintf | 执行格式化写入到屏幕或文件。 |
| input   | 显示提示并等待输入。     |
| ;       | 禁止显示网版印刷       |

fscanf和fprintf命令的行为像C scanf和printf函数。他们支持格式如下代码：

| 格式代码   | 目的/作用                    |
| ------ | ------------------------ |
| **%s** | 输出字符串                    |
| **%d** | 输出整数                     |
| **%f** | 输出浮点数                    |
| **%e** | 显示科学计数法形式                |
| **%g** | %f 和%e 的结合，根据数据选择适当的显示方式 |



## 数据导入

MATLAB中导入数据意味着从外部文件加载数据。importdata 函数允许加载各种数据的不同格式的文件。它具有以下五种形式：

| S.N. | 函数&说明                                    |
| ---- | ---------------------------------------- |
| 1    | **A = importdata(filename)**将数据从文件名所表示的文件中加载到数组 A 中。 |
| 2    | **A = importdata('-pastespecial') **从系统剪贴板加载数据，而不是从文件加载数据。 |
| 3    | **A = importdata(___, delimiterIn) **将 delimiterIn 解释为 ASCII 文件、文件名或剪贴板数据中的列分隔符。可以将 delimiterIn 与上述语法中的任何输入参数一起使用。 |
| 4    | **A = importdata(___, delimiterIn, headerlinesIn)**从 ASCII 文件、文件名或剪贴板加载数据，并从 lineheaderlinesIn+1 开始读取数字数据。 |
| 5    | **[A, delimiterOut, headerlinesOut] = importdata(___)**在分隔符输出中返回检测到的分隔符字符，并使用前面语法中的任何输入参数检测headerlinesOut 中检测到的标题行数。 |



我们建立以空格分隔的 ASCII 文件的列标题，文件名为 weeklydata.txt。

文本文件 weeklydata.txt 内容如下：

```
SunDay  MonDay  TuesDay  WednesDay  ThursDay  FriDay  SatureDay
95.01   76.21   61.54    40.57       55.79    70.28   81.53
73.11   45.65   79.19    93.55       75.29    69.87   74.68
60.68   41.85   92.18    91.69       81.32    90.38   74.51
48.60   82.14   73.82    41.03       0.99     67.22   93.18
89.13   44.47   57.63    89.36       13.89    19.88   46.60
```

在MATLAB中建立一个脚本文件，并输入下述代码：

```
filename = 'weeklydata.txt';
delimiterIn = ' ';
headerlinesIn = 1;
A = importdata(filename,delimiterIn,headerlinesIn);
% View data
for k = [1:7]
   disp(A.colheaders{1, k})
   disp(A.data(:, k))
   disp(' ')
end
```



## 绘图

在MATLAB中建立一个脚本文件，输入下述代码：

```
x = [0:0.01:10];
y = sin(x);
plot(x, y), xlabel('x'), ylabel('Sin(x)'), title('Sin(x) Graph'),
grid on, axis equal
```



### 同一张图上绘制多个函数

```
x = [0 : 0.01: 10];
y = sin(x);
g = cos(x);
plot(x, y, x, g, '.-'), legend('Sin(x)', 'Cos(x)')
```

```
x = [-10 : 0.01: 10];
y = 3*x.^4 + 2 * x.^3 + 7 * x.^2 + 2 * x + 9;
g = 5 * x.^3 + 9 * x + 2;
plot(x, y, 'r', x, g, 'g')
```



### 设置轴刻度

```
axis ( [xmin xmax ymin ymax] )
```

```
x = [0 : 0.01: 10];
y = exp(-x).* sin(2*x + 3);
plot(x, y), axis([0 10 -1 1])
```



### 生成子图

```
subplot(m, n, p)
```

其中，m 和 n 为积阵列的行和列的数量，p 指定把一个特定的积。

```
x = [0:0.01:5];
y = exp(-1.5*x).*sin(10*x);
subplot(1,2,1)
plot(x,y), xlabel('x'),ylabel('exp(–1.5x)*sin(10x)'),axis([0 5 -1 1])
y = exp(-2*x).*sin(10*x);
subplot(1,2,2)
plot(x,y),xlabel('x'),ylabel('exp(–2x)*sin(10x)'),axis([0 5 -1 1])
```



## MATLAB图形

### 绘制条形图

使用 bar 命令绘制一个二维条形图

```
x = [1:10];
y = [75, 58, 90, 87, 50, 85, 92, 75, 60, 95];
bar(x,y), xlabel('Student'),ylabel('Score'),
title('First Sem:')
print -deps graph.eps
```



### 绘制等值线

绘制函数 g = f(x, y), where −5 ≤ x ≤ 5, −3 ≤ y ≤ 3，这两个值的增量为0.1。

```
[x,y] = meshgrid(-5:0.1:5,-3:0.1:3); %independent variables
g = x.^2 + y.^2;                     % our function
[C, h] = contour(x,y,g);             % call the contour function
set(h,'ShowText','on','TextStep',get(h,'LevelStep')*2)
print -deps graph.eps
```



### 绘制三维图

让我们建立一个三维地图函数表面 
$$
g = x*e-(x2 + y2)
$$

```
[x,y] = meshgrid(-2:.2:2);
g = x .* exp(-x.^2 - y.^2);
surf(x, y, g)
print -deps graph.eps
```



## 代数

MATLAB 中使用 solve 命令求解代数方程组。在其最简单的形式，solve 函数需要括在引号作为参数方程。

例如，让我们在方程求解 x， x-5 = 0

```
solve('x-5=0')
```

如果公式涉及多个符号，那么MATLAB默认情况下，假定正在解决 x，解决命令具有另一种形式：

```
solve(equation, variable)
```

在那里，还可以提到的变量。

例如，让我们来解决方程 v – u – 3t2 = 0, 或 v 在这种情况下，我们应该这样写：

```
solve('v-u-3*t^2=0', 'v')
```



### 解决二次方程

```
eq = 'x^2 -7*x + 12 = 0';
s = solve(eq);
disp('The first root is: '), disp(s(1));
disp('The second root is: '), disp(s(2));
```



### 解高阶方程

solve 命令还可以解决高阶方程。例如，让我们来解决一个三次方程 (x-3)2(x-7) = 0

```
solve('(x-3)^2*(x-7)=0')
```



下面的例子解决了四阶方程 x4 − 7x3 + 3x2 − 5x + 9 = 0.

```
eq = 'x^4 - 7*x^3 + 3*x^2 - 5*x + 9 = 0';
s = solve(eq);
disp('The first root is: '), disp(s(1));
disp('The second root is: '), disp(s(2));
disp('The third root is: '), disp(s(3));
disp('The fourth root is: '), disp(s(4));
% converting the roots to double type
disp('Numeric value of first root'), disp(double(s(1)));
disp('Numeric value of second root'), disp(double(s(2)));
disp('Numeric value of third root'), disp(double(s(3)));
disp('Numeric value of fourth root'), disp(double(s(4)));
```



在 Octave求解高阶方程

下面的例子解决了四阶方程 x4 − 7x3 + 3x2 − 5x + 9 = 0.

```
v = [1, -7,  3, -5, 9];

s = roots(v);
% converting the roots to double type
disp('Numeric value of first root'), disp(double(s(1)));
disp('Numeric value of second root'), disp(double(s(2)));
disp('Numeric value of third root'), disp(double(s(3)));
disp('Numeric value of fourth root'), disp(double(s(4)));
```



### 求解方程组

5x + 9y = 5

3x – 6y = 4

在MATLAB中建立一个脚本文件，并输入下述代码：

```
s = solve('5*x + 9*y = 5','3*x - 6*y = 4');
s.x
s.y
```



### 扩大和收集方程

```
syms x %symbolic variable x
syms y %symbolic variable x
% expanding equations
expand((x-5)*(x+9))
expand((x+2)*(x-3)*(x-5)*(x+7))
expand(sin(2*x))
expand(cos(x+y))
 
% collecting equations
collect(x^3 *(x-7))
collect(x^4*(x-3)*(x-5))
```

运行该文件，显示以下结果：

```
ans =
 x^2 + 4*x - 45
 ans =
 x^4 + x^3 - 43*x^2 + 23*x + 210
 ans =
 2*cos(x)*sin(x)
 ans =
cos(x)*cos(y) - sin(x)*sin(y)
 ans =
 x^4 - 7*x^3
 ans =
 x^6 - 8*x^5 + 15*x^4 
```



## 微积分

计算一个函数的极限 f(x) = (x3 + 5)/(x4 + 7)， 当 x 趋于零。

```
syms x
limit((x^3 + 5)/(x^4 + 7))
```

你需要使用 SYMS 命令告诉 MATLAB你使用的符号变量。

计算函数极限 f(x) = (x-3)/(x-1),  x 无限接近于 1.

```
limit((x - 3)/(x-1),1)
```

```
limit(x^2 + 5, 3)
```



### 左，右侧限制

```
f = (x - 3)/abs(x-3);
ezplot(f,[-1,5])
l = limit(f,x,3,'left')
r = limit(f,x,3,'right')
```



## 多项式

方程 P(x) = x4 + 7x3 - 5x + 9 可以表示为：

p = [1 7 0 -5 9]；

要计算多项式 p，输入根：

```
p = [1 7 0  -5 9];
r = roots(p)
```



### 多项式曲线拟合

polyfit 函数找到一个多项式的系数，适合采用最小二乘意义上的一组中的数据。

如果 x 和 y 是两个向量含有的 x 和 y 被拟合数据的一个 n 次多项式，那么我们得到的多项式拟合的数据通过写入

```
p = polyfit(x,y,n)
```

```
x = [1 2 3 4 5 6]; y = [5.5 43.1 128 290.7 498.4 978.67];  %data
p = polyfit(x,y,4)   %get the polynomial
% Compute the values of the polyfit estimate over a finer range, 
% and plot the estimate over the real data values for comparison:
x2 = 1:.1:6;          
y2 = polyval(p,x2);
plot(x,y,'o',x2,y2)
grid on
```



## 拉普拉斯变换

### 拉普拉斯变换

拉普拉斯变换将微分方程转化为代数。要计算一个函数 f（t）的拉普拉斯变换，这样写：

```
laplace(f(t))
```

```
syms s t a b w
laplace(a)
laplace(t^2)
laplace(t^9)
laplace(exp(-b*t))
laplace(sin(w*t))
laplace(cos(w*t))
```

### 逆拉普拉斯变换

```
ilaplace(1/s^3)
```

```
syms s t a b w
ilaplace(1/s^7)
ilaplace(2/(w+s))
ilaplace(s/(s^2+4))
ilaplace(exp(-b*t))
ilaplace(w/(s^2 + w^2))
ilaplace(s/(s^2 + w^2))
```

### 傅立叶变换

```
syms x 
f = exp(-2*x^2);  %our function
ezplot(f,[-2,2])  % plot of our function
FT = fourier(f)	% Fourier transform
```

### 傅立叶逆变换

```
f = ifourier(-2*exp(-abs(w)))
```



## Matlab曲线拟合工具箱

Custom Equations：用户自定义的函数类型
Exponential：指数逼近，有2种类型， a*exp(b*x) 、 a*exp(b*x) + c*exp(d*x)
Fourier：傅立叶逼近，有7种类型，基础型是 a0 + a1*cos(x*w) + b1*sin(x*w)
Gaussian：高斯逼近，有8种类型，基础型是 a1*exp(-((x-b1)/c1)^2)
Interpolant：插值逼近，有4种类型，linear、nearest neighbor、cubic spline、shape-preserving
Polynomial：多形式逼近，有9种类型，linear ~、quadratic ~、cubic ~、4-9th degree ~
Power：幂逼近，有2种类型，a*x^b 、a*x^b + c
Rational：有理数逼近，分子、分母共有的类型是linear ~、quadratic ~、cubic ~、4-5th degree ~；此外，分子还包括constant型
Smoothing Spline：平滑逼近（翻译的不大恰当，不好意思）
Sum of Sin Functions：正弦曲线逼近，有8种类型，基础型是 a1*sin(b1*x + c1)
Weibull：只有一种，a*b*x^(b-1)*exp(-a*x^b)

