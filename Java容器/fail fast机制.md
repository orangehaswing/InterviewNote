# fail fast机制

## fail-fast是什么

fail-fast的字面意思是“快速失败”。当我们在遍历集合元素的时候，经常会使用迭代器，但在迭代器遍历元素的过程中，如果集合的结构被改变的话，就会抛出异常，防止继续遍历。这就是所谓的快速失败机制。

下面我们来看看官方文档在HashMap这个集合中，它是怎么解释fail-fast的(如下图)：

![img](https://pic4.zhimg.com/v2-8d9159b8a5e5e56e50db3d9197b43963_b.jpg)

意思就是说，当Iterator这个迭代器被创建后，除了迭代器本身的方法(remove)可以改变集合的结构外，其他的因素如若**改变了集合的结构**，都被抛出ConcurrentModificationException异常。

请在继续看官方的描述：

![img](https://pic3.zhimg.com/v2-c5a3b393aa4d1f84b68f6f57c791fbaa_b.jpg)

意思就是说：迭代器的快速失败行为是不一定能够得到保证的，一般来说，存在非同步的并发修改时，不可能做出任何坚决的保证的。但是快速失败迭代器会做出最大的努力来抛出ConcurrentModificationException。因此，编写依赖于此异常的程序的做法是不正确的。正确的做法应该是：迭代器的快速失败行为应该仅用于检测程序中的bug.

**稍微总结下：fail-fast,即快速失败机制，它是java集合中的一种错误检测机制，当多个线程（当个线程也是可以滴）,在结构上对集合进行改变时，就有可能会产生fail-fast机制。**

> 这里，我解释下什么是结构上的改变。
> 例如集合上的插入和删除就是结构上的改变，但是，如果是对集合中某个元素进行修改的话，并不是结构上的改变哦。 
> **下面，我们来演示下在单线程的环境下，fail-fast抛出异常的实例：**

```
for(int i = 10; i < 100; i++){
        map.put(i, i);
    }
    List<Integer> list = new ArrayList<>();
    for(int i = 0; i < 20; i++){
        list.add(i);
    }
    Iterator<Integer> it = list.iterator();
    int temp = 0;
    while(it.hasNext()){
        if(temp == 3){
            temp++;
            list.remove(3);
        }else{
            temp++;
            System.out.println(it.next());
        }
    }
}

```

打印结果：

![img](https://pic4.zhimg.com/v2-1610052124811f23b7fc73b15e5030f3_b.jpg)

**结果分析：**因为当temp==3的时候，执行list.remove()方法，集合的结构被改变了，所以再次遍历迭代器的时候，就会抛出异常。   

------

## fail-fast的工作原理

我们首先先来看下源码：

![img](https://pic3.zhimg.com/v2-3446ea214ef699b4a142ae51fd4478be_b.jpg)

**分析：**从源码我们可以发现，迭代器在执行next()等方法的时候，都会调用checkForComodification()这个方法，查看modCount==expectedModCount?如果相等则抛出异常。

expectedModcount:这个值在对象被创建的时候就被赋予了一个固定的值modCount。也就是说这个值是不变的。也就是说，如果在迭代器遍历元素的时候，如果modCount这个值发生了改变，那么再次遍历时就会抛出异常。 
**什么时候modCount会发生改变呢？**

这次就不带大家看源码了。其实当我们对集合的元素的个数做出改变的时候，modCount的值就会被改变，如果删除，插入。但修改则不会。

------

## fail-fast的一些处理方法 

**在多线程环境下，用Iterator遍历时要加锁**

```
Iterator<String> iterator = list.iterator();
        while (iterator.hasNext()){
            synchronized (对象){
                String item = iterator.next();
                if(){
                    iterator.remove();
                }
            }
        }

```

**使用CopyOnWriteArrayList 代替 ArrayList**

CopyOnWriteArrayList内部对iterator加锁，COW是fail-safe的，它实行读写分离，写操作会复制一个新集合，在新集合上操作元素，操作完成后将原来的引用指向新集合。  使用COWList有两个注意点： * 尽量设置合理的初始容量，扩容成本较大。 * 使用批处理操作，addAll和removeAll等。

COWList适用于读多写极少的场景。

COW满足了一致性，但是读取不到最新的数据（CAP 一致性和可用性之间的矛盾）。

## 事故现场

## 事故一

```
public class Test3 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(5);
        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        list.add("5");

        List<String> subList = list.subList(0,3);

        //下面三行不注释，操作subList时会抛出ConcurrentModificationException
        list.remove(0);
        list.add("10");
        list.clear();

        subList.clear();
        subList.add("6");
        subList.add("7");
        subList.remove(0);

        for(String s : list){
            System.out.println(s);
        }

        //打印[7, 4, 5]
        System.out.println(list);
    }
}

```

这里面有几个注意点： * subList()出来的子列表无法序列化 subList返回的是`ArrayList`的内部类`SubList`,它没有实现Searlizable接口。

`private class SubList extends AbstractList implements RandomAccess`

-  subList的修改会同时修改list。
-  list（主列表）的修改会让子列表操作抛出异常。

## 事故二

用foreach遍历集合演示fail-fast机制。

```
public class Test2 {

    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        list.add("5");

        for (String s : list) {
            System.out.println(s);
            //将“4”换成其他元素则会异常
            if(s.equals("4")){
                list.remove(s);
            }
            System.out.println("after if");
        }
        //输出[1, 2, 3, 5]
        System.out.println(list);
    }
}

```

依然是一个list里面有五个数据，然后foreach循环，当等于"4"的时候，将它从列表的移除。编译运行一切正常，最后输出结果。但是将判断条件换成“4”以外的元素的时候，就会触发fail-fast，抛出ConcurrentModificationException。

## fail-safe机制

fail-safe任何对集合结构的修改都会在一个复制的集合上进行修改，因此不会抛出ConcurrentModificationException

fail-safe机制有两个问题

1. 需要复制集合，产生大量的无效对象，开销大
2. 无法保证读取的数据是目前原始数据结构中的数据。