# Java基础概念

## 反射

　　当我们的程序在运行时，需要动态的加载一些类这些类可能之前用不到所以不用加载到 JVM，而是在运行时根据需要才加载，这样的好处对于服务器来说不言而喻。

　　举个例子我们的项目底层有时是用 mysql，有时用 oracle，需要动态地根据实际情况加载驱动类，这个时候反射就有用了，假设 com.java.dbtest.myqlConnection，com.java.dbtest.oracleConnection 这两个类我们要用，这时候我们的程序就写得比较动态化，通过 Class tc = Class.forName("com.java.dbtest.TestConnection"); 通过类的全类名让 JVM 在服务器中找到并加载这个类，而如果是 Oracle 则传入的参数就变成另一个了。这时候就可以看到反射的好处了，这个动态性就体现出 Java 的特性了！

　　举个例子，大家如果接触过 spring，会发现当你配置各种各样的 bean 时，是以配置文件的形式配置的，你需要用到哪些 bean 就配哪些，spring 容器就会根据你的需求去动态加载，你的程序就能健壮地运行。

### 什么是反射

　　反射 (Reflection) 是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。通过 Class 获取 class 信息称之为反射（Reflection）

　　简而言之，通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。

　　程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。

　　反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

　　Java 反射框架主要提供以下功能：

　　1. 在运行时判断任意一个对象所属的类

　　2. 在运行时构造任意一个类的对象

　　3. 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用 private 方法）

　　4. 在运行时调用任意一个对象的方法

　　**重点**：是运行时而不是编译时

### 主要用途

　　很多人都认为反射在实际的 Java 开发应用中并不广泛，其实不然。

当我们在使用 IDE （如Eclipse，IDEA）时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。

　　**反射最重要的用途就是开发各种通用框架**

　　很多框架（比如 Spring ）都是配置化的（比如通过 XML 文件配置 JavaBean,Action 之类的），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。

　　对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

### 获得Class对象

1. 调用运行时类本身的 `.class` 属性

```
Class clazz1 = Person.class;
System.out.println(clazz1.getName());
```

1. 通过运行时类的对象获取 `getClass();`

```
Person p = new Person();
Class clazz3 = p.getClass();
System.out.println(clazz3.getName());
```

1. 使用 Class 类的 `forName` 静态方法

```
public static Class<?> forName(String className)
// 在JDBC开发中常用此方法加载数据库驱动:
Class.forName(driver);
```

1. （了解）通过类的加载器 ClassLoader

```
ClassLoader classLoader = this.getClass().getClassLoader();
Class clazz5 = classLoader.loadClass(className);
System.out.println(clazz5.getName());
```

## 注解

### 什么是注解

　　Annontation 是 Java5 开始引入的新特征，中文名称叫注解。它提供了一种安全的类似注释的机制，用来**将任何的信息或元数据（metadata）与程序元素（类、方法、成员变量等）进行关联**。为程序的元素（类、方法、成员变量）加上更直观更明了的说明，这些说明信息是与程序的业务逻辑无关，并且供指定的工具或框架使用。Annontation 像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的声明语句中。

　　Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。包含在 `java.lang.annotation` 包中。

　　简单来说：注解其实就是**代码中的特殊标记**，这些标记可以**在编译、类加载、运行时被读取，并执行相对应的处理**。

### 为什么要用注解

传统的方式，我们是通过配置文件 `.xml` 来告诉类是如何运行的。

有了注解技术以后，我们就可以通过注解告诉类如何运行

例如：我们以前编写 Servlet 的时候，需要在 web.xml 文件配置具体的信息。我们使用了注解以后，可以直接在 Servlet 源代码上，增加注解...Servlet 就被配置到 Tomcat 上了。也就是说，注解可以给类、方法上注入信息。

明显地可以看出，这样是非常直观的，并且 Servlet 规范是推崇这种配置方式的。

### 基本Annotation

在 java.lang 包下存在着5个基本的 Annotation，重点掌握前三个。

1. @Override 重写注解
   - 如果我们使用IDE重写父类的方法，我们就可以看见它了。
   - @Override是告诉编译器要检查该方法是实现父类的，可以帮我们避免一些低级的错误。
   - 比如，我们在实现 equals() 方法的时候，把 euqals() 打错了，那么编译器就会发现该方法并不是实现父类的，与注解 @Override 冲突，于是就会给予错误。
2. @Deprecated 过时注解
   - 该注解也非常常见，Java 在设计的时候，可能觉得某些方法设计得不好，为了兼容以前的程序，是不能直接把它抛弃的，于是就设置它为过时。
   - Date对象中的 toLocalString() 就被设置成过时了
   - 当我们在程序中调用它的时候，在 IDE 上会出现一条横杠，说明该方法是过时的。

```
@Deprecated
public String toLocaleString() {
    DateFormat formatter = DateFormat.getDateTimeInstance();
    return formatter.format(this);
}
```

1. @SuppressWarnings 抑制编译器警告注解
   - 该注解在我们写程序的时候并不是很常见，我们可以用它来让编译器不给予我们警告
   - 当我们在使用集合的时候，如果没有指定泛型，那么会提示安全检查的警告
   - 如果我们在类上添加了@SuppressWarnings这个注解，那么编译器就不会给予我们警告了
2. @SafeVarargs Java 7“堆污染”警告
   - 什么是堆污染呢？？当把一个不是泛型的集合赋值给一个带泛型的集合的时候，这种情况就很容易发生堆污染。
   - 这个注解也是用来抑制编译器警告的注解，用的地方并不多。
3. @FunctionalInterface 用来指定该接口是函数式接口
   - 用该注解显示指定该接口是一个函数式接口。

### 自定义注解类编写规则

1. Annotation 型定义为 @interface, 所有的 Annotation 会自动继承 java.lang.Annotation 这一接口，并且不能再去继承别的类或是接口.
2. 参数成员只能用 public 或默认(default)这两个访问权修饰
3. 参数成员只能用基本类型 byte,short,char,int,long,float,double,boolean 八种基本数据类型和 String、Enum、Class、annotations 等数据类型，以及这一些类型的数组
4. 要获取类方法和字段的注解信息，必须通过 Java 的反射技术来获取 Annotation 对象，因为你除此之外没有别的获取注解对象的方法
5. 注解也可以没有定义成员, 不过这样注解就没啥用了 PS：自定义注解需要使用到元注解

### 自定义注解实例

```
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

/**
 * 水果名称注解
 */
@Target(FIELD)
@Retention(RUNTIME)
@Documented
public @interface FruitName {
    String value() default "";
}
```

## 泛型

### 通俗解释

　　通俗的讲，泛型就是操作类型的 占位符，即：假设占位符为 T，那么此次声明的数据结构操作的数据类型为T类型。

　　假定我们有这样一个需求：写一个排序方法，能够对整型数组、字符串数组甚至其他任何类型的数组进行排序，该如何实现？答案是可以使用 **Java 泛型**。

　　使用 Java 泛型的概念，我们可以写一个泛型方法来对一个对象数组排序。然后，调用该泛型方法来对整型数组、浮点数数组、字符串数组等进行排序。

### 泛型方法

　　你可以写一个泛型方法，该方法在调用时可以接收不同类型的参数。根据传递给泛型方法的参数类型，编译器适当地处理每一个方法调用。

下面是定义泛型方法的规则：

- 所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前（在下面例子中的 <E>）。
- 每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。
- 类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符。
- 泛型方法体的声明和其他方法一样。注意类型参数 **只能代表引用型类型，不能是原始类型** （像 int,double,char 的等）。

```
public class GenericMethodTest
{
   // 泛型方法 printArray                         
   public static < E > void printArray( E[] inputArray )
   {
      // 输出数组元素            
         for ( E element : inputArray ){        
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
 
    public static void main( String args[] )
    {
        // 创建不同类型数组： Integer, Double 和 Character
        Integer[] intArray = { 1, 2, 3, 4, 5 };
        Double[] doubleArray = { 1.1, 2.2, 3.3, 4.4 };
        Character[] charArray = { 'H', 'E', 'L', 'L', 'O' };
 
        System.out.println( "整型数组元素为:" );
        printArray( intArray  ); // 传递一个整型数组
 
        System.out.println( "\n双精度型数组元素为:" );
        printArray( doubleArray ); // 传递一个双精度型数组
 
        System.out.println( "\n字符型数组元素为:" );
        printArray( charArray ); // 传递一个字符型数组
    } 
}
```

### 泛型类

　　泛型类的声明和非泛型类的声明类似，除了在类名后面添加了类型参数声明部分。

　　和泛型方法一样，泛型类的类型参数声明部分也包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。因为他们接受一个或多个参数，这些类被称为参数化的类或参数化的类型。

```
public class Box<T> {
	private T t;
	public void add(T t) {
	    this.t = t;
	}

	public T get() {
	    return t;
	}

	public static void main(String[] args) {
	    Box<Integer> integerBox = new Box<Integer>();
	    Box<String> stringBox = new Box<String>();

	    integerBox.add(new Integer(10));
	    stringBox.add(new String("菜鸟教程"));

	    System.out.printf("整型值为 :%d\n\n", integerBox.get());
	    System.out.printf("字符串为 :%s\n", stringBox.get());
	}
}
```

### 类型通配符

1. 类型通配符一般是使用 `?` 代替具体的类型参数。例如 `List` 在逻辑上是 `List`，`List` 等所有 **List<具体类型实参>** 的父类。
2. 类型通配符上限通过形如 List 来定义，如此定义就是通配符泛型值接受 Number 及其下层子类类型。
3. 类型通配符下限通过形如 List<? super Number> 来定义，表示类型只能接受 Number 及其三层父类类型，如 Objec 类型的实例。

## 深拷贝与浅拷贝

- **浅拷贝**：被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。换言之，浅拷贝仅仅复制所拷贝的对象，而不复制它所引用的对象。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/shadow_copy2.jpg)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/shadow_copy2.jpg)

- **深拷贝**：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/deep_copy2.jpg)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/deep_copy2.jpg)

## Lamda表达式

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

### 语法

lambda 表达式的语法格式如下：

```
(parameters) -> expression
或
(parameters) -> { statements; }
```

以下是 lambda 表达式的重要特征:

- **可选类型声明：** 不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：** 一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：** 如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：** 如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

### Lambda 表达式实例

Lambda 表达式的简单例子:

```
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
```

在 Java8Tester.java 文件输入以下代码：

```
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester tester = new Java8Tester();
        
      // 类型声明
      MathOperation addition = (int a, int b) -> a + b;
        
      // 不用类型声明
      MathOperation subtraction = (a, b) -> a - b;
        
      // 大括号中的返回语句
      MathOperation multiplication = (int a, int b) -> { return a * b; };
        
      // 没有大括号及返回语句
      MathOperation division = (int a, int b) -> a / b;
        
      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        
      // 不用括号
      GreetingService greetService1 = message ->
      System.out.println("Hello " + message);
        
      // 用括号
      GreetingService greetService2 = (message) ->
      System.out.println("Hello " + message);
        
      greetService1.sayMessage("Runoob");
      greetService2.sayMessage("Google");
   }
    
   interface MathOperation {
      int operation(int a, int b);
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
    
   private int operate(int a, int b, MathOperation mathOperation){
      return mathOperation.operation(a, b);
   }
}
```

执行以上脚本，输出结果为：

```
$ javac Java8Tester.java 
$ java Java8Tester
10 + 5 = 15
10 - 5 = 5
10 x 5 = 50
10 / 5 = 2
Hello Runoob
Hello Google
```

使用 Lambda 表达式需要注意以下两点：

- Lambda 表达式主要用来定义行内执行的方法类型接口，例如，一个简单方法接口。在上面例子中，我们使用各种类型的 Lambda 表达式来定义 MathOperation 接口的方法。然后我们定义了 sayMessage 的执行。
- Lambda 表达式免去了使用匿名方法的麻烦，并且给予 Java 简单但是强大的函数化的编程能力。

### 变量作用域

lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

在 Java8Tester.java 文件输入以下代码：

```
public class Java8Tester {
 
   final static String salutation = "Hello! ";
   
   public static void main(String args[]){
      GreetingService greetService1 = message -> 
      System.out.println(salutation + message);
      greetService1.sayMessage("Runoob");
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
}
```

执行以上脚本，输出结果为：

```
$ javac Java8Tester.java 
$ java Java8Tester
Hello! Runoob
```

我们也可以直接在 lambda 表达式中访问外层的局部变量：

```
public class Java8Tester {
    public static void main(String args[]) {
        final int num = 1;
        Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));
        s.convert(2);  // 输出结果为 3
    }
 
    public interface Converter<T1, T2> {
        void convert(int i);
    }
}
```

lambda 表达式的局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）

```
int num = 1;  
Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));
s.convert(2);
num = 5;  
//报错信息：Local variable num defined in an enclosing scope must be final or effectively 
 final
```

在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。

```
String first = ""; 

//编译会出错
Comparator<String> comparator = (first, second) -> Integer.compare(first.length(), second.length());
```

##  面向对象

开发的六个基本原则，在项目中用过哪些原则

- **六个基本原则**
  - **单一职责**（Single Responsibility Principle 简称 SRP）：**一个类应该仅有一个引起它变化的原因**。在面向对象中，如果只让一个类完成它该做的事，而不涉及与它无关的领域就是践行了高内聚的原则，这个类就只有单一职责。
  - **里氏替换**（Liskov Substitution Principle 简称 LSP）：**任何时候子类型能够替换掉它们的父类型**。子类一定是增加父类的能力而不是减少父类的能力，因为子类比父类的能力更多，把能力多的对象当成能力少的对象来用当然没有任何问题。
  - **依赖倒置**（Dependence Inversion Principle 简称 DIP）：**要依赖于抽象，不要依赖于具体类**。要做到依赖倒置，应该做到：①高层模块不应该依赖底层模块，二者都应该依赖于抽象；②抽象不应该依赖于具体实现，具体实现应该依赖于抽象。
  - **接口隔离**（Interface Segregation Principle 简称 ISP）：**不应该强迫客户依赖于他们不用的方法** 。接口要小而专，绝不能大而全。臃肿的接口是对接口的污染，既然接口表示能力，那么一个接口只应该描述一种能力，接口也应该是高度内聚的。
  - **最少知识原则**（Least Knowledge Principle 简称 LKP）：**只和你的朋友谈话**。迪米特法则又叫最少知识原则，一个对象应当对其他对象有尽可能少的了解。
  - **开闭原则**（Open Closed Principle 简称 OCP）：**软件实体应当对扩展开放，对修改关闭**。要做到开闭有两个要点：①抽象是关键，一个系统中如果没有抽象类或接口系统就没有扩展点；②封装可变性，将系统中的各种可变因素封装到一个继承结构中，如果多个可变因素混杂在一起，系统将变得复杂而换乱。
- 其他原则
  - 合成聚和复用：优先使用聚合或合成关系复用代码
  - 面向接口编程
  - 优先使用组合，而非继承
  - 一个类需要的数据应该隐藏在类的内部
  - 类之间应该零耦合，或者只有传导耦合，换句话说，类之间要么没关系，要么只使用另一个类的接口提供的操作
  - 在水平方向上尽可能统一地分布系统功能
- 项目中用到的原则
  - 单一职责、开放封闭、合成聚合复用(最简单的例子就是String类)、接口隔离

## final、finally和finalize区别

### final

final 用于声明属性、方法和类，分别表示属性不可变、方法不可覆盖和类不可被继承。

- final 属性：被final修饰的变量不可变（引用不可变）
- final 方法：不允许任何子类重写这个方法，但子类仍然可以使用这个方法
- final 参数：用来表示这个参数在这个函数内部不允许被修改
- final 类：此类不能被继承，所有方法都不能被重写

### finally

　　在异常处理的时候，提供 finally 块来执行任何的清除操作。如果抛出一个异常，那么相匹配的 catch 字句就会执行，然后控制就会进入 finally 块，前提是有 finally 块。例如：数据库连接关闭操作上

　　finally 作为异常处理的一部分，它只能用在 try/catch 语句中，并且附带一个语句块，表示这段语句最终一定会被执行（不管有没有抛出异常），经常被用在需要释放资源的情况下。（×）（**这句话其实存在一定的问题，还没有深入了解，欢迎大家在 issue 中提出自己的见解）**

- 异常情况说明：
  - 在执行 try 语句块之前已经返回或抛出异常，所以 try 对应的 finally 语句并没有执行。
  - 我们在 try 语句块中执行了 System.exit (0) 语句，终止了 Java 虚拟机的运行。那有人说了，在一般的 Java 应用中基本上是不会调用这个 System.exit(0) 方法的
  - 当一个线程在执行 try 语句块或者 catch 语句块时被打断（interrupted）或者被终止（killed），与其相对应的 finally 语句块可能不会执行
  - 还有更极端的情况，就是在线程运行 try 语句块或者 catch 语句块时，突然死机或者断电，finally 语句块肯定不会执行了。可能有人认为死机、断电这些理由有些强词夺理，没有关系，我们只是为了说明这个问题。

### finalize

　　finalize() 是 Object 中的方法，当垃圾回收器将要回收对象所占内存之前被调用，即当一个对象被虚拟机宣告死亡时会先调用它 finalize() 方法，让此对象处理它生前的最后事情（这个对象可以趁这个时机挣脱死亡的命运）。要明白这个问题，先看一下虚拟机是如何判断一个对象该死的。

　　可以覆盖此方法来实现对其他资源的回收，例如关闭文件。

#### 判定死亡

　　Java 采用可达性分析算法来判定一个对象是否死期已到。Java中以一系列 "GC Roots" 对象作为起点，如果一个对象的引用链可以最终追溯到 "GC Roots" 对象，那就天下太平。

　　否则如果只是A对象引用B，B对象又引用A，A B引用链均未能达到 "GC Roots" 的话，那它俩将会被虚拟机宣判符合死亡条件，具有被垃圾回收器回收的资格。

#### 最后的救赎

上面提到了判断死亡的依据，但被判断死亡后，还有生还的机会。

如何自我救赎：

1. 对象覆写了 finalize() 方法（这样在被判死后才会调用此方法，才有机会做最后的救赎）；
2. 在 finalize() 方法中重新引用到 "GC Roots" 链上（如把当前对象的引用 this 赋值给某对象的类变量/成员变量，重新建立可达的引用）.

需要注意：

　　finalize() 只会在对象内存回收前被调用一次 (The finalize method is never invoked more than once by a Java virtual machine for any given object. )

　　finalize() 的调用具有不确定性，只保证方法会调用，但不保证方法里的任务会被执行完（比如一个对象手脚不够利索，磨磨叽叽，还在自救的过程中，被杀死回收了）。

#### finalize()的作用

　　虽然以上以对象救赎举例，但 finalize() 的作用往往被认为是用来做最后的资源回收。 　　基于在自我救赎中的表现来看，此方法有很大的不确定性（不保证方法中的任务执行完）而且运行代价较高。所以用来回收资源也不会有什么好的表现。

　　综上：finalize() 方法并没有什么鸟用。

　　至于为什么会存在一个鸡肋的方法：书中说 “它不是 C/C++ 中的析构函数，而是 Java 刚诞生时为了使 C/C++ 程序员更容易接受它所做出的一个妥协”。

## ValueOf缓存池

　　new Integer(123) 与 Integer.valueOf(123) 的区别在于，new Integer(123) 每次都会新建一个对象，而 Integer.valueOf(123) 可能会使用缓存对象，因此多次使用 Integer.valueOf(123) 会取得同一个对象的引用。

```
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

　　编译器会在自动装箱过程调用 valueOf() 方法，因此多个 Integer 实例使用自动装箱来创建并且值相同，那么就会引用相同的对象。

```
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接使用缓存池的内容。

```
// valueOf 源码实现
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

```
static final int low = -128;
static final int high;
static final Integer cache[];

static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
}
```

Java 还将一些其它基本类型的值放在缓冲池中，包含以下这些：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

因此在使用这些基本类型对应的包装类型时，就可以直接使用缓冲池中的对象。

[reference](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/01-Java%E5%9F%BA%E7%A1%80.md)

## Java 语言有哪些特点

1. 简单易学；
2. 面向对象（封装，继承，多态）；
3. 平台无关性（ Java 虚拟机实现平台无关性）；
4. 可靠性；
5. 安全性；
6. 支持多线程（ C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持）；
7. 支持网络编程并且很方便（ Java 语言诞生本身就是为简化网络编程设计的，因此 Java 语言不仅支持网络编程而且很方便）；
8. 编译与解释并存

##  Java 程序的主类 应用程序和小程序的主类有何不同

一个程序中可以有多个类，但只能有一个类是主类。在 Java 应用程序中，这个主类是指包含 main（）方法的类。而在 Java 小程序中，这个主类是一个继承自系统类 JApplet 或 Applet 的子类。应用程序的主类不一定要求是 public 类，但小程序的主类要求必须是 public 类。主类是 Java 程序执行的入口点。

## 字符型常量和字符串常量的区别

1. 形式上: 字符常量是单引号引起的一个字符 字符串常量是双引号引起的若干个字符
2. 含义上: 字符常量相当于一个整形值( ASCII 值),可以参加表达式运算 字符串常量代表一个地址值(该字符串在内存中存放位置)
3. 占内存大小 字符常量只占2个字节 字符串常量占若干个字节(至少一个字符结束标志) (**注意： char在Java中占两个字节**)

[![img](https://camo.githubusercontent.com/d913ab9b3880feab7d326a0904caac5f5e285a56/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d31352f38363733353531392e6a7067)](https://camo.githubusercontent.com/d913ab9b3880feab7d326a0904caac5f5e285a56/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d31352f38363733353531392e6a7067)

## 构造器 Constructor 是否可被 override

在讲继承的时候我们就知道父类的私有属性和构造方法并不能被继承，所以 Constructor 也就不能被 override（重写）,但是可以 overload（重载）,所以你可以看到一个类中有多个构造函数的情况。

## Java 面向对象编程三大特性:封装、继承、多态

### 封装

封装把一个对象的属性私有化，同时提供一些可以被外界访问的属性的方法，如果属性不想被外界访问，我们大可不必提供方法给外界访问。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。

### 继承

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承我们能够非常方便地复用以前的代码。

**关于继承如下 3 点请记住：**

1. 子类拥有父类非 private 的属性和方法。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。

### 多态

所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

在Java中有两种形式可以实现多态：继承（多个子类对同一方法的重写）和接口（实现接口并覆盖接口中同一方法）。

## 自动装箱与拆箱

**装箱**：将基本类型用它们对应的引用类型包装起来；

**拆箱**：将包装类型转换为基本数据类型；

## 在一个静态方法内调用一个非静态成员为什么是非法的

由于静态方法可以不通过对象进行调用，因此在静态方法里，不能调用其他非静态变量，也不可以访问非静态变量成员。

## 在 Java 中定义一个不做事且没有参数的构造方法的作用

　Java 程序在执行子类的构造方法之前，如果没有用 super() 来调用父类特定的构造方法，则会调用父类中“没有参数的构造方法”。因此，如果父类中只定义了有参数的构造方法，而在子类的构造方法中又没有用 super() 来调用父类中特定的构造方法，则编译时将发生错误，因为 Java 程序在父类中找不到没有参数的构造方法可供执行。解决办法是在父类里加上一个不做事且没有参数的构造方法。 　

## import java和javax有什么区别

刚开始的时候 JavaAPI 所必需的包是 java 开头的包，javax 当时只是扩展 API 包来说使用。然而随着时间的推移，javax 逐渐的扩展成为 Java API 的组成部分。但是，将扩展从 javax 包移动到 java 包将是太麻烦了，最终会破坏一堆现有的代码。因此，最终决定 javax 包将成为标准API的一部分。

所以，实际上java和javax没有区别。这都是一个名字。

## 接口和抽象类的区别是什么

1. 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），抽象类可以有非抽象的方法
2. 接口中的实例变量默认是 final 类型的，而抽象类中则不一定
3. 一个类可以实现多个接口，但最多只能实现一个抽象类
4. 一个类实现接口的话要实现接口的所有方法，而抽象类不一定
5. 接口不能用 new 实例化，但可以声明，但是必须引用一个实现该接口的对象 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

## 成员变量与局部变量的区别有那些

1. 从语法形式上，看成员变量是属于类的，而局部变量是在方法中定义的变量或是方法的参数；成员变量可以被 public,private,static 等修饰符所修饰，而局部变量不能被访问控制修饰符及 static 所修饰；但是，成员变量和局部变量都能被 final 所修饰；
2. 从变量在内存中的存储方式来看，成员变量是对象的一部分，而对象存在于堆内存，局部变量存在于栈内存
3. 从变量在内存中的生存时间上看，成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。
4. 成员变量如果没有被赋初值，则会自动以类型的默认值而赋值（一种情况例外被 final 修饰但没有被 static 修饰的成员变量必须显示地赋值）；而局部变量则不会自动赋值。

## 创建一个对象用什么运算符？对象实体与对象引用有何不同？

new运算符，new创建对象实例（对象实例在堆内存中），对象引用指向对象实例（对象引用存放在栈内存中）。一个对象引用可以指向0个或1个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有n个引用指向它（可以用n条绳子系住一个气球）。

## 什么是方法的返回值？返回值在类的方法里的作用是什么？

方法的返回值是指我们获取到的某个方法体中的代码执行后产生的结果！（前提是该方法可能产生结果）。返回值的作用:接收出结果，使得它可以用于其他的操作！

## 一个类的构造方法的作用是什么 若一个类没有声明构造方法，该程序能正确执行吗 为什么

主要作用是完成对类对象的初始化工作。可以执行。因为一个类即使没有声明构造方法也会有默认的不带参数的构造方法。

## 构造方法有哪些特性

1. 名字与类名相同；
2. 没有返回值，但不能用void声明构造函数；
3. 生成类的对象时自动执行，无需调用。

## 静态方法和实例方法有何不同

1. 在外部调用静态方法时，可以使用"类名.方法名"的方式，也可以使用"对象名.方法名"的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。
2. 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），而不允许访问实例成员变量和实例方法；实例方法则无此限制.

## 对象的相等与指向他们的引用相等，两者有什么不同？

对象的相等，比的是内存中存放的内容是否相等。而引用相等，比较的是他们指向的内存地址是否相等。

##  在调用子类构造方法之前会先调用父类没有参数的构造方法，其目的是？

帮助子类做初始化工作。

## == 与 equals

**==** : 它的作用是判断两个对象的地址是不是相等。即，判断两个对象是不是同一个对象。(基本数据类型==比较的是值，引用数据类型==比较的是内存地址)

**equals()** : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：

- 情况1：类没有覆盖 equals() 方法。则通过 equals() 比较该类的两个对象时，等价于通过“==”比较这两个对象。
- 情况2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来两个对象的内容相等；若它们的内容相等，则返回 true (即，认为这两个对象相等)。

**举个例子：**

```
public class test1 {
    public static void main(String[] args) {
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab"); // b为另一个引用,对象的内容一样
        String aa = "ab"; // 放在常量池中
        String bb = "ab"; // 从常量池中查找
        if (aa == bb) // true
            System.out.println("aa==bb");
        if (a == b) // false，非同一对象
            System.out.println("a==b");
        if (a.equals(b)) // true
            System.out.println("aEQb");
        if (42 == 42.0) { // true
            System.out.println("true");
        }
    }
}
```

**说明：**

- String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
- 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象。

##  hashCode 与 equals

面试官可能会问你：“你重写过 hashcode 和 equals 么，为什么重写equals时必须重写hashCode方法？”

### hashCode（）介绍

hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数。

散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！（可以快速找到所需要的对象）

### 为什么要有 hashCode

**我们以“HashSet 如何检查重复”为例子来说明为什么要有 hashCode：**

当你把对象加入 HashSet 时，HashSet 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与其他已经加入的对象的 hashcode 值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 equals（）方法来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的Java启蒙书《Head fist java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

### hashCode（）与equals（）的相关规定

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个对象分别调用equals方法都返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. **因此，equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

## 为什么Java中只有值传递

[为什么Java中只有值传递？](https://github.com/Snailclimb/Java-Guide/blob/master/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87/%E6%9C%80%E6%9C%80%E6%9C%80%E5%B8%B8%E8%A7%81%E7%9A%84Java%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/%E7%AC%AC%E4%B8%80%E5%91%A8%EF%BC%882018-8-7%EF%BC%89.md)

## 线程，程序、进程的基本概念

**线程**与进程相似，但线程是一个比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，也正因为如此，线程也被称为轻量级进程。

**程序**是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，也就是说程序是静态的代码。

**进程**是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。简单来说，一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如CPU时间，内存空间，文件，文件，输入输出设备的使用权等等。换句话说，当程序在执行时，将会被操作系统载入内存中。 线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。

## Java 中的异常处理

### Java异常类层次结构图

[![Java异常类层次结构图](https://camo.githubusercontent.com/e6b358c7ceca03c7128d52004c45a746728c76f4/687474703a2f2f696d61676573323031352e636e626c6f67732e636f6d2f626c6f672f3634313030332f3230313630372f3634313030332d32303136303730363233323034343238302d3335353335343739302e706e67)](https://camo.githubusercontent.com/e6b358c7ceca03c7128d52004c45a746728c76f4/687474703a2f2f696d61676573323031352e636e626c6f67732e636f6d2f626c6f672f3634313030332f3230313630372f3634313030332d32303136303730363233323034343238302d3335353335343739302e706e67) 

在 Java 中，所有的异常都有一个共同的祖先java.lang包中的 **Throwable类**。Throwable： 有两个重要的子类：**Exception（异常）** 和 **Error（错误）** ，二者都是 Java 异常处理的重要子类，各自都包含大量子类。

**Error（错误）:是程序无法处理的错误**，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。

这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（Virtual MachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。

**Exception（异常）:是程序本身可以处理的异常**。Exception 类有一个重要的子类 **RuntimeException**。RuntimeException 异常由Java虚拟机抛出。**NullPointerException**（要访问的变量没有引用任何对象时，抛出该异常）、**ArithmeticException**（算术运算异常，一个整数除以0时，抛出该异常）和 **ArrayIndexOutOfBoundsException** （下标越界异常）。

**注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。**

### Throwable类常用方法

- **public string getMessage()**:返回异常发生时的详细信息
- **public string toString()**:返回异常发生时的简要描述
- **public string getLocalizedMessage()**:返回异常对象的本地化信息。使用Throwable的子类覆盖这个方法，可以声称本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与getMessage（）返回的结果相同
- **public void printStackTrace()**:在控制台上打印Throwable对象封装的异常信息

### 异常处理总结

- try 块：用于捕获异常。其后可接零个或多个catch块，如果没有catch块，则必须跟一个finally块。
- catch 块：用于处理try捕获到的异常。
- finally 块：无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return语句时，finally语句块将在方法返回之前被执行。

**在以下4种特殊情况下，finally块不会被执行：**

1. 在finally语句块中发生了异常。
2. 在前面的代码中用了System.exit()退出程序。
3. 程序所在的线程死亡。
4. 关闭CPU。

## Java序列话中如果有些字段不想进行序列化 怎么办

对于不想进行序列化的变量，使用transient关键字修饰。

transient关键字的作用是：阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transient只能修饰变量，不能修饰类和方法。

# 关键字

## final 关键字

**final关键字主要用在三个地方：变量、方法、类。**

1. **对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。**
2. **当用final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。**
3. 使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为final。

## static 关键字

**static 关键字主要有以下四种使用场景：**

1. **修饰成员变量和成员方法:** 被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：`类名.静态变量名` `类名.静态方法名()`
2. **静态代码块:** 静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只执行一次.
3. **静态内部类（static修饰类的话只能修饰内部类）：** 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。没有这个引用就意味着：1. 它的创建是不需要依赖外围类的创建。2. 它不能使用任何外围类的非static成员变量和方法。
4. **静态导包(用来导入类中的静态资源，1.5之后的新特性):** 格式为：`import static` 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

## this 关键字

this关键字用于引用类的当前实例。 例如：

```
class Manager {
    Employees[] employees;
     
    void manageEmployees() {
        int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
     
    void report() { }
}
```

在上面的示例中，this关键字用于两个地方：

- this.employees.length：访问类Manager的当前实例的变量。
- this.report（）：调用类Manager的当前实例的方法。

此关键字是可选的，这意味着如果上面的示例在不使用此关键字的情况下表现相同。 但是，使用此关键字可能会使代码更易读或易懂。

## super 关键字

super关键字用于从子类访问父类的变量和方法。 例如：

```
public class Super {
    protected int number;
     
    protected showNumber() {
        System.out.println("number = " + number);
    }
}
 
public class Sub extends Super {
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```

在上面的例子中，Sub 类访问父类成员变量 number 并调用其其父类 Super 的 `showNumber（）` 方法。

**使用 this 和 super 要注意的问题：**

- super 调用父类中的其他构造方法时，调用时要放在构造方法的首行！this 调用本类中的其他构造方法时，也要放在首行。
- this、super不能用在static方法中。

**简单解释一下：**

被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享。而 this 代表对本类对象的引用，指向本类对象；而 super 代表对父类对象的引用，指向父类对象；所以， **this和super是属于对象范畴的东西，而静态方法是属于类范畴的东西**。



## static 关键字详解

### static 关键字主要有以下四种使用场景

1. 修饰成员变量和成员方法
2. 静态代码块
3. 修饰类(只能修饰内部类)
4. 静态导包(用来导入类中的静态资源，1.5之后的新特性)

### 修饰成员变量和成员方法(常用)

被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。

HotSpot 虚拟机中方法区也常被称为 “永久代”，本质上两者并不等价。仅仅是因为 HotSpot 虚拟机设计团队用永久代来实现方法区而已，这样 HotSpot 虚拟机的垃圾收集器就可以像管理 Java 堆一样管理这部分内存了。但是这并不是一个好主意，因为这样更容易遇到内存溢出问题。

调用格式：

- 类名.静态变量名
- 类名.静态方法名()

如果变量或者方法被 private 则代表该属性或者该方法只能在类的内部被访问而不能在类的外部被方法。

测试方法：

```
public class StaticBean {

    String name;
    静态变量
    static int age;

    public StaticBean(String name) {
        this.name = name;
    }
    静态方法
    static void SayHello() {
        System.out.println(Hello i am java);
    }
    @Override
    public String toString() {
        return StaticBean{ +
                name=' + name + ''' + age + age +
                '}';
    }
}
```

```
public class StaticDemo {

    public static void main(String[] args) {
        StaticBean staticBean = new StaticBean(1);
        StaticBean staticBean2 = new StaticBean(2);
        StaticBean staticBean3 = new StaticBean(3);
        StaticBean staticBean4 = new StaticBean(4);
        StaticBean.age = 33;
        StaticBean{name='1'age33} StaticBean{name='2'age33} StaticBean{name='3'age33} StaticBean{name='4'age33}
        System.out.println(staticBean+ +staticBean2+ +staticBean3+ +staticBean4);
        StaticBean.SayHello();Hello i am java
    }

}
```

### 静态代码块

静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。 该类不管创建多少对象，静态代码块只执行一次.

静态代码块的格式是

```
static {    
语句体;   
}
```

一个类中的静态代码块可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果静态代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。

[![img](https://camo.githubusercontent.com/327768e0c67af4610be2e4e36062d2c2d32d6263/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d31342f38383533313037352e6a7067)](https://camo.githubusercontent.com/327768e0c67af4610be2e4e36062d2c2d32d6263/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d31342f38383533313037352e6a7067)

静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问.

### 静态内部类

静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围内，但是静态内部类却没有。没有这个引用就意味着：

1. 它的创建是不需要依赖外围类的创建。
2. 它不能使用任何外围类的非static成员变量和方法。

Example（静态内部类实现单例模式）

```
public class Singleton {
    
    声明为 private 避免调用默认构造方法创建对象
    private Singleton() {
    }
    
    声明为 private 表明静态内部该类只能在该 Singleton 类中被访问
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

当 Singleton 类加载时，静态内部类 SingletonHolder 没有被加载进内存。只有当调用 `getUniqueInstance() `方法从而触发 `SingletonHolder.INSTANCE` 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。

这种方式不仅具有延迟初始化的好处，而且由 JVM 提供了对线程安全的支持。

### 静态导包

格式为：import static

这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法

```
  Math. --- 将Math中的所有静态资源导入，这时候可以直接使用里面的静态方法，而不用通过类名进行调用
  如果只想导入单一某个静态方法，只需要将换成对应的方法名即可
 
import static java.lang.Math.;

  换成import static java.lang.Math.max;具有一样的效果
 
public class Demo {
	public static void main(String[] args) {
 
		int max = max(1,2);
		System.out.println(max);
	}
}
```

## 补充内容

### 静态方法与非静态方法

静态方法属于类本身，非静态方法属于从该类生成的每个对象。 如果您的方法执行的操作不依赖于其类的各个变量和方法，请将其设置为静态（这将使程序的占用空间更小）。 否则，它应该是非静态的。

Example

```
class Foo {
    int i;
    public Foo(int i) { 
       this.i = i;
    }

    public static String method1() {
       return An example string that doesn't depend on i (an instance variable);
       
    }

    public int method2() {
       return this.i + 1;  Depends on i
    }

}
```

你可以像这样调用静态方法：`Foo.method1（）`。 如果您尝试使用这种方法调用 method2 将失败。 但这样可行：`Foo bar = new Foo（1）;bar.method2（）;`

总结：

- 在外部调用静态方法时，可以使用”类名.方法名”的方式，也可以使用”对象名.方法名”的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。
- 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），而不允许访问实例成员变量和实例方法；实例方法则无此限制

### static{}静态代码块与{}非静态代码块（构造代码块）

相同点： 都是在JVM加载类时且在构造方法执行之前执行，在类中都可以定义多个，定义多个时按定义的顺序执行，一般在代码块中对一些static变量进行赋值。

不同点： 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。静态代码块只在第一次new执行一次，之后不再执行，而非静态代码块在每new一次就执行一次。 非静态代码块可在普通方法中定义(不过作用不大)；而静态代码块不行。

一般情况下,如果有些代码比如一些项目最常用的变量或对象必须在项目启动的时候就执行的时候,需要使用静态代码块,这种代码是主动执行的。如果我们想要设计不需要创建对象就可以调用类中的方法，例如：Arrays类，Character类，String类等，就需要使用静态方法, 两者的区别是 静态代码块是自动执行的而静态方法是被调用的时候才执行的.

Example

```
public class Test {
    public Test() {
        System.out.print(默认构造方法！--);
    }

     非静态代码块
    {
        System.out.print(非静态代码块！--);
    }
     静态代码块
    static {
        System.out.print(静态代码块！--);
    }

    public static void test() {
        System.out.print(静态方法中的内容! --);
        {
            System.out.print(静态方法中的代码块！--);
        }

    }
    public static void main(String[] args) {

        Test test = new Test();   
        Test.test();静态代码块！--静态方法中的内容! --静态方法中的代码块！--
    }
```

当执行 `Test.test();` 时输出：

```
静态代码块！--静态方法中的内容! --静态方法中的代码块！--
```

当执行 `Test test = new Test();` 时输出：

```
静态代码块！--非静态代码块！--默认构造方法！--
```

非静态代码块与构造函数的区别是： 非静态代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，构造代码块中定义的是不同对象共性的初始化内容。





# String和StringBuffer、StringBuilder

**可变性** 　

简单的来说：String 类中使用 final 关键字字符数组保存字符串，`private　final　char　value[]`，所以 String 对象是不可变的。而StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串`char[]value` 但是没有用 final 关键字修饰，所以这两种对象都是可变的。

StringBuilder 与 StringBuffer 的构造方法都是调用父类构造方法也就是 AbstractStringBuilder 实现的，大家可以自行查阅源码。

AbstractStringBuilder.java

```
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;
    AbstractStringBuilder() {
    }
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
```

**线程安全性**

String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。 　　

**性能**

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象。StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用 StirngBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

**对于三者使用的总结：**

1. 操作少量的数据 = String
2. 单线程操作字符串缓冲区下操作大量数据 = StringBuilder
3. 多线程操作字符串缓冲区下操作大量数据 = StringBuffer

## String不可变

简单来说就是String类利用了final修饰的char类型数组存储字符，源码如下图所以：

```
    /** The value is used for character storage. */
    private final char value[];
```

**String真的是不可变的吗？**

我觉得如果别人问这个问题的话，回答不可变就可以了。 下面只是给大家看两个有代表性的例子：

**1) String不可变但不代表引用不可以变**

```
		String str = "Hello";
		str = str + " World";
		System.out.println("str=" + str);
```

结果：

```
str=Hello World
```

解析：

实际上，原来String的内容是不变的，只是str由原来指向"Hello"的内存地址转为指向"Hello World"的内存地址而已，也就是说多开辟了一块内存区域给"Hello World"字符串。

**2) 通过反射是可以修改所谓的“不可变”对象**

```
		// 创建字符串"Hello World"， 并赋给引用s
		String s = "Hello World";

		System.out.println("s = " + s); // Hello World

		// 获取String类中的value字段
		Field valueFieldOfString = String.class.getDeclaredField("value");

		// 改变value属性的访问权限
		valueFieldOfString.setAccessible(true);

		// 获取s对象上的value属性的值
		char[] value = (char[]) valueFieldOfString.get(s);

		// 改变value所引用的数组中的第5个字符
		value[5] = '_';

		System.out.println("s = " + s); // Hello_World
```

结果：

```
s = Hello World
s = Hello_World
```

解析：

用反射可以访问私有成员， 然后反射出String对象中的value属性， 进而改变通过获得的value引用改变数组的结构。但是一般我们不会这么做，这里只是简单提一下有这个东西。

## 反射机制

### 介绍

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

### 静态编译和动态编译

- **静态编译：**在编译时确定类型，绑定对象
- **动态编译：**运行时确定类型，绑定对象

### 优缺点

- **优点：** 运行期类型的判断，动态加载类，提高代码灵活度。
- **缺点：** 性能瓶颈：反射相当于一系列解释操作，通知 JVM 要做的事情，性能比直接的java代码要慢很多。

### 应用场景

反射是框架设计的灵魂。

在我们平时的项目开发过程中，基本上很少会直接使用到反射机制，但这不能说明反射机制没有用，实际上有很多设计、开发都与反射机制有关，例如模块化的开发，通过反射去调用对应的字节码；动态代理设计模式也采用了反射机制，还有我们日常使用的 Spring／Hibernate 等框架也大量使用到了反射机制。

举例：①我们在使用JDBC连接数据库时使用Class.forName()通过反射加载数据库的驱动程序；②Spring框架也用到很多反射机制，最经典的就是xml的配置模式。Spring 通过 XML 配置模式装载 Bean 的过程：1) 将程序内所有 XML 或 Properties 配置文件加载入内存中; 2)Java类里面解析xml或properties里面的内容，得到对应实体类的字节码字符串以及相关的属性信息; 3)使用反射机制，根据这个字符串获得某个类的Class实例; 4)动态配置实例的属性

## JDK,JRE, JVM

这几个是Java中很基本很基本的东西，但是我相信一定还有很多人搞不清楚！为什么呢？因为我们大多数时候在使用现成的编译工具以及环境的时候，并没有去考虑这些东西。

**JDK:** 顾名思义它是给开发者提供的开发工具箱,是给程序开发者用的。它除了包括完整的JRE（Java Runtime Environment），Java运行环境，还包含了其他供开发者使用的工具包。

**JRE:** 普通用户而只需要安装JRE（Java Runtime Environment）来运行Java程序。而程序开发者必须安装JDK来编译、调试程序。

**JVM：** 当我们运行一个程序时，JVM负责将字节码转换为特定机器代码，JVM提供了内存管理/垃圾回收和安全机制等。这种独立于硬件和操作系统，正是java程序可以一次编写多处执行的原因。

**区别与联系：**

1. JDK用于开发，JRE用于运行java程序 ；
2. JDK和JRE中都包含JVM ；
3. JVM是java编程语言的核心并且具有平台独立性。

## 字节码

**先看下java中的编译器和解释器：** 　　

Java中引入了虚拟机的概念，即在机器和编译程序之间加入了一层抽象的虚拟的机器。这台虚拟的机器在任何平台上都提供给编译程序一个的共同的接口。编译程序只需要面向虚拟机，生成虚拟机能够理解的代码，然后由解释器来将虚拟机代码转换为特定系统的机器码执行。在Java中，这种供虚拟机理解的代码叫做`字节码`（即扩展名为`.class`的文件），它不面向任何特定的处理器，只面向虚拟机。每一种平台的解释器是不同的，但是实现的虚拟机是相同的。Java源程序经过编译器编译后变成字节码，字节码由虚拟机解释执行，虚拟机将每一条要执行的字节码送给解释器，解释器将其翻译成特定机器上的机器码，然后在特定的机器上运行。这也就是解释了Java的编译与解释并存的特点。

Java源代码---->编译器---->jvm可执行的Java字节码(即虚拟指令)---->jvm---->jvm中解释器----->机器可执行的二进制机器码---->程序运行。

**采用字节码的好处：** 　　

Java语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以Java程序运行时比较高效，而且，由于字节码并不专对一种特定的机器，因此，Java程序无须重新编译便可在多种不同的计算机上运行。

## Java和C++的区别

我知道很多人没学过C++，但是面试官就是没事喜欢拿咱们Java和C++比呀！没办法！！！就算没学过C++，也要记下来！

- 都是面向对象的语言，都支持封装、继承和多态
- Java不提供指针来直接访问内存，程序内存更加安全
- Java的类是单继承的，C++支持多重继承；虽然Java的类不可以多继承，但是接口可以多继承。
- Java有自动内存管理机制，不需要程序员手动释放无用内存

## 接口和抽象类的区别

1. 接口的方法默认是public，所有方法在接口中不能有实现，抽象类可以有非抽象的方法
2. 接口中的实例变量默认是final类型的，而抽象类中则不一定
3. 一个类可以实现多个接口，但最多只能实现一个抽象类
4. 一个类实现接口的话要实现接口的所有方法，而抽象类不一定
5. 接口不能用new实例化，但可以声明，但是必须引用一个实现该接口的对象 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

## 成员变量与局部变量的区别

1. 从语法形式上，看成员变量是属于类的，而局部变量是在方法中定义的变量或是方法的参数；成员变量可以被public,private,static等修饰符所修饰，而局部变量不能被访问控制修饰符及static所修饰；但是，成员变量和局部变量都能被final所修饰；
2. 从变量在内存中的存储方式来看，成员变量是对象的一部分，而对象存在于堆内存，局部变量存在于栈内存
3. 从变量在内存中的生存时间上看，成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失。
4. 成员变量如果没有被赋初值，则会自动以类型的默认值而赋值（一种情况例外被final修饰但没有被static修饰的成员变量必须显示地赋值）；而局部变量则不会自动赋值。

## 重载和重写的区别

**重载：** 发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同，发生在编译时。 　　

**重写：** 发生在父子类中，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类；如果父类方法访问修饰符为private则子类就不能重写该方法。

## 字符型常量和字符串常量的区别

1. 形式上: 字符常量是单引号引起的一个字符 字符串常量是双引号引起的若干个字符
2. 含义上: 字符常量相当于一个整形值(ASCII值),可以参加表达式运算 字符串常量代表一个地址值(该字符串在内存中存放位置)
3. 占内存大小 字符常量只占一个字节 字符串常量占若干个字节(至少一个字符结束标志)

## 为什么 Java 中只有值传递？

首先回顾一下在程序设计语言中有关将参数传递给方法（或函数）的一些专业术语。**按值调用(call by value)表示方法接收的是调用者提供的值，而按引用调用（call by reference)表示方法接收的是调用者提供的变量地址。一个方法可以修改传递引用所对应的变量值，而不能修改传递值调用所对应的变量值。** 它用来描述各种程序设计语言（不只是Java)中方法参数传递方式。

**Java程序设计语言总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容。**

**下面通过 3 个例子来给大家说明**

**example 1**

```
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;

    swap(num1, num2);

    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;

    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```

**结果：**

```
a = 20
b = 10
num1 = 10
num2 = 20
```

**解析：**

[![example 1 ](https://camo.githubusercontent.com/ab46506b1a5ce09a516051c35f981e55255337f9/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f32323139313334382e6a7067)](https://camo.githubusercontent.com/ab46506b1a5ce09a516051c35f981e55255337f9/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f32323139313334382e6a7067)

在swap方法中，a、b的值进行交换，并不会影响到 num1、num2。因为，a、b中的值，只是从 num1、num2 的复制过来的。也就是说，a、b相当于num1、num2 的副本，副本的内容无论怎么修改，都不会影响到原件本身。

**通过上面例子，我们已经知道了一个方法不能修改一个基本数据类型的参数，而对象引用作为参数就不一样，请看 example2.**

**example 2**

```
	public static void main(String[] args) {
		int[] arr = { 1, 2, 3, 4, 5 };
		System.out.println(arr[0]);
		change(arr);
		System.out.println(arr[0]);
	}

	public static void change(int[] array) {
		// 将数组的第一个元素变为0
		array[0] = 0;
	}
```

**结果：**

```
1
0
```

**解析：**

[![example 2](https://camo.githubusercontent.com/b7bad9506150c29bb8d7debd3905bd7a71cd6611/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f333832353230342e6a7067)](https://camo.githubusercontent.com/b7bad9506150c29bb8d7debd3905bd7a71cd6611/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f333832353230342e6a7067)

array 被初始化 arr 的拷贝也就是一个对象的引用，也就是说 array 和 arr 指向的时同一个数组对象。 因此，外部对引用对象的改变会反映到所对应的对象上。

**通过 example2 我们已经看到，实现一个改变对象参数状态的方法并不是一件难事。理由很简单，方法得到的是对象引用的拷贝，对象引用及其他的拷贝同时引用同一个对象。**

**很多程序设计语言（特别是，C++和Pascal)提供了两种参数传递的方式：值调用和引用调用。有些程序员（甚至本书的作者）认为Java程序设计语言对对象采用的是引用调用，实际上，这种理解是不对的。由于这种误解具有一定的普遍性，所以下面给出一个反例来详细地阐述一下这个问题。**

**example 3**

```
public class Test {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Student s1 = new Student("小张");
		Student s2 = new Student("小李");
		Test.swap(s1, s2);
		System.out.println("s1:" + s1.getName());
		System.out.println("s2:" + s2.getName());
	}

	public static void swap(Student x, Student y) {
		Student temp = x;
		x = y;
		y = temp;
		System.out.println("x:" + x.getName());
		System.out.println("y:" + y.getName());
	}
}
```

**结果：**

```
x:小李
y:小张
s1:小张
s2:小李
```

**解析：**

交换之前：

[![img](https://camo.githubusercontent.com/9d6dd0313695d309280675cd3251b47432a28814/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f38383732393831382e6a7067)](https://camo.githubusercontent.com/9d6dd0313695d309280675cd3251b47432a28814/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f38383732393831382e6a7067)

交换之后：

[![img](https://camo.githubusercontent.com/6bea9b0ed65609d699207ab787f631f7ba0a9246/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f33343338343431342e6a7067)](https://camo.githubusercontent.com/6bea9b0ed65609d699207ab787f631f7ba0a9246/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f33343338343431342e6a7067)

通过上面两张图可以很清晰的看出： **方法并没有改变存储在变量 s1 和 s2 中的对象引用。swap方法的参数x和y被初始化为两个对象引用的拷贝，这个方法交换的是这两个拷贝**

**总结**

Java程序设计语言对对象采用的不是引用调用，实际上，对象引用是按 值传递的。

下面再总结一下Java中方法参数的使用情况：

- 一个方法不能修改一个基本数据类型的参数（即数值型或布尔型》
- 一个方法可以改变一个对象参数的状态。
- 一个方法不能让对象参数引用一个新的对象。