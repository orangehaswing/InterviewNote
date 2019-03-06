# Java基础

## 第三章 操作符

## 第四章 控制执行流程







## 第五章 初始化与清理

​	构造器

​		无参构造器

​		有参构造器

​		无返回值

​		构造器采用与类相同的名称





​	

## 重载和重写的区别

**重载：** 发生在同一个类中，方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回值和访问修饰符可以不同，发生在编译时。 　　

**重写：** 发生在父子类中，方法名、参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类；如果父类方法访问修饰符为private则子类就不能重写该方法。

​	传入数据类型（实际参数类型）小于方法中声明的形式参数类型，实际数据类型就会被提升。char型略有不同，无法找到恰好接收char参数的方法，就会把char直接提升至int型。如果传入实际参数较大，就通过类型转换来执行窄化转换。

this关键字

1、方法内部使用，表示对‘调用方法的那个对象’的引用。但要注意，如果在方法内部调用同一个类的另一个方法。

2、一个类可以写多个构造器，在一个构造器调用另一个构造器，以避免重复代码。

3、类中置入static方法可以访问其他static方法和static域。



清理垃圾回收

finalize();

1、垃圾回收只与内存有关，分配内存时可能采用类似C语言中的做法。在使用本地方法情况下，一种在Java中调用非Java代码的情况（本地方法调用C、C++）。

2、对象可能不被垃圾回收

3、垃圾回收并不等于构析

注：不要过多使用finalize()



成员、构造器初始化



数据初始化

数据成员的初值没有给出，但是他们确实有初值。对一个对象引用，如果不将其初始化，此引用就会获得一个特殊值null

 

无法阻止自动初始化的进行，它将在构造器被调用之前发生。

变量定义的先后顺序决定了初始化的顺序。即使变量定义散布于方法定义之间，它们仍旧会在任何方法（包括构造器）被调用之前得到初始化。

 

初始化的顺序是先静态对象，而后是非静态对象。

编译器都会使用自动包装机制来匹配重载的方法，然后调用最明确匹配的方法。



## 第六章 访问控制权限

权限排序：public、protected、包访问权限（没有关键字）private

 

Public：接口访问权限，public之后紧跟的成员申明自己对每个人都是可用的。

Private：无法访问，除了包含该成员的类之外，其他任何类都无法访问这个成员。

Protected：继承访问权限，也提供包访问权限，相同包内的其他类可以访问protected元素。

 

限制：

每个编译单元都只能有一个public类

Public类的名称必须完全与含有该编译单元的文件名相匹配，包括大小写。

若编译单元完全不带public类，可以随意对文件命名。

 

类既不可以是private的，也不可以是protected。

对类访问权限，仅有两个选项：包访问权限、public





## 第七章 复用类

向上转型：导出类转型成基类，在继承图上是向上移动。从一个较为专用类型向较通用类型转换，所以总是安全的。

toString():每一个非基本类型的对象都有一个toString()方法，而且当编译器需要一个String而却只有一个对象时，该方法便会被调用。 

final关键字

对基本类型：final使数值恒定不变

对对象引用：final使引用恒定不变，一旦引用被初始化指向一个对象，就无法再把它改为指向另一个对象。然而，对象其自身却是可以被修改的。	

空白final：被声明为final但又未给定初值的域。

final参数：在参数列表中以声明的方式将参数指向为final，这意味着无法在方法中更改参数所引用所指向的对象。

final方法：1、把方法锁定，以防任何继承类修改它的含义。2、效率，性能提高。

final和private关键字：类中所有的private方法都隐式地指定位final。由于无法取用private方法，所以也就无法覆盖它。

final类：当将某个类的整体定义为final时，就表明你不打算继承该类，而且也不允许别人这样做。

### **final**

**1. 数据**

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

```
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

**2. 方法** 

声明方法不能被子类覆盖。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是覆盖基类方法，而是在子类中定义了一个新的方法。

**3. 类**

声明类不允许被继承。

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

### 继承

### **访问权限**

Java 中有三个访问权限修饰符：private、protected 以及 public，如果不加访问修饰符，表示包级可见。

可以对类或类中的成员（字段以及方法）加上访问修饰符。

- 成员可见表示其它类可以用这个类的实例访问到该成员；
- 类可见表示其它类可以用这个类创建对象。

protected 用于修饰成员，表示在继承体系中成员对于子类可见，但是这个访问修饰符对于类没有意义。

设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。

如果子类的方法覆盖了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足里氏替换原则。

字段决不能是公有的，因为这么做的话就失去了对这个字段修改行为的控制，客户端可以对其随意修改。可以使用公有的 getter 和 setter 方法来替换公有字段。

```
public class AccessExample {
    public int x;
}
```

```
public class AccessExample {
    private int x;

    public int getX() {
        return x;
    }

    public void setX(int x) {
        this.x = x;
    }
}
```

但是也有例外，如果是包级私有的类或者私有的嵌套类，那么直接暴露成员不会有特别大的影响。

```
public class AccessWithInnerClassExample {
    private class InnerClass {
        int x;
    }

    private InnerClass innerClass;

    public AccessWithInnerClassExample() {
        innerClass = new InnerClass();
    }

    public int getValue() {
        return innerClass.x; // 直接访问
    }
}
```

### **覆盖与重载**

- 覆盖（Override）存在于继承体系中，指子类实现了一个与父类在方法声明上完全相同的一个方法；
- 重载（Overload）存在于同一个类中，指一个方法与已经存在的方法名称上相同，但是参数类型、个数、顺序至少有一个不同。应该注意的是，返回值不同，其它都相同不算是重载。

### **static**

**1. 静态变量**

静态变量在内存中只存在一份，只在类初始化时赋值一次。

- 静态变量：类所有的实例都共享静态变量，可以直接通过类名来访问它；
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。

```
public class A {
    private int x;        // 实例变量
    public static int y;  // 静态变量
}
```

**2. 静态方法**

静态方法在类加载的时候就存在了，它不依赖于任何实例，所以 static 方法必须实现，也就是说它不能是抽象方法（abstract）。

**3. 静态语句块**

静态语句块在类初始化时运行一次。

**4. 静态内部类**

内部类的一种，静态内部类不依赖外部类，且不能访问外部类的非 static 变量和方法。

**5. 静态导包**

```
import static com.xxx.ClassName.*
```

在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

**6. 变量赋值顺序**

静态变量的赋值和静态语句块的运行优先于实例变量的赋值和普通语句块的运行，静态变量的赋值和静态语句块的运行哪个先执行取决于它们在代码中的顺序。

```
public static String staticField = "静态变量";
```

```
static {
    System.out.println("静态语句块");
}
```

```
public String field = "实例变量";
```

```
{
    System.out.println("普通语句块");
}
```

最后才运行构造函数

```
public InitialOrderTest() {
    System.out.println("构造函数");
}
```

存在继承的情况下，初始化顺序为：

1. 父类（静态变量、静态语句块）

2. 子类（静态变量、静态语句块）

3. 父类（实例变量、普通语句块）

4. 父类（构造函数）

5. 子类（实例变量、普通语句块）

6. 子类（构造函数）

   ​

初始化的实际顺序:

1、 在其他任何事物发生之前，将分配给对象的存储空间初始化成二进制的零

2、 调用基类构造器，反复迭代，首先构造层次结构的根，然后是下一层导出类

3、 按照声明的顺序调用成员的初始化方法

4、 调用导出类的构造器主体

 

向下转型：所有类型都会得到检查。

类转型异常（ClassCastException）



## 第八章 多态

Java所有方法都是通过动态绑定实现多态。

静态方法是与类，而非与单个对象相关联的。

构造器不同于其他种类的方法（static方法，只是static声明是隐式的）。

基类的构造器总是在导出类的构造过程中被调用，而且按照继承层次逐渐向上链接，以使每个基类的构造器都能得到调用。



构造器的调用顺序：

1、调用基类构造器，这个步骤会反复递归，首先是构造这种层次结构的根，然后是下一层导出类。直到最底层的导出类。

2、按声明顺序调用成员的初始化方法。

3、调用导出类构造器的主体。

### **泛型**

```
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

> [Java 泛型详解](https://www.ziwenxie.site/2017/03/01/java-generic/)
> [10 道 Java 泛型面试题](https://cloud.tencent.com/developer/article/1033693)

## 第九章 接口

抽象方法：仅有声明而没有方法体

抽象类：包含抽象方法

接口：允许在类中创建一个或多个没有任何定义的方法，但是没有提供任何相应的具体实现，这些实现是由此类的继承者创建的

### 抽象类

特点: 

1. 抽象类中可以构造方法 
2. 抽象类中可以存在普通属性，方法，静态属性和方法。 
3. 抽象类中可以存在抽象方法。 
4. 如果一个类中有一个抽象方法，那么当前类一定是抽象类；抽象类中不一定有抽象方法。
5. 抽象类中的抽象方法，需要有子类实现，如果子类不实现，则子类也需要定义为抽象的。 
6. 抽象类不能被实例化，抽象类和抽象方法必须被abstract修饰

关键字使用注意： 

抽象类中的抽象方法（其前有abstract修饰）不能用private、static、synchronized、native访问修饰符修饰。

原因如下：

- 抽象方法没有方法体，是用来被继承的，所以不能用private修饰；
- static修饰的方法可以通过类名来访问该方法（即该方法的方法体），抽象方法用static修饰没有意义；
- 使用synchronized关键字是为该方法加一个锁。而如果该关键字修饰的方法是static方法。则使用的锁就是class变量的锁。如果是修饰类方法。则用this变量锁。但是抽象类不能实例化对象，因为该方法不是在该抽象类中实现的。是在其子类实现的。所以。锁应该归其子类所有。所以。抽象方法也就不能用synchronized关键字修饰了；
- native，这个东西本身就和abstract冲突，他们都是方法的声明，只是一个把方法实现移交给子类，另一个是移交给本地操作系统。如果同时出现，就相当于即把实现移交给子类，又把实现移交给本地操作系统，那到底谁来实现具体方法呢？

### 接口

1. 在接口中只有方法的声明，没有方法体。 
2. 在接口中只有常量，因为定义的变量，在编译的时候都会默认加上public static final
3. 在接口中的方法，永远都被public来修饰。 
4. 接口中没有构造方法，也不能实例化接口的对象。（所以接口不能继承类） 
5. 接口可以实现多继承接口，用extends
6. 接口中定义的方法都需要有实现类来实现，如果实现类不能实现接口中的所有方法则实现类定义为抽象类。

接口中的变量默认是public static final 的，方法默认是public abstract 的

接口是一种特殊的抽象类，接口中的方法全部是抽象方法（但其前的abstract可以省略），所以抽象类中的抽象方法不能用的访问修饰符这里也不能用。而且protected访问修饰符也不能使用，因为接口可以让所有的类去实现（非继承），不只是其子类，但是要用public去修饰。接口可以去继承一个已有的接口。

## 接口和抽象类的区别

1. 接口的方法默认是public，所有方法在接口中不能有实现，抽象类可以有非抽象的方法
2. 接口中的实例变量默认是final类型的，而抽象类中则不一定
3. 一个类可以实现多个接口，但最多只能实现一个抽象类
4. 一个类实现接口的话要实现接口的所有方法，而抽象类不一定
5. 接口不能用new实例化，但可以声明，但是必须引用一个实现该接口的对象 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

## 第十章 内部类

内部类：将一个类的定义放在另一个类的定义内部

需要生成对外部类对象的引用，可以使用外部类的名字后面紧跟圆点和this(return class.this)

告知某些其他对象，去创建其某个内部类的对象，使用.new语法。（DotNew.Inner dni = dn.new Inner();）

创建内部类的对象，必须使用外部类的对象来创建该内部类对象



每个内部类都能独立地继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。内部类有效地实现了多重继承。



## 第十一章 持有对象

容器类保存对象，基本类型是List, Set, Queue, Map

应用预定义的泛型，声明 `ArrayList<Apple>` ，而不仅仅是ArrayList，其中尖括号括起来的是类型参数（可以有多个），它指定了这个容器实例可以保存的类型。

Java容器类类库的用途是保存对象，并将其划分为两个不同的概念：

1、Collection，一个独立的元素序列。List必须按照插入顺序保存元素，而Set不能有重复元素。Queue按照队列规则来确定对象产生的顺序。

2、Map，一组成对的键值对对象。允许使用键来查找值。

ArrayList允许使用数字来查找值。

LinkedList：还添加可以使其作栈，队列或者双端队列的方法。

Stack：LinkedList具有能够直接实现栈功能的方法，因此可以直接将LinkedList作为栈使用。

Stack是用LinkedList实现的。

Set：不保存重复的元素，Set最常被用到的是测试归属性，可以很容易地询问某个对象是否在某个Set中。Set具有与Collection完全一样的接口，因此没有任何额外的功能，只是行为不同。

Map：Map与数组和其他的Collection一样，可以很容易地扩展到多维，而我们只需将其值设置为Map（这些Map的值可以是其他容器，甚至是其他Map）。

Map可以返回它的键的Set，它的值的Collection，或者是它的键值对的Set。

 

Queue

队列是一个典型的先进先出的容器，即从容器的一端放入事物，从另一端取出，并且事物放入容器的顺序与取出的顺序是相同的。

 

Foreach

Foreach语法主要用于数组，但是它也是可以应用于任何Collection对象。

 

容器：Map，List，Set，Queue

常用容器：HashMap，ArrayList，LinkedList，HashSet

添加一组元素

ArrayList.asList()方法接受一个数组或是一个用逗号分隔的元素列表，并将其转换成一个List对象。Collection.addAll()方法接收一个Collection对象，以及一个数组或是一个用逗号分隔的列表，并将元素添加到Collection中。



容器的打印

使用Arrays.toString()产生数组的可打印表示

Collection打印出来的内容用方括号括住，每个元素由逗号分隔。Map则用大括号括住，键和值由等号联系。



List类型：ArrayList和LinkedList

基本的ArrayList：擅长于随机访问元素，但是在List的中间插入和移除元素时较慢

LinkedList：通过代价较低的List中间进行的插入和删除操作，提供了优化的顺序访问。在随机访问方面相对较慢，但它的特性集较ArrayList更大。



Set类型：HashSet  TreeSet 和LinkedHashSet

Map类型：HashMap  TreeMap  和LinkedHashMap



迭代器

迭代器是一个对象，它的工作是遍历并选择序列的对象，而客户端程序员不必知道或关系该序列底层的结构。

Iterator只能单向移动。只能用来：

1、使用方法iterator()要求容器返回一个Iterator。Iterator准备好返回序列的第一个元素。

2、使用next()湖区序列中的下一个元素。

3、使用hasNext()见长序列中是否还有元素

4、使用remove()将迭代器新近返回的元素删除。



ListIterator迭代

只能用于各种List访问，可以双向移动。还可以产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引，并且可以使用set()方法替换它访问过的最后一个元素。





## 第十二章 通过异常处理错误

## Java 中的异常处理

### Java异常类层次结构图

![Java异常类层次结构图](https://camo.githubusercontent.com/e6b358c7ceca03c7128d52004c45a746728c76f4/687474703a2f2f696d61676573323031352e636e626c6f67732e636f6d2f626c6f672f3634313030332f3230313630372f3634313030332d32303136303730363233323034343238302d3335353335343739302e706e67)

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



使用finally进行清理：

希望无论try块中的异常是否抛出，它们都能得到执行。可以在异常处理程序后面加上finally子句。

清理资源包括：已经打开的文件或网络连接，在屏幕上画的图形，外部世界的某个开关。



异常匹配

抛出异常的时候，异常处理系统会按照代码的书写顺序找出最近的处理程序，找到匹配的处理程序后，它就认为异常得到处理，然后不再继续查找。



异常最重要的方面之一就是如果发生问题，它们将不允许程序沿着其正常的路径继续走下去。异常允许我们强制程序停止运行。

异常类型的根，Throwable可以爆出任何类型。

 

异常处理基本模型：

终止模型，将假设错误非常关键，以至于程序无法返回到异常发生的地方继续执行。

恢复模型，异常处理程序的工作是修正错误，然后重新尝试调用出问题的方法，并认为第二次能成功。

 

创建自定义异常

必须从已有的异常类继承，最好选择意思相近的异常类继承。

 

捕获所有异常

通过捕获异常类型的积累Exception，就可以捕获所有异常。

```
Catch(Exception e){
	Sout();
}

```



异常对象都是用new在堆上创建的对象，所以垃圾回收器会自动把它们清理掉。

 

异常链：常常会想要在捕获一个异常后抛出另一个异常，并且希望把原始异常的信息保存下来。在Throwable的子类中，只有三种基本的异常类型提供了带cause参数的构造器。它们是Error，Exception，RuntimeException。



## 第十三章 字符串

### **String**

### **String,  StringBuffer and StringBuilder**

**1. 是否可变**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 是否线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 来同步

### **String 不可变的原因**

**可以缓存 hash 值**

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/f76067a5-7d5f-4135-9549-8199c77d8f1c.jpg)

**安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

### **String.intern()**

使用 String.intern() 可以保证相同内容的字符串实例引用相同的内存对象。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同对象，而 s3 是通过 s1.intern() 方法取得一个对象引用，这个方法首先把 s1 引用的对象放到 String Pool（字符串常量池）中，然后返回这个对象引用。因此 s3 和 s1 引用的是同一个字符串常量池的对象。

```
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
System.out.println(s1.intern() == s3);  // true
```

如果是采用 "bbb" 这种使用双引号的形式创建字符串实例，会自动地将新建的对象放入 String Pool 中。

```
String s4 = "bbb";
String s5 = "bbb";
System.out.println(s4 == s5);  // true
```

在 Java 7 之前，字符串常量池被放在运行时常量池中，它属于永久代。而在 Java 7，字符串常量池被放在堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。



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

String对象是不可变的。

用于String的‘+’与‘+=’是Java中仅有的两个重载过的操作符，而Java并不允许程序员重载任何操作符。

当需要改变字符串的内容时，String类的方法都会返回一个新的String对象。同时，如果内容没有发生变化，String类的方法只能返回指向原对象的引用。



正则表达式

String类自带正则表达式工具-split()方法



在编写新的表达式之前，通常会参考代码中已经用到的正则表达式

Pattern和Matcher：根据String类型的正则表达式生成一个Pattern对象。接下来，把想要检索的字符串传入Pattern对象的matcher()方法，生成一个Matcher对象。

find()可以在输入的任意位置定位正则表达式

lookingAt()只要输入的第一部分匹配就会成功

matches()只有在整个输入都匹配正则表达式时才会成功





## 第四十章 类型信息

Java让我们在运行时识别对象和类的信息，两种方法：1、RTTI，假定我们在编译时知道了所有的类型。2、反射机制，它允许我们在运行时发现和使用类的信息。

所有的类都是在对其第一次使用时，动态加载到JVM中。

Class.forName()是Class类的一个static成员。取得Class对象引用的一种方法。

类字面常量：FancyToy.class。对于基本数据类型的包装器类，还有一个标准字段TYPE。TYPE字段是一个引用，指向对应的基本数据类型的Class对象

boolean.class 等价于 Boolean.TYPE

char.class 等价于 Character.TYPE

......

为了使用类而做的准备工作实际包含三个步骤：

1、加载，将查找字节码，并从这些字节码中创建一个Class对象。

2、链接，验证类中的字节码，为静态域分配存储空间，并且如果必需的话，将解析这个类创建的对其他类的所有引用。

3、初始化，如果该类具有超类，则对其初始化，执行静态初始化器和静态初始化块。



为了在使用泛化的Class引用时放松限制，使用通配符，它是Java泛型的一部分。通配符就是‘？’，表示任何事务。

为创建一个Class引用，它被限定为某种类型，或者该类型的任何子类型，需要将通配符与extends关键字相结合，创建一个范围。Class<? extends Number> bounded = int.class;

如果是超类，编译器只允许声明超类的引用时‘某个类，它是FancyToy超类’，就像在表达式Class<? Super FancyToy>中看到的，而不会接受



RTTI形式：

1、传统的类型转换，如shape，由RTTI确保类型转换的正确性，如果执行了一个错误的类型转换，就会抛出一个ClassCastException异常。

2、代表对象的类型的Class对象。通过查询Class对象可以获取运行时所需的信息。

3、关键字instanceof，它返回一个布尔值，告诉我们对象是不是某个特定类型的实例



反射

Class类和java.lang.reflect类库一起对反射的概念进行了支持。

类：Field，Method，Constructor

使用Constructor创建对象，使用get()和set()方法读取和修改与Filed对象关联的字段。

用invoke()方法调用与Method对象相关联的方法

调用getFileds(), getMethod()和getContructors()的方法，以返回表示字段、方法以及构造器的对象数组

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

### **反射**

每个类都有一个 **Class** 对象，包含了与类有关的信息。当编译一个新类时，会产生一个同名的 .class 文件，该文件内容保存着 Class 对象。

类加载相当于 Class 对象的加载。类在第一次使用时才动态加载到 JVM 中，可以使用 Class.forName("com.mysql.jdbc.Driver") 这种方式来控制类的加载，该方法会返回一个 Class 对象。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

1. **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
2. **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
3. **Constructor** ：可以用 Constructor 创建新的对象。

IDE 使用反射机制获取类的信息，在使用一个类的对象时，能够把类的字段、方法和构造函数等信息列出来供用户选择。

**Advantages of Using Reflection:**

- **Extensibility Features** : An application may make use of external, user-defined classes by creating instances of extensibility objects using their fully-qualified names.
- **Class Browsers and Visual Development Environments** : A class browser needs to be able to enumerate the members of classes. Visual development environments can benefit from making use of type information available in reflection to aid the developer in writing correct code.
- **Debuggers and Test Tools** : Debuggers need to be able to examine private members on classes. Test harnesses can make use of reflection to systematically call a discoverable set APIs defined on a class, to insure a high level of code coverage in a test suite.

**Drawbacks of Reflection:**

Reflection is powerful, but should not be used indiscriminately. If it is possible to perform an operation without using reflection, then it is preferable to avoid using it. The following concerns should be kept in mind when accessing code via reflection.

- **Performance Overhead** : Because reflection involves types that are dynamically resolved, certain Java virtual machine optimizations can not be performed. Consequently, reflective operations have slower performance than their non-reflective counterparts, and should be avoided in sections of code which are called frequently in performance-sensitive applications.
- **Security Restrictions** : Reflection requires a runtime permission which may not be present when running under a security manager. This is in an important consideration for code which has to run in a restricted security context, such as in an Applet.
- **Exposure of Internals** :Since reflection allows code to perform operations that would be illegal in non-reflective code, such as accessing private fields and methods, the use of reflection can result in unexpected side-effects, which may render code dysfunctional and may destroy portability. Reflective code breaks abstractions and therefore may change behavior with upgrades of the platform.

Java 反射主要提供以下功能：

- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
- 在运行时调用任意一个对象的方法


## 反射

　　当我们的程序在运行时，需要动态的加载一些类这些类可能之前用不到所以不用加载到 JVM，而是在运行时根据需要才加载，这样的好处对于服务器来说不言而喻。

　　举个例子我们的项目底层有时是用 mysql，有时用 oracle，需要动态地根据实际情况加载驱动类，这个时候反射就有用了，假设 com.java.dbtest.myqlConnection，com.java.dbtest.oracleConnection 这两个类我们要用，这时候我们的程序就写得比较动态化，通过 Class tc = Class.forName("com.java.dbtest.TestConnection"); 通过类的全类名让 JVM 在服务器中找到并加载这个类，而如果是 Oracle 则传入的参数就变成另一个了。这时候就可以看到反射的好处了，这个动态性就体现出 Java 的特性了！

　　举个例子，大家如果接触过 spring，会发现当你配置各种各样的 bean 时，是以配置文件的形式配置的，你需要用到哪些 bean 就配哪些，spring 容器就会根据你的需求去动态加载，你的程序就能健壮地运行。

### 什么是反射

　　反射 (Reflection) 是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。通过 Class 获取 class 信息称之为反射（Reflection）

　　简而言之，通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。

　　程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。

　　反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

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



动态代理

调用静态方法Proxy.newProxyInstance()可以创建动态代理，这个方法需要一个类加载器，一个希望代理实现的接口列表，以及InvocationHandler接口的一个实现。

```
eg：Inerface proxy = (Interface)Proxy.newProxyInstance(
	Instance.class.getClassLoader(),
	new Class[]{Interface.class},
	new DynamicProxyHandler(real) );
```



## 第十五章 泛型

概念：实现参数化类型的概念，使代码可以应用于多种类型。

泛型接口：

泛型方法：要定义泛型方法，只需将泛型参数列表置于返回值之前。

泛型数组：在任何想要创建泛型数组的地方都使用ArrayList

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

### 多态

- 使用父类类型的引用指向子类的对象；
- 该引用只能调用父类中定义的方法和变量；
- 如果子类中重写了父类中的一个方法，那么在调用这个方法的时候，将会调用子类中的这个方法；（动态连接、动态调用）
- 变量不能被重写（覆盖），”重写“的概念只针对方法，如果在子类中”重写“了父类中的变量，那么在编译时会报错。

多态的3个必要条件：

        1.继承   2.重写   3.父类引用指向子类对象。

向上转型： Person p = new Man() ; //向上转型不需要强制类型转化

向下转型： Man man = (Man)new Person() ; //必须强制类型转化



构造函数不能被继承，构造方法只能被显式或隐式的调用。

### 通配符

List<? extends Fruit>:具有任何从Fruit继承的类型的列表

List<? super Fruit>:声明通配符是由某个特定类的任何基类来界定的，甚至使用类型参数:<? super T>

<?>:无界通配符，声明用Java的泛型来编写这段代码，这里并不是要用原生类型，但是在当前这种情况下，泛型参数可以持有任何类型。



## 第十六章 数组

数组与其他种类的容器之间的区别：效率、类型、保存基本类型的能力



Arrays类的使用功能：

equals()比较两个数组数否相等；fill()用于填充数组；sort()对数组排序；binarySearch()用于在已经排序对数组中查找元素；toString()产生数组对String表示；hashCode产生数组对散列码。Arrays.asList()接受任意序列或数组作为其参数，并将其转变为List容器。



数组与泛型：数组与泛型不能很好的结合，擦除会移除参数类型信息，而数组必须知道它们所持有的确切类型，以强制保证类型的安全。但是编译器允许创建对这种数组的引用

List<String>[] ls;

数组或对象填充数组：Arrays.fill()，只能用同一个值来填充各个位置，而针对对象而言，就是复制同一个引用进行填充。



## 第十七章 容器深入研究

## **容器中的设计模式**

### **迭代器模式**

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/Iterator-1.jpg)



Collection 实现了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。

从 JDK 1.5 之后可以使用 foreach 方法来遍历实现了 Iterable 接口的聚合对象。

```
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
for (String item : list) {
    System.out.println(item);
}
```

### **适配器模式**

java.util.Arrays#asList() 可以把数组类型转换为 List 类型。

```
@SafeVarargs
public static <T> List<T> asList(T... a)
```

如果要将数组类型转换为 List 类型，应该注意的是 asList() 的参数为泛型的变长参数，因此不能使用基本类型数组作为参数，只能使用相应的包装类型数组。

```
Integer[] arr = {1, 2, 3};
List list = Arrays.asList(arr);
```

也可以使用以下方式生成 List。

```
List list = Arrays.asList(1,2,3);
```

Set：存入set的每个元素必须是唯一的，不保存重复元素，加入Set的元素必须定义equals()方法以确保对象的唯一性。

HashSet：为快速查找设计，存入HashSet的元素必须定义hashCode()（没有其他限制，默认使用选项）

TreeSet：保持次序的Set，底层为树结构，从Set中提取有序的序列，元素必须实现Comparable接口

LinkedHashSet：具有HashSet的查询速度，迭代器遍历时，结果会按照元素的插入顺序显示，元素也必须定义hashCode()方法



Queue仅有的两个实现：LinkedList和PriorityQueue



Map的几种基本实现：HashMap，TreeMap，LinkedHashMap，WeakHashMap，ConcurrentHashMap，IdentityHashMap。

HashMap：插入和查询键值对的开销是固定的，可以通过构造器设置容量和负载因子，调整容器的性能

TreeMap：基于红黑树的实现，特点是结果是经过排序的。可以返回一个子树

LinkedHashMap：迭代遍历它时，取得的键值对的顺序是其插入次序，或者最近最少使用的次序。只比HashMap慢一点，而在迭代访问时反而更快，因为使用链表维护内部次序

WeakHashMap：弱键映射，允许释放映射所指向的对象。如果映射之外没有引用指向某个键，则此键可以被垃圾收集器回收

ConcurrentHashMap：一种线程安全的Map，不涉及同步加锁

IdentityHashMap：使用 == 代替equals()对键进行比较的散列映射

总结：任何键都必须具有一个equals()方法，如果键被用于散列Map，还必须具有恰当的hashCode()方法，被用于TreeMap，必须实现ComParable。

### **Object 通用方法**

**概览**

```
public final native Class<?> getClass()

public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException

protected void finalize() throws Throwable {}
```

**源码**

```
// 基于jdk1.8版本做了简化
public class Object {
    public final native Class<?> getClass();
    public native int hashCode();
    public boolean equals(Object obj) {
        return (this == obj);
    }
    protected native Object clone() throws CloneNotSupportedException;
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    public final native void notify();
    public final native void notifyAll();
    public final native void wait(long timeout) throws InterruptedException;
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
    public final void wait() throws InterruptedException {
        wait(0);
    }
    protected void finalize() throws Throwable { }
}
```

**方法详解**

getClass方法

**定义**

返回运行时对象的Class对象。

**官方注释**

> Returns the runtime class of this Object. The returned Class object is the object that is locked by static synchronized methods of the represented class.

**注意**

- getClass是一个final方法，所以不允许子类覆盖。
- getClass是一个native方法，即由非Java语言实现的Java方法（A native method is a Java method whose implementation is provided by non-java code.）。

hashCode方法

**定义**

返回对象的哈希码，主要用在哈希表中，比如HashMap。

**官方注释**

> Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by HashMap.

**哈希码的通用约定**

1. 在java程序执行过程中，在一个对象没有被改变的前提下，无论这个对象被调用多少次，hashCode方法都会返回相同的整数值。对象的哈希码没有必要在不同的程序中保持相同的值。
2. 如果2个对象使用equals方法进行比较并且相同的话，那么这2个对象的hashCode方法的值也必须相等。
3. 如果根据equals方法，得到两个对象不相等，那么这2个对象的hashCode值不需要必须不相同。但是，不相等的对象的hashCode值不同的话可以提高哈希表的性能。

**注意**

- hashCode是一个native方法。
- 通常情况下，不同的对象产生的哈希码是不同的。默认情况下，对象的哈希码是通过将该对象的内部地址转换成一个整数来实现的。

equals方法

**定义**

返回两个对象是否相等

**官方注释**

> Indicates whether some other object is "equal to" this one.

**equals方法在非空对象引用上的特性**

1. reflexive，自反性。任何非空引用值x，对于x.equals(x)必须返回true
2. symmetric，对称性。任何非空引用值x和y，如果x.equals(y)为true，那么y.equals(x)也必须为true
3. transitive，传递性。任何非空引用值x、y和z，如果x.equals(y)为true并且y.equals(z)为true，那么x.equals(z)也必定为true
4. consistent，一致性。任何非空引用值x和y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改

**注意**

- Object类中的equals方法使用的是==比较两个对象，而==符号比较的是对象在内存中的地址是否相同，所以equals方法默认比较的是对象在内存中的地址是否相同。
- 如果子类覆盖了equals方法，则也需要覆盖hashCode方法

**例子**

String类中覆盖了equals方法，同时也覆盖了hashCode方法。

从而保证了两个字符串A和B，即使其在内存中的地址不一样，只要其字符串内容一样，则A.equals(B)返回true，且A.hashCode() = B.hashCode。

```
// String类中的equals方法和hashCode方法，基于jdk1.8版本
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}

```

clone方法

**定义**

创建并返回对象的一份拷贝

**官方注释**

> Creates and returns a copy of this object.

**注意**

- 由于Object本身没有实现Cloneable接口，所以子类不覆盖clone方法并且进行调用的话会发生CloneNotSupportedException异常。
- 关于浅拷贝与深拷贝见[浅拷贝与深拷贝](https://zhuanlan.zhihu.com/p/35641963)

toString方法

**定义**

返回对象的字符串描述

**官方注释**

> Returns a string representation of the object.

**注意**

- toString方法默认输出对象的类的名字+@+哈希码的16进制表示
- toString方法的结果应该是一个简明但易于读懂的字符串。建议Object所有的子类都重写这个方法。

notify、notifyAll、wait方法

见[wait、notify](https://zhuanlan.zhihu.com/p/40699648)

finalize方法

**定义**

当垃圾收集器认为对象没有与GC Roots相连接的引用链时触发的操作，相当于对象死前的最后一波挣扎。

**官方注释**

> Called by the garbage collector on an object when garbage collection determines that there are no more references to the object. A subclass overrides the finalize method to dispose of system resources or to perform other cleanup.

**注意**

- 任何一个对象的finalize方法都只会被系统自动调用一次。
- 不建议使用这个方法来拯救对象，因为其运行代价高昂，不确定性大，无法保证各个对象的调用顺序。

### **equals()**

**1. equals() 与 == 的区别**

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个实例是否引用同一个对象，而 equals() 判断引用的对象是否等价。

```
Integer x = new Integer(1);
Integer y = new Integer(1);
System.out.println(x.equals(y)); // true
System.out.println(x == y);      // false
```

**2. 等价关系**

（一）自反性

```
x.equals(x); // true
```

（二）对称性

```
x.equals(y) == y.equals(x) // true
```

（三）传递性

```
if(x.equals(y) && y.equals(z)) {
    x.equals(z); // true;
}
```

（四）一致性

多次调用 equals() 方法结果不变

```
x.equals(y) == x.equals(y); // true
```

（五）与 null 的比较

对任何不是 null 的对象 x 调用 x.equals(null) 结果都为 false

```
x.euqals(null); // false;
```

**实现**

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 实例进行转型；
- 判断每个关键域是否相等。

```
public class EqualExample {
    private int x;
    private int y;
    private int z;

    public EqualExample(int x, int y, int z) {
        this.x = x;
        this.y = y;
        this.z = z;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    }
}
```

### **hashCode()**

hasCode() 返回散列值，而 equals() 是用来判断两个实例是否等价。等价的两个实例散列值一定要相同，但是散列值相同的两个实例不一定等价。

在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证相等的两个实例散列值也等价。

下面的代码中，新建了两个等价的实例，并将它们添加到 HashSet 中。我们希望将这两个实例当成一样的，只在集合中添加一个实例，但是因为 EqualExample 没有实现 hasCode() 方法，因此这两个实例的散列值是不同的，最终导致集合添加了两个等价的实例。

```
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size()); // 2
```

理想的散列函数应当具有均匀性，即不相等的实例应当均匀分布到所有可能的散列值上。这就要求了散列函数要把所有域的值都考虑进来，可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位。

一个数与 31 相乘可以转换成移位和减法：31*x == (x<<5)-x。

```
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + x;
    result = 31 * result + y;
    result = 31 * result + z;
    return result;
}
```

### **toString()**

默认返回 ToStringExample@4554617c 这种形式，其中 @ 后面的数值为散列码的无符号十六进制表示。

```
public class ToStringExample {
    private int number;

    public ToStringExample(int number) {
        this.number = number;
    }
}
ToStringExample example = new ToStringExample(123);
System.out.println(example.toString());
ToStringExample@4554617c
```

### **clone()**

**1. cloneable**

clone() 是 Object 的受保护方法，这意味着，如果一个类不显式去覆盖 clone() 就没有这个方法。

```
public class CloneExample {
    private int a;
    private int b;
}
```

```
CloneExample e1 = new CloneExample();
// CloneExample e2 = e1.clone(); // 'clone()' has protected access in 'java.lang.Object'
```

接下来覆盖 Object 的 clone() 得到以下实现：

```
public class CloneExample {
    private int a;
    private int b;

    @Override
    protected CloneExample clone() throws CloneNotSupportedException {
        return (CloneExample)super.clone();
    }
}
```

```
CloneExample e1 = new CloneExample();
try {
    CloneExample e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
```

```
java.lang.CloneNotSupportedException: CloneTest
```

以上抛出了 CloneNotSupportedException，这是因为 CloneTest 没有实现 Cloneable 接口。

```
public class CloneExample implements Cloneable {
    private int a;
    private int b;

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

应该注意的是，clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

**2. 深拷贝与浅拷贝**

- 浅拷贝：拷贝实例和原始实例的引用类型引用同一个对象；
- 深拷贝：拷贝实例和原始实例的引用类型引用不同对象。

```
public class ShallowCloneExample implements Cloneable {
    private int[] arr;

    public ShallowCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected ShallowCloneExample clone() throws CloneNotSupportedException {
        return (ShallowCloneExample) super.clone();
    }
}
```

```
ShallowCloneExample e1 = new ShallowCloneExample();
ShallowCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 222
```

```
public class DeepCloneExample implements Cloneable {
    private int[] arr;

    public DeepCloneExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }

    @Override
    protected DeepCloneExample clone() throws CloneNotSupportedException {
        DeepCloneExample result = (DeepCloneExample) super.clone();
        result.arr = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            result.arr[i] = arr[i];
        }
        return result;
    }
}
```

```
DeepCloneExample e1 = new DeepCloneExample();
DeepCloneExample e2 = null;
try {
    e2 = e1.clone();
} catch (CloneNotSupportedException e) {
    e.printStackTrace();
}
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

使用 clone() 方法来拷贝一个对象即复杂又有风险，它会抛出异常，并且还需要类型转换。Effective Java 书上讲到，最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

```
public class CloneConstructorExample {
    private int[] arr;

    public CloneConstructorExample() {
        arr = new int[10];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = i;
        }
    }

    public CloneConstructorExample(CloneConstructorExample original) {
        arr = new int[original.arr.length];
        for (int i = 0; i < original.arr.length; i++) {
            arr[i] = original.arr[i];
        }
    }

    public void set(int index, int value) {
        arr[index] = value;
    }

    public int get(int index) {
        return arr[index];
    }
}
CloneConstructorExample e1 = new CloneConstructorExample();
CloneConstructorExample e2 = new CloneConstructorExample(e1);
e1.set(2, 222);
System.out.println(e2.get(2)); // 2
```

equals()方法必须满足下列5个条件：

1、自反性：对任意 x，x.equals(x)一定返回true。

2、对称性：对任意x 和y，如果y.equals(x)返回true，则x.equals(y)也返回true。

3、传递性：对任意x，y，z，如果有x.equals(y)返回ture，y.equals(z)返回true，则x.equals(z)一定返回true。

4、一致性：对任意 x 和y，如果对象中用于等价比较的信息没有改变，那么无论调用x.equals(y)多少次，返回的结果应该保持一致，要么一直是true，要么一直是false。

5、对任何不是 null 的x，x.equals(null)一定返回false。再说一次，默认的 Object.equals()只是比较对象的地址



hashCode()基本指导：

1、给int变量result赋予某个非零值常量

2、为对象内每个有意义的域f计算出一个int散列码c

3、合并计算得到散列码：

result  = 37 * result + c

4、返回result

5、检查hashCode()最后生成的结果，确保相同的对象有相同的散列码



HashMap的性能因子：

容量：表中桶位数

初始容量：表在创建时所拥有的桶位数

尺寸：表中当前存储的项数

负载因子：尺寸/容量。负载轻的表产生冲突的可能性小，对于插入和查找都是理想的（会减慢使用迭代器进行遍历的过程）。当负载情况达到负载因子的水平时，容器将会自动增加其容量，实现方式是使容量大致加倍，并重新将现有对象分布到新到桶位集中（再散列），使用的默认负载因子是0.75。



## 第十八章 Java I/O系统

## **概览**

Java 的 I/O 大概可以分成以下几类：

1. 磁盘操作：File
2. 字节操作：InputStream 和 OutputStream
3. 字符操作：Reader 和 Writer
4. 对象操作：Serializable
5. 网络操作：Socket
6. 新的输入/输出：NIO

## **磁盘操作**

File 类可以用于表示文件和目录，但是它只用于表示文件的信息，而不表示文件的内容。



## **字节操作**

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/DP-Decorator-java.io.png)

Java I/O 使用了装饰者模式来实现。以 InputStream 为例，InputStream 是抽象组件，FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作。FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能，例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

```
BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
```

DataInputStream 装饰者提供了对更多数据类型进行输入的操作，比如 int、double 等基本类型。

批量读入文件内容到字节数组：

```
byte[] buf = new byte[20*1024];
int bytes = 0;
// 最多读取 buf.length 个字节，返回的是实际读取的个数，返回 -1 的时候表示读到 eof，即文件尾
while((bytes = in.read(buf, 0 , buf.length)) != -1) {
    // ...
}
```



## **字符操作**

不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符，所以 I/O 操作的都是字节而不是字符。但是在程序中操作的通常是字符形式的数据，因此需要提供对字符进行操作的方法。

InputStreamReader 实现从文本文件的字节流解码成字符流；OutputStreamWriter 实现字符流编码成为文本文件的字节流。它们继承自 Reader 和 Writer。

编码就是把字符转换为字节，而解码是把字节重新组合成字符。

```
byte[] bytes = str.getBytes(encoding);     // 编码
String str = new String(bytes, encoding)； // 解码
```

如果编码和解码过程使用不同的编码方式那么就出现了乱码。

- GBK 编码中，中文占 2 个字节，英文占 1 个字节；
- UTF-8 编码中，中文占 3 个字节，英文占 1 个字节；
- UTF-16be 编码中，中文和英文都占 2 个字节。

UTF-16be 中的 be 指的是 Big Endian，也就是大端。相应地也有 UTF-16le，le 指的是 Little Endian，也就是小端。

Java 使用双字节编码 UTF-16be，这不是指 Java 只支持这一种编码方式，而是说 char 这种类型使用 UTF-16be 进行编码。char 类型占 16 位，也就是两个字节，Java 使用这种双字节编码正是为了让一个中文或者一个英文都能使用一个 char 来存储。



## **对象操作**

序列化就是将一个对象转换成字节序列，方便存储和传输。

序列化：ObjectOutputStream.writeObject()

反序列化：ObjectInputStream.readObject()

序列化的类需要实现 Serializable 接口，它只是一个标准，没有任何方法需要实现。

transient 关键字可以使一些属性不会被序列化。

ArrayList 序列化和反序列化的实现 ：ArrayList 中存储数据的数组是用 transient 修饰的，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化。通过重写序列化和反序列化方法，使得可以只序列化数组中有内容的那部分数据。

```
private transient Object[] elementData;
```



## **网络操作**

Java 中的网络支持：

1. InetAddress：用于表示网络上的硬件资源，即 IP 地址；
2. URL：统一资源定位符，通过 URL 可以直接读取或者写入网络上的数据；
3. Sockets：使用 TCP 协议实现网络通信；
4. Datagram：使用 UDP 协议实现网络通信。

### **InetAddress**

没有公有构造函数，只能通过静态方法来创建实例。

```
InetAddress.getByName(String host);
InetAddress.getByAddress(byte[] addr);
```

### **URL**

可以直接从 URL 中读取字节流数据

```
URL url = new URL("http://www.baidu.com");
InputStream is = url.openStream();                           // 字节流
InputStreamReader isr = new InputStreamReader(is, "utf-8");  // 字符流
BufferedReader br = new BufferedReader(isr);
String line = br.readLine();
while (line != null) {
    System.out.println(line);
    line = br.readLine();
}
br.close();
isr.close();
is.close();
```

### **Sockets**

- ServerSocket：服务器端类
- Socket：客户端类
- 服务器和客户端通过 InputStream 和 OutputStream 进行输入输出。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/ClienteServidorSockets1521731145260.jpg)

### **Datagram**

- DatagramPacket：数据包类
- DatagramSocket：通信类

## **NIO**

- [Java NIO Tutorial](http://tutorials.jenkov.com/java-nio/index.html)
- [Java NIO 浅析](https://tech.meituan.com/nio.html)
- [IBM: NIO 入门](https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html)

新的输入/输出 (NIO) 库是在 JDK 1.4 中引入的。NIO 弥补了原来的 I/O 的不足，提供了高速的、面向块的 I/O。

### **流与块**

I/O 与 NIO 最重要的区别是数据打包和传输的方式，I/O 以流的方式处理数据，而 NIO 以块的方式处理数据。

面向流的 I/O 一次处理一个字节数据，一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。不利的一面是，面向流的 I/O 通常相当慢。

面向块的 I/O 一次处理一个数据块，按块处理数据比按流处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

I/O 包和 NIO 已经很好地集成了，java.io.* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。



## **通道与缓冲区**

#### **通道**

通道 Channel 是对原 I/O 包中的流的模拟，可以通过它读取和写入数据。

通道与流的不同之处在于，流只能在一个方向上移动，(一个流必须是 InputStream 或者 OutputStream 的子类)，而通道是双向的，可以用于读、写或者同时用于读写。

通道包括以下类型：

- FileChannel：从文件中读写数据；
- DatagramChannel：通过 UDP 读写网络中数据；
- SocketChannel：通过 TCP 读写网络中数据；
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

#### **缓冲区**

发送给一个通道的所有数据都必须首先放到缓冲区中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下类型：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

### **缓冲区状态变量**

- capacity：最大容量；
- position：当前已经读写的字节数；
- limit：还可以读写的字节数。

状态变量的改变过程举例：

① 新建一个大小为 8 个字节的缓冲区，此时 position 为 0，而 limit = capacity = 8。capacity 变量不会改变，下面的讨论会忽略它。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/1bea398f-17a7-4f67-a90b-9e2d243eaa9a.png)

② 从输入通道中读取 5 个字节数据写入缓冲区中，此时 position 移动设置为 5，limit 保持不变。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/80804f52-8815-4096-b506-48eef3eed5c6.png)

③ 在将缓冲区的数据写到输出通道之前，需要先调用 flip() 方法，这个方法将 limit 设置为当前 position，并将 position 设置为 0。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/952e06bd-5a65-4cab-82e4-dd1536462f38.png)

④ 从缓冲区中取 4 个字节到输出缓冲中，此时 position 设为 4。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/b5bdcbe2-b958-4aef-9151-6ad963cb28b4.png)

⑤ 最后需要调用 clear() 方法来清空缓冲区，此时 position 和 limit 都被设置为最初位置。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/67bf5487-c45d-49b6-b9c0-a058d8c68902.png)



### **文件 NIO 实例**

以下展示了使用 NIO 快速复制文件的实例：

```
public class FastCopyFile {
    public static void main(String args[]) throws Exception {

        String inFile = "data/abc.txt";
        String outFile = "data/abc-copy.txt";

        // 获得源文件的输入字节流
        FileInputStream fin = new FileInputStream(inFile);
        // 获取输入字节流的文件通道
        FileChannel fcin = fin.getChannel();

        // 获取目标文件的输出字节流
        FileOutputStream fout = new FileOutputStream(outFile);
        // 获取输出字节流的通道
        FileChannel fcout = fout.getChannel();

        // 为缓冲区分配 1024 个字节
        ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

        while (true) {
            // 从输入通道中读取数据到缓冲区中
            int r = fcin.read(buffer);
            // read() 返回 -1 表示 EOF
            if (r == -1)
                break;
            // 切换读写
            buffer.flip();
            // 把缓冲区的内容写入输出文件中
            fcout.write(buffer);
            // 清空缓冲区
            buffer.clear();
        }
    }
}
```

### **选择器**

一个线程 Thread 使用一个选择器 Selector 通过轮询的方式去检查多个通道 Channel 上的事件，从而让一个线程就可以处理多个事件。

因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件具有更好的性能。

![img](https://github.com/CyC2018/Interview-Notebook/raw/master/pics/4d930e22-f493-49ae-8dff-ea21cd6895dc.png)



#### **创建选择器**

```
Selector selector = Selector.open();
```

#### **将通道注册到选择器上**

```
ServerSocketChannel ssChannel = ServerSocketChannel.open();
ssChannel.configureBlocking(false);
ssChannel.register(selector, SelectionKey.OP_ACCEPT);
```

通道必须配置为非阻塞模式，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它时间，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。

在将通道注册到选择器上时，还需要指定要注册的具体事件，主要有以下几类：

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE

它们在 SelectionKey 的定义如下：

```
public static final int OP_READ = 1 << 0;
public static final int OP_WRITE = 1 << 2;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_ACCEPT = 1 << 4;
```

可以看出每个事件可以被当成一个位域，从而组成事件集整数。例如：

```
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

#### **监听事件**

```
int num = selector.select();
```

使用 select() 来监听事件到达，它会一直阻塞直到有至少一个事件到达。

#### **获取到达的事件**

```
Set<SelectionKey> keys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = keys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    if (key.isAcceptable()) {
        // ...
    } else if (key.isReadable()) {
        // ...
    }
    keyIterator.remove();
}
```

#### **事件循环**

因为一次 select() 调用不能处理完所有的事件，并且服务器端有可能需要一直监听事件，因此服务器端处理事件的代码一般会放在一个死循环内。

```
while (true) {
    int num = selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = keys.iterator();
    while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if (key.isAcceptable()) {
            // ...
        } else if (key.isReadable()) {
            // ...
        }
        keyIterator.remove();
    }
}
```



### **套接字 NIO 实例**

```
public class NIOServer {

    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();

        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        ssChannel.configureBlocking(false);
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);

        ServerSocket serverSocket = ssChannel.socket();
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
        serverSocket.bind(address);

        while (true) {
            selector.select();
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = keys.iterator();
            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();
                if (key.isAcceptable()) {
                    ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();
                    // 服务器会为每个新连接创建一个 SocketChannel
                    SocketChannel sChannel = ssChannel1.accept();
                    sChannel.configureBlocking(false);
                    // 这个新连接主要用于从客户端读取数据
                    sChannel.register(selector, SelectionKey.OP_READ);
                } else if (key.isReadable()) {
                    SocketChannel sChannel = (SocketChannel) key.channel();
                    System.out.println(readDataFromSocketChannel(sChannel));
                    sChannel.close();
                }
                keyIterator.remove();
            }
        }
    }

    private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        StringBuffer data = new StringBuffer();
        while (true) {
            buffer.clear();
            int n = sChannel.read(buffer);
            if (n == -1)
                break;
            buffer.flip();
            int limit = buffer.limit();
            char[] dst = new char[limit];
            for (int i = 0; i < limit; i++)
                dst[i] = (char) buffer.get(i);
            data.append(dst);
            buffer.clear();
        }
        return data.toString();
    }
}
```

```
public class NIOClient {

    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8888);
        OutputStream out = socket.getOutputStream();
        String s = "hello world";
        out.write(s.getBytes());
        out.close();
    }
}
```



### **内存映射文件**

内存映射文件 I/O 是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的 I/O 快得多。

只有文件中实际读取或者写入的部分才会映射到内存中。

现代操作系统一般会根据需要将文件的部分映射为内存的部分，从而实现文件系统。Java 内存映射机制只不过是在底层操作系统中可以采用这种机制时，提供了对该机制的访问。

向内存映射文件写入可能是危险的，仅只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

下面代码行将文件的前 1024 个字节映射到内存中，map() 方法返回一个 MappedByteBuffer，它是 ByteBuffer 的子类。因此，您可以像使用其他任何 ByteBuffer 一样使用新映射的缓冲区，操作系统会在需要时负责执行映射。

```
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
```



### **对比**

NIO 与普通 I/O 的区别主要有以下两点：

- NIO 是非阻塞的。应当注意，FileChannel 不能切换到非阻塞模式，套接字 Channel 可以。
- NIO 面向块，I/O 面向流。



实现Externalzable接口：代替实现Serializable，在不希望对象的某一部分被序列化，或者一个对象被还原以后，某子对象需要重新创建，从而不必将该子对象序列化。

InputStream和OutputStream类在以面向字节形式的I/O中仍然可以提供有价值的功能；Reader和Writer则提供兼容Unicode与面向字符的I/O功能。

 

InputStream子类：

ByteArrayInputStream

StringBufferInputStream

FileInputStream

PipedInputStream

SequenceInputStream

FilterInputStream



OutputStream子类

ByteArrayOutputStream

FileOutputStream

PipedOutputStream

FilterOutputStream

 

“big endian”（高位优先）将最重要的字节存放在地址最低的存储器单元。而“little endian”（低位优先）则是将最重要的字节放在地址最高的存储器单元。

 对象序列化：对象能够在程序不运行的情况下仍能存在并保存其信息，Java对象序列化将那些实现了Serializable接口的对象转换成一个字节序列，并能够在以后将这个字节序列完全恢复为原来的对象。

 

## 第十九章 枚举类型

Enum

EnumSet

EnumMap



## 第二十章 注解

Java内置注解：

@Override：当前的方法定义将覆盖超类的方法

@Deprecated：如果程序员使用了注解它的元素，编译器将会发出警告

@SuppressWarnings：关闭不当的编译器警告信息



XML描述文件：提供一些基本对象/关系映射功能，能够自动生成数据库表，用以储存JavaBean对象。

Junit单元测试

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

## 第二十一章 并发

使用Executor管理Thread对象

Callable接口是一种具有参数类型的泛型，它的参数类型表示的是从call()中返回的值，并且必须使用ExcutorService.submit()方法调用它。

优先级：唯一可移植的方法是当调整优先级的时候，只使用MAX_PRIORITY，NORM_PRIORITY，MIN_PRIORITY





























































