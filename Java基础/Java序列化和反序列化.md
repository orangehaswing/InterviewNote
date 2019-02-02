# Java序列化和反序列化

## 什么是序列化？为什么要序列化？

Java 序列化就是指将对象转换为字节序列的过程，而反序列化则是只将字节序列转换成目标对象的过程。

我们都知道，在进行浏览器访问的时候，我们看到的文本、图片、音频、视频等都是通过二进制序列进行传输的，那么如果我们需要将Java对象进行传输的时候，是不是也应该先将对象进行序列化？答案是肯定的，我们需要先将Java对象进行序列化，然后通过网络，IO进行传输，当到达目的地之后，再进行反序列化获取到我们想要的对象，最后完成通信。

## 如何实现序列化

2.1、使用到JDK中关键类 ObjectOutputStream 和ObjectInputStream

ObjectOutputStream 类中：通过使用writeObject(Object object) 方法，将对象以二进制格式进行写入。

ObjectInputStream 类中：通过使用readObject（）方法，从输入流中读取二进制流，转换成对象。

2.2、目标对象需要先实现 Seriable接口

我们创建一个Student类：

public class Student implements Serializable {
   private static final long serialVersionUID = 3404072173323892464L;    private String name;    private transient String id;    private String age;    @Override
   public String toString() {        return "Student{" +                "name='" + name + '\'' +                ", id='" + id + '\'' +                ", age='" + age + '\'' +                '}';
   }    public String getAge() {        return age;
   }    public void setAge(String age) {        this.age = age;
   }    public Student(String name, String id) {
​       System.out.println("args Constructor");        this.name = name;        this.id = id;
   }    public Student() {
​       System.out.println("none-arg Constructor");
   }    public String getName() {        return name;
   }    public void setName(String name) {        this.name = name;
   }    public String getId() {        return id;
   }    public void setId(String id) {        this.id = id;
   }
}

代码中Student类实现了Serializable 接口，并且生成了一个版本号：

private static final long serialVersionUID = 3404072173323892464L;

首先：

1、Serializable 接口的作用只是用来标识我们这个类是需要进行序列化，并且Serializable 接口中并没有提供任何方法。

2、serialVersionUid 序列化版本号的作用是用来区分我们所编写的类的版本，用于判断反序列化时类的版本是否一直，如果不一致会出现版本不一致异常。

3、transient 关键字，主要用来忽略我们不希望进行序列化的变量

2.3、将对象进行序列或和反序列化

如果你想学习Java可以来这个群，首先是一二六，中间是五三四，最后是五一九，里面有大量的学习资料可以下载。

2.3.1 第一种写入方式： 

public static  void main(String[] args){
​       File file = new File("D:/test.txt");
​       Student student = new Student("孙悟空","12");        try {
​           ObjectOutputStream outputStream = new ObjectOutputStream(new FileOutputStream(file));
​           outputStream.writeObject(student);
​           outputStream.close();
​       } catch (IOException e) {
​           e.printStackTrace();
​       }        try {
​           ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream(file));
​           Student s = (Student) objectInputStream.readObject();
​           System.out.println(s.toString());
​           System.out.println(s.equals(student));
​       } catch (IOException e) {
​           e.printStackTrace();
​       } catch (ClassNotFoundException e) {
​           e.printStackTrace();
​       }
   }

创建对象Student ，然后通过ObjectOutputStream类中的writeObject()方法，将对象输出到文件中。

然后通过ObjectinputStream 类中的readObject（）方法反序列化，获取对象。

2.3.2 第二种写入方式：

在Student 类中实现writeObject（）和readObject（）方法：

private void writeObject(ObjectOutputStream objectOutputStream) throws IOException {
​       objectOutputStream.defaultWriteObject();
​       objectOutputStream.writeUTF(id);
   }    private void readObject(ObjectInputStream objectInputStream) throws IOException, ClassNotFoundException {
​       objectInputStream.defaultReadObject();
​       id = objectInputStream.readUTF();
   }

通过这中方式进行序列话，我们可以自定义想要进行序列化的变量，将输入流和输出流传入对线实例中，然后进行序列化以及反序列化。

2.3.3 第三种写入方式：

Student 实现 Externalnalizable接口 而不实现Serializable 接口

Externaliable 接口是 Serializable 的子类，有着和Serializable接口同样的功能：

public class Student implements Externalizable {
   private static final long serialVersionUID = 3404072173323892464L;    private String name;    private transient String id;    private String age;    @Override
   public String toString() {        return "Student{" +                "name='" + name + '\'' +                ", id='" + id + '\'' +                ", age='" + age + '\'' +                '}';
   }    public String getAge() {        return age;
   }    public void setAge(String age) {        this.age = age;
   }    public Student(String name, String id) {
​       System.out.println("args Constructor");        this.name = name;        this.id = id;
   }    public Student() {
​       System.out.println("none-arg Constructor");
   }    public String getName() {        return name;
   }    public void setName(String name) {        this.name = name;
   }    public String getId() {        return id;
   }    public void setId(String id) {        this.id = id;
   }    @Override
   public void writeExternal(ObjectOutput out) throws IOException {
​       out.writeObject(name);
​       out.writeObject(id);
   }    @Override
   public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
​       name = (String) in.readObject();
​       id = (String) in .readObject();
   }
}

通过和前面的第二种写入方法对比，我们可以发现他们的实现原理都是十分的类似，不过实现Externalnalizable接口 并不支持第一种序列化方法，它只能够通过实现接口中的writeExternal()和readExternal（）方法实现对象的序列化。

## 面试中关于序列化的问题

1、什么是序列化，如何实现序列化

java中对象的序列化就是将对象转换成二进制序列，反序列化则是将二进制序列转换成对象

Java 实现序列化有多种方式

1、首先需要使用到工具类ObjectInputStream 和ObjectOutputStream 两个IO类

2、实现Serializable 接口：

有两种具体序列化方法：

直接通过ObjectOutputStream 和 ObjectInputStream 类中的 writeObject（）和readObject（）方法

通过在序列化对象中实现writeObject（）和readObject（）方法，传入ObjectOutputStream和ObjectInputStream对象，完成序列化 
3、实现Externalizable 接口： 
只能够通过实现接口中的writeExternal（）和readExternal（）方法实现对象的序列化

2、transient 关键字？如何将transient修饰符修饰的变量序列化？ 
transient 的作用是用来屏蔽我们不希望进行序列化的变量，是对象在进行序列化和反序列话的过程中忽略该变量。我们可以通过上述序列化方法中的 实现writeObject 和readObject 方法，在方法中调用输出流或输入流的writeUTF（）和readUTF（）方法。或者通过实现Externalizable 接口，实现writeExternal（）和readExternal（）方法，然后再自定义序列话对象。

3、如何保证序列化和反序列化后的对象一致？（如有异议望指正） 
对于这个问题我在查阅了一些资料之后，发现并不能保证序列化和反序列化之后的对象是一致的，因为我们在反序列化的过程中，是先创建一个对象，然后再通过对对象进行赋值来完成对象的反序列化，这样问题就来了，在创建了一个新的对象之后，对象引用和原本的对象并不是指向同一个目标。因此我们只能保证他们的数据和版本一致，并不能保证对象一致。













