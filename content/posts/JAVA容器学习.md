+++
author = "pikachu"
title = "Java容器学习"
date = "2019-07-29"
description = "Lorem Ipsum Dolor Si Amet"
tags = [
	"java"
]
categories = [
    "IT"
]
+++


## 一、容器间的区别

#### 1.1、List、Map、Set三者间的区别

- **List**：List接口存储一组可重复的、有序的对象
- **Map**：Map接口存储以key为索引，value为值数据，其中key不可以重复，value可以
- **Set**：Set接口存储一组唯一的数据，不允许多个元素引用相同的对象


#### 1.2、ArrayList和LinkedList的区别

- **是否线程安全**：都是非线程安全的

- **底层数据结构**：
    - ArrayList：底层是由`Object`数组实现
    - LinkedList：是用双向链表实现（JDK1.7前为循环双向链表

- **插入及删除效率是否受位置影响**：
    - ArrayList：由于数组的特性，插入及删除效率受位置的影响，需要对相关数据进行移动。
    - LinkedList：由于链表的特性，其插入及删除效率不受位置影响，时间复杂度都为O(1)

- **是否支持快速随机访问**：
    - ArrayList：支持高速随机访问，实现了`RandomAccess`标识接口（实现则采用`indexBinarySearch`方法，否则采用`iteratorBinarySearch`方法）
    - LinkedList：不支持高速随机访问，需要遍历数据

- **内存空间占用**：
    - ArrayList：在List结尾会预留一定的容量空间
    - LinkedList：则主要体现在每个元素都需要空间存放直接前驱和直接后继。

- **扩容机制**：
    - ArrayList默认长度为10，扩容时为原来的`1.5倍`


#### 1.3、ArrayList和Vector的区别

- Vector：所有方法都是**线程同步**的，所以在同步操作上需要耗费大量时间
- ArrayList：是非线程安全的，在单线程的情况下采用ArrayList会有更好的性能


#### 1.4、HashMap和HashTable（已被淘汰）的区别

- **是否线程安全**：HashTable为线程安全

- **效率**：HashTable由于是全表同步的，其同步开销较大，效率较低

- **对Null key和value的支持**：
    - HashMap：支持key和value为null，但是key中的null必须是唯一的
    - HashTable：支持value为null，当key为null时会报`NullPointException`

- **初始容量及扩容**：
    - HashTable：默认大小为11，扩容为原来的`2n+1`，当给定了初始值时，会直接使用该值
    - HashMap：默认大小为16，扩容时为原来的`2倍`，当给定了初始值时，会先扩充为`2的幂次方`

```
HashMap的构造方法：

 public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
     public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

- **底层数据结构**：
    - JDK1.8后，HashMap在处理哈希冲突上采用的是`数组 + 链表`的形式，同时在链表长度超过阈值时（默认为8），将链表转换为`红黑树`，提高了搜索效率，将链表（寻址时间复杂度为`O(N)`）转换为红黑树（寻址时间复杂度为`O(log(N))`），而HashTable则没有


#### 1.5、HashMap和HashSet的区别

- HashSet：
    - 底层是基于HashMap实现 
    - 在校验重复上，插入时先比较HashCode是否相同，如果相同则调用`equal`方法比较，都相同则不能成功插入。

#### 1.6、HashTable和ConCurrentHashMap的区别

- **底层数据结构**：
    - **HashTable**：数组（主体） + 链表（解决哈希冲突）
    - **ConCurrentHashMap**：
        - *JDK1.7*：分段数组 + 链表
        - *JDK1.8*：数组 + 链表/红黑二叉树

- **实现线程安全的方式**：
    - **HashTable**：
        - 同一把锁，使用`synchronized`保证线程安全，效率非常低下。
        ![image](https://user-images.githubusercontent.com/38284818/62021127-548c5700-b1f8-11e9-8902-927c850b7774.png)
    - **ConCurrentHashMap**：
        - *JDK1.7*：采用`分段锁`，对整个桶数据进行了分割分段，每段有一把锁，多线程并发不同数据段的数据，不会有锁竞争，提高了并发效率。
        ![image](https://user-images.githubusercontent.com/38284818/62021137-5f46ec00-b1f8-11e9-8635-958ee4cb7d3a.png)
        - *JDK1.8*：取消了分段锁机制，采用`Node 数组（实现了Map接口） + 链表 + 红黑树`，并发控制使用`synchronized `和`CAS`。synchronized只锁定当前链表或红黑二叉树的`首节点`，这样只要hash不冲突，就不会产生并发，效率又提升N倍。
        ![image](https://user-images.githubusercontent.com/38284818/62021143-640ba000-b1f8-11e9-98f7-5a2e79dc5d63.png)
        
&nbsp;
&nbsp;

## 二、容器的特性

#### 2.1、HashMap的长度为什么是2的幂次方

- 为了让HashMap存取高效，尽量减少碰撞，也就是尽量的把数据分配均匀。Hash 值的范围值`-2147483648`到`2147483647`，前后加起来大概`40亿`的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，`内存是放不下的`。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度`取模运算`，得到的`余数`才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是`(length - 1) & hash`。（n代表数组长度）。这也就解释了 HashMap 的长度为什么是2的幂次方。

- &的运算比%的运算效率是高很多的，为了提高运算，在保证长度为2的幂次方的条件下， `hash & (length - 1)` = `hash % length`是成立的，此时使用位运算`&`可以提高运行效率。


#### 2.2、Comparable和Comparator的区别

- comparable接口出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序，一般是在实体中实现此接口并重写compareTo方法。

```
//person对象没有实现Comparable接口，所以必须实现，这样才不会出错，才可以使TreeMap中的数据按顺序排列，而String和Integer等类型都已经默认实现了Comparable接口

public  class Person implements Comparable<Person> {
    private int age;
    
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    /**
     * TODO重写compareTo方法实现按年龄来排序
     */
    @Override
    public int compareTo(Person o) {
        // TODO Auto-generated method stub
        if (this.age > o.getAge()) {
            return 1;
        } else if (this.age < o.getAge()) {
            return -1;
        }
        return age;
    }
    
     public static void main(String[] args) {
        //TreeMap是
        TreeMap<Person, String> pdata = new TreeMap<Person, String>();
        pdata.put(new Person("张三", 30), "zhangsan");
        pdata.put(new Person("李四", 20), "lisi");
        pdata.put(new Person("王五", 10), "wangwu");
        pdata.put(new Person("小红", 5), "xiaohong");
        
        // 得到key的值的同时得到key所对应的值
        Set<Person> keys = pdata.keySet();
        for (Person key : keys) {
            System.out.println(key.getAge() + "-" + key.getName());

        }
    }
}
```

- comparator接口出自`java.util`包，它有一个`compare(Object obj1, Object obj2)`方法用来排序，通常是与`Collections`类一起使用

```
ArrayList<Integer> arrayList = new ArrayList<Integer>();

//反转
Collections.reverse(arrayList);
//按自然排序升序排序
Collections.sort(arrayList);
// 定制排序的用法
Collections.sort(arrayList, new Comparator<Integer>() {

    @Override
    public int compare(Integer o1, Integer o2) {
        return o2.compareTo(o1);
    }
});
```


#### 2.3、哈希冲突的解决方法

- **开放地址法**：从发生冲突的那个单元起，按照一定的次序，从哈希表中选择一个空闲的单元，将发生冲突的单元放入即可，开放地址法的缺点是不能真正的删除元素，只能做特殊标识，直到有下个元素插入才可以真正删除，否则会引起查找错误。
    - *线行探查法*：依次判断
    - *平方探查法*：d[i] + n^2
    - 双散列函数探查法：
- **拉链法**：`数组 + 链表`
- **再哈希法**：构造`多个不同`的哈希函数，出现ch
- **建立公共溢出区**：将哈希表分为`公共表`和`溢出表`，当溢出发生时，将所有溢出数据统一放到溢出区。

&nbsp;

## 三、容器底层数据结构总结

#### 3.1、List：

- **ArrayList**：Object数组
- **Vector**：Object数组
- **LinkedList**：双向链表（1.6前为循环双向链表）


#### 3.2、Map

- **HashMap**：
    - 1.7以前：数组 + 链表
    - 1.8以后：数组 + 链表/红黑二叉树
- **HashTable**：
    - 数组 + 链表
- **ConcurrentHashMap**：
    - 1.7以前：数组 + 链表（分段锁）
    - 1.8以后：Node数组 + 链表/红黑二叉树（只锁定当前链表或红黑二叉树的首节点）
- **TreeMap**：红黑树（自平衡的排序二叉树）
- **LinkedHashMap**：`继承`自 HashMap，在 HashMap 基础上，通过维护一条`双向链表`，解决了 HashMap 不能随时`保持遍历顺序和插入顺序一致`的问题（参考：https://www.imooc.com/article/22931）


#### 3.3、Set

- **HashSet（无序、唯一）**：基于 HashMap 实现的，底层采用 HashMap 来保存元素
- **LinkedHashSet**： LinkedHashSet 继承于 HashSet，并且其内部是通过 LinkedHashMap 来实现的。
- **TreeSet（有序，唯一）**： 红黑树(自平衡的排序二叉树，需要重写`Compare`或者`CompareTo`方法)
- **参考**：https://blog.csdn.net/lijock/article/details/80410202


#### 3.4、集合的选用

- 主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用Map接口下的集合，需要排序时选择TreeMap,不需要排序时就选择HashMap,需要保证线程安全就选用ConcurrentHashMap.当我们只需要存放元素值时，就选择实现Collection接口的集合，需要保证元素唯一时选择实现Set接口的集合比如TreeSet或HashSet，不需要就选择实现List接口的比如ArrayList或LinkedList，然后再根据实现这些接口的集合的特点来选用。

&nbsp;

## 四、参考

- https://www.jianshu.com/p/4d3cb99d7580
- https://snailclimb.top/JavaGuide/#/java/collection/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98?id=collection