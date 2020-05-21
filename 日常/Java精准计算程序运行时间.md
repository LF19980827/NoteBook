## 1.Java精准计算程序运行时间

1，以毫秒计时

```java
long startTime=System.currentTimeMillis();   //获取开始时间

long endTime=System.currentTimeMillis(); //获取结束时间
 
System.out.println("程序运行时间： "+(end-start)+"ms");
```

 

2，以纳秒计时

```java
long startTime=System.nanoTime();   //获取开始时间
 
long endTime=System.nanoTime(); //获取结束时间
 
System.out.println("程序运行时间： "+(end-start)+"ns");
```

