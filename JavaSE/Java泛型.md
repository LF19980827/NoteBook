# 1、概念

​	泛型，即“参数化类型”。允许在定义类、接口时通过一个标识表示类中某个属性的类型或者是某个方法的返回值及参数类型。 Java中的泛型（**伪泛型**），只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦除。

# 2、泛型方法

```java
public static <E> void print(E[] Array) {
	for (E element : Array) {
		System.out.printf("%s ", element);
	}
}
```



# 3、泛型类

```java
public class Test<T> {
	private T t;
	public Test(T t) {
		this.t = t;
	}
	public T get() {
		return t;
	}
}
```



# 4、类型通配符(?)

​	(1) List<?> 相当于是List,List 等所有 List<具体类型实参>的父类 

 （2）G<? extends A> 可以作为G<A>和G<B>的父类，其中B是A的子类

 （3）G<? super A> 可以作为G<A>和G<B>的父类，其中B是A的父类

