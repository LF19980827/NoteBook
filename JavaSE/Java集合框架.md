# 1、Java集合框架概述

![](G:\个人数据\笔记\JavaSE\img\007.jpg)

​	java中集合类集定义在java.util包下，Collection接口是构造类集框架的基础，它声明所有类集将拥有的核心方法 

## 1.1 常用接口

- List——有序集合（“动态”数组）
- Set——无序集合
- Map——映射关系的集合数据结构

# 2、接口

## 2.1 Collection接口

  Collection是java集合框架最基本的接口。 Collection派生的常用子接口有List，Set，Queue。 

###  2.1.1 Collection集合与数组间的转换

​	集合 --->数组：toArray()

​	数组 --->集合：Arrays.asList(T ... t)

## 2.2 Iterator接口（迭代器）

​	定义：Iterator对象称为迭代器(设计模式的一种)，主要用于遍历 Collection 集合中的元素

​	迭代器模式：提供一种方法访问一个容器(container)对象中各个元素，而又不需暴露该对象的内部细节。迭代器模式，就是为容器而生。

### 2.2.1 使用Iterator遍历

```java
Iterator iterator = coll.iterator();
//hasNext():判断是否还下一个元素
while(iterator.hasNext()){
    //next():①指针下移 ②将下移以后集合位置上的元素返回
    System.out.println(iterator.next());
}
```

### 2.2.2 使用foreach循环遍历

```java
int[] arr = new int[]{1,2,3};
//for(数组元素的类型 局部变量 : 数组对象)
for(int i : arr){
    System.out.println(i);
}
```

### 2.2.3 将集合元素移除的唯一方法

```java
Iterator<Integer> itr = list.iterator();
while(itr.hasNext()) {
   itr.remove();
}
```



## 2.3 List接口

- List是有序集合，允许重复元素，可以根据索引进行访问。
- 实现List接口的常用类：LinkedList,ArrayList,Vector,Stack。

## 2.4 Set接口

- Set是无序集合，不包括重复元素。
- 常用的子类有HashSet，TreeSet。

## 2.5 Map接口

-  Map接口没有继承Collection接口，Map提供Key到value的关系映射。Map中不能含有重复的Key，每个key只能映射一个value 。
-  常用的子类有：HashMap。TreeMap，子接口有SortedMap。 

# 3、List的实现类

##  3.1 ArrayList、LinkedList、Vector

- ArrayList：List接口的主要实现类，线程不安全的，效率高；底层使用Object[] elementData存储
- LinkedList：适合插入、删除操作效率高于ArrayList，底层使用双向链表存储
- Vector：古老的实现类；线程安全的，效率低；底层使用Object[] elementData存储

## 3.2 Arraylist 与 LinkedList 

- 都不保证线程安全。
- `ArrayList`是实现了基于动态数组的数据结构，`LinkedList`基于双向链表的数据结构。
- 对于随机访问get和set，`ArrayList`优于`LinkedList`，因为LinkedList要移动指针。
- `LinkedList`不支持高效的随机元素访问，而`ArrayList `支持。
- 对于新增和删除操作add和remove，`LinedList`比较占优势，因为`ArrayList`要移动数据。
- `ArrayList`的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而`LinkedList`的空间花费则体现在它的每一个元素都需要消耗比`ArrayList`更多的空间（因为要存放直接后继和直接前驱以及数据）。

## 3.3 ArrayList 与 Vector 

  `Vector`类的所有方法都是同步的,  `Arraylist`不是同步的。 

# 4、Set的实现类

## 4.1 HashSet、LinkedHashSet、TreeSet

- HashSet：存储无序的、不可重复的数据， 基于HashMap实现，底层使用HashMap保存所有元素 。
- LinkedHashSet：HashSet的子类，遍历其内部数据时，可以按照添加的顺序遍历。
- TreeSet：可以照添加对象的指定属性，进行排序。

## 4.2 List转换为Set

```java
//方法一：
Set<Integer> set = new HashSet<Integer>(list);

//方法二：
Set<Integer> set = new TreeSet<Integer>(aComparator);
set.addAll(list);
```

### 4.2.1 移除 ArrayList中的重复元素

```java
HashSet<String> set=new HashSet<>();     //创建Set集合
set.addAll(list);                        //把list集合中的元素添加到Set集合中
list.clear();                           //清空list集合
list.addAll(set);                        //把Set集合中（去掉重复的元素）放到list中
```



# 5、Map的实现类

##  5.1  HashMap、TreeMap、Hashtable

- HashMap：Map的主要实现类；线程不安全的，效率高，允许存储null的key和value，不保证映射顺序。底层（数组+链表+红黑树）
- TreeMap： 基于红黑树的SortedMap接口实现，保证映射按照升序排列关键字，根据使用的构造方法不同，可能会按不同的方法排序。 
- Hashtable：古老的实现类；线程安全的，效率低；不能存储null的key和value，其他类似与HashMap

## 5.2 HashMap与HashTable

- `HashMap`是线程不安全的，`Hashtable`使用了synchronized关键字，是线程安全的。
- `HashMap`允许K/V都为null，后者K/V都不允许为null。
- `HashMap`继承自AbstractMap类，而`Hashtable`继承自Dictionary类。

# 6、Collections工具类

##  6.1 常用方法

- reverse(List)：反转 List 中元素的顺序
- shuffle(List)：对 List 集合元素进行随机排序
- sort(List)：根据元素的自然顺序对指定 List 集合元素升序排序
- sort(List，Comparator)：根据指定的 Comparator 产生的顺序对 List 集合元素进行排序
- swap(List，int， int)：将指定 list 集合中的 i 处元素和 j 处元素进行交换
- Object max(Collection)：根据元素的自然顺序，返回给定集合中的最大元素
- Object max(Collection，Comparator)：根据 Comparator 指定的顺序，返回给定集合中的最大元素
- Object min(Collection)
- Object min(Collection，Comparator)
- int frequency(Collection，Object)：返回指定集合中指定元素的出现次数
- void copy(List dest,List src)：将src中的内容复制到dest中
- boolean replaceAll(List list，Object oldVal，Object newVal)：使用新值替换 List 对象的所旧值

## 6.2 Collection 和 Collections

-  collection：（接口）是一个顶级父类接口 常用主要实现类有 list和set
-  collections：（工具类）是一个包装类, 是服务于collection的一个工具类 

