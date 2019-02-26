# Java面试-序列化和反序列化

1. 什么是序列化，如何实现序列化

   java中对象的序列化就是将对象转换成二进制序列，反序列化则是将二进制序列转换成对象

   Java 实现序列化有多种方式

   - 首先需要使用到工具类ObjectInputStream 和ObjectOutputStream 两个IO类

   - 实现Serializable 接口：

     有两种具体序列化方法：

     直接通过ObjectOutputStream 和 ObjectInputStream 类中的 writeObject() 和readObject() 方法

     通过在序列化对象中实现writeObject() 和readObject() 方法，传入ObjectOutputStream和ObjectInputStream对象，完成序列化 

   - 实现Externalizable 接口： 

     只能够通过实现接口中的writeExternal() 和readExternal() 方法实现对象的序列化

2. transient 关键字？如何将transient修饰符修饰的变量序列化？ 

   transient 的作用是用来屏蔽我们不希望进行序列化的变量，是对象在进行序列化和反序列话的过程中忽略该变量。我们可以通过上述序列化方法中的 实现writeObject 和readObject 方法，在方法中调用输出流或输入流的writeUTF（）和readUTF（）方法。或者通过实现Externalizable 接口，实现writeExternal（）和readExternal（）方法，然后再自定义序列话对象。

3. 如何保证序列化和反序列化后的对象一致？（如有异议望指正） 

   对于这个问题我在查阅了一些资料之后，发现并不能保证序列化和反序列化之后的对象是一致的，因为我们在反序列化的过程中，是先创建一个对象，然后再通过对对象进行赋值来完成对象的反序列化，这样问题就来了，在创建了一个新的对象之后，对象引用和原本的对象并不是指向同一个目标。因此我们只能保证他们的数据和版本一致，并不能保证对象一致。



