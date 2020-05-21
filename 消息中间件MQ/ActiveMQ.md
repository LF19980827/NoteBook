------

[TOC]

------

# 1、ActiveMQ概述

## 1.1 定义

是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并基于数据通信来进行分布式系统的集成。

​	通过提供消息传递和消息排队模型在分布式环境下提供应用解耦、弹性伸缩、冗余存储、流量削峰、异步通信、数据同步等功能

## 1.2 过程

​	发送者把消息发送给消息服务器，消息服务器将消息存放在若干**队列/主题**中，在合适的时候，消息服务器会将消息转发给接受者。

​	在这个过程中，**发送和接受是异步的**，也就是发送无需等待，而且发送者和接受者的生命周期也没有必然关系

​	尤其在发布pub/订阅sub模式下，也可以完成一对多的通信，即让一个消息有多个接受者。

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\1.1.png)

## 1.3 特点与作用

- 异步通信
- 应用解耦
- 流量削峰

eg:

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\1.2.png)



# 2、Linux环境下的安装

注：

```
后台：采用61616端口提供JMS服务
前端：采用8161端口提供管理控制台服务
```

​	1.官网下载 http://activemq.apache.org

​	2.解压安装

​	3.普通启动（在mq文件夹的bin目录下执行）

```
./activemq start
```

​	4.查看是否启动的三种方法

```
ps -ef|grep activemq

netstat -anp|grep 61616

lsof -i:61616
```

​	5.关闭

```
./activemq stop
```

​	6.[补充]带运行日志的启动方式

```
./activemq start>/myactivemq/run_activemq.log
```

​	7.apache activeMQ控制台[默认用户名密码是：admin/admin

```
http://localhost:8161
```



# 3、Java编码实现ActiveMQ通讯

## 3.1 了解JMS编码总体架构

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\2.1.png)

## 3.2 粗说目的地Destination

### 	1.两大模式特性

在点对点的消息传递域中，目的地被称为队列(queue)

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\2.2.png)

​	在发布订阅消息传递域中，目的地被称为主题(topic)

![2.2](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\2.3.png)

### 2.两大模式比较

|            | Topic模式                                                    | Queue模式                                                    |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 工作模式   | “订阅发布“模式，如果当前没有订阅者，消息将会被丢弃.如果有多个订阅者，那么这些订阅者都会收到消息 | “负载均衡模式，如果当前没有消费者,消息也不会丢弃;如果有多个消费者，那么一条消息也只会发送始其中一个消费者， 并且要求消费者ack信息. |
| 有无状态   | 无状态                                                       | Queue数据默认会在mq服务器上以文件形式保存，比如Active MQ一般保存在$AMQ_ HOME\data\kr- store\data下面。也可以配置成DB存储 |
| 传递完整性 | 如果没有订阅者，消息会被丢弃                                 | 消息不会丢弃                                                 |
| 处理效率   | 由于消息要按照订阅者的数量进行复制，所以处理性能会随着订阅者的增加而明显降低，并且还要结合不同消息协议自身的性能差异 | 由于条消息只发送给个消费者，所以就算消费者再多，性能也不会有明显降低。当然不同消息协议的具体性能也是有差异的 |



## 3.3 JMS开发的基本步骤

1. 创建一个connection factory
2. 通过connection factory来创建JMS connection
3. 启动JMS connection
4. 通过connection创建JMS session
5. 创建JMS destination
6. 创建JMS producer或者创建JMS message并设置destination
7. 创建JMS consumer或者是注册一个JMS message listener
8. 发送或者接受JMS message(s)
9. 关闭所有的JMS资源(connection, session, producer, consumer等)

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\2.4.png)



## 3.4 两种消费方式

- 同步阻塞方式(receive)：订阅者或接收者调用MessageConsumer的receive()方法来接收消息，receive 方法在能够接收到消息之前(或超时之前)将一直阻塞。
- 异步非阻塞方式(监听器onMessage()：订阅者或接收者通过MessageConsumer的setMessageListener(MessageListener listener)注册一个消息监听器，当消息到达之后，系统自动调用监听器MessageListener的onMessage(Message message)方法。

## 3.5 生产者与消费者编码（队列）

​	**1.项目需要导入的jar**

```xml
<!-- https://mvnrepository.com/artifact/org.apache.activemq/activemq-all -->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.12</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.xbean/xbean-spring -->
<dependency>
    <groupId>org.apache.xbean</groupId>
    <artifactId>xbean-spring</artifactId>
    <version>4.16</version>
</dependency>
```

​	**2.生产者编码**

```java
public class JmsProduce{
    public static final string ACTIVEMQ_URL="tcp://192.168.111.136:61616";
    public static final string QUEUE_NAME="queue01";
    
    public static void main(String[] args) throws JMSException{
        //1、创建连按工厂,按照給定url地址，采用默认用户名和密码
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnection actory(ACTIVEMQ_URL);
        //2、通过连接工厂，获得连接connection并启动访问
        Connection connection = activeMQconnectionFactory.createConnection();
        connection.start();
        //3、创建会话session
        //两个参数，第一个叫事务/第二个叫签收
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
    	//4、创建目的地(具体是队列还是主题topic)
        Queue queue = session.createQueue(QUEUE_NAME);
        
        //5、创建消息的生产者
        MessageProducer messageProducer = session.createProducer(queue);
        //6、通过使用messageProducer生产3条消息发送到MQ的队列里面
        for(inti=1;i<=3;i++){
			//7、创建消息,好比学生按照阳哥的要求写好的面试题消息，
			TextMessage textMessage = session.createTextMessage("msg---" + i);//理解为一个字符串
			//8、通过messageProducer发送给mq
			messageProducer.send(textMessage);
        }
		//9、关闭资源
		messageProducer.close();
		session.close();
		connection.close();
		System.out.println("*****消息发布到MQ完成");
    }
}
```

​	**3.消费者编码1**

```java
public class JmsComsumer{
    public static final string ACTIVEMQ_URL="tcp://192.168.111.136:61616";
    public static final string QUEUE_NAME="queue01";
    
    public static void main(String[] args) throws JMSException{
        //1、创建连按工厂,按照給定url地址，采用默认用户名和密码
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnection actory(ACTIVEMQ_URL);
        //2通过连接工厂，获得连接connection并启动访问
        Connection connection = activeMQconnectionFactory.createConnection();
        connection.start();
        //3创建会话session
        //两个参数，第一个叫事务/第二个叫签收
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
    	//4创建目的地(具体是队列还是主题topic)
        Queue queue = session.createQueue(QUEUE_NAME);
        
        //5创建消费者
		MessageConsumer messageConsumer = session.createConsumer(queue);
		while(true){
			TextMessage textMessage = (TextMessage)messageConsumer.receive();
			if(nu1ll != textMessage){
				System.out.println("****消费者接收到消息: "+textMessage.getText());
			}else{
				break;
            }
       	}
		messageConsumer.close();
		session.close();
		connection.close();
    }
}
```

# 4、JMS规范

## 4.1 JMS概述

### 	1.JavaEE概念

​	JavaEE是一套使用Java进行企业级应用开发的大家一致遵循的13个核心规范T业标准。JavaEE平台提供了一个基于组件的方法来加快设计、开发、装配及部署企业应用程序

- JDBC (Java Database)数据库连接
- JNDI (Java Naming and Diredtory Interfaces) Java 的命名和目录接口
- EJB ( Enterprise JavaBean)
- RMI (Remote Method Invoke)远程方法调用
- Java IDL(Interface Description Language)CORBA(Common Object Broker Architecture)接口定义语言/公用对象请求代理程序体系结构
- JSP (Java Server Pages)
- Servlet
- XML ( Extensible Markup Language)可扩展白标记语言
- JMS (Java Message Service) Java 消息服务
- JTA (Java Transaction API) Java 事务API
- JTS (Java Transaction Service) Java事务服务
- JavaMail
- JAF (JavaBean Activation Framework

### 	2.JMS概念

​	Java消息服务指的是两个应用程序之间进行异步通信的API，它为标准消息协议和消息服务提供了一组通用接口，包括创建、发送、读取消息等，用于支持JAVA应用程序开发。在JavaEE中，当两个应用程序使用JMS进行通信时，它们之间并不是直接相连的，而是通过一个共同的消息收发服务组件关联起来以达到解耦/异步削峰的效果。

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\4.1.png)

### 	3.JMS的四大元素

- JMSprovider：实现IMS接口和规范的消息中间件，也就是我们的MQ服务器
- JMS producer：消息生产者，创建和发送JMS消息的客户端应用
- JMS consumer：消息消费者，接收和处理JMs消息的客户端应用|
- JMS message：消息，由消息头、消息属性、消息体组成

## 4.2 JMS message重要属性介绍

### 1.消息头

- JMSDestination：消息发送的目的地，主要是指Queue和Topic
- JMSDeliveryMode：设置持久/非持久模式。
  - 持久性的消息:应该被传送“一次仅仅一次”，这就意味者如果JMS提供者现故障，该消息并不会丢失，它会在服务器恢复之后再次传递。
  - 非持久的消息:最多会传送一次，这意味这服务器出现故障，该消息将永远丢失。
- JMSExpiration ：可以设置消息在一定时间后过期，默认是永不过期
- JMSPriority ：消息优先级，从0-9十个级别，0到4是普通消息,5到9是加急消息。（JMS不要求MQ严格按照这十个优先级发送消息，但必须保证加急消息要先于普通消息到达。默认是4级。）
- JMSMessagelD：唯一识别每个消息的标识由MQ产生。

### 2.消息体

​	消息体：封装具体的消息数据（发送和接受的消息体类型必须一一致对应）

​	5种消息体格式：

- TextMessage：普通字符串消息，包含一个string
- MapMessage：一个Map类型的消息，key为string类型，而值为Java的基本类型
- BytesMessage：二进制数组消息，包含一个byte[]
- StreamMessage：Java数据流消息，用标准流操作来顺序的填充和读取
- ObjectMessage：对象消息，包含一个可序列化的Java对象

### 3.消息属性

​	定义：他们是以属性名和属性值对的形式制定的。可以将属性是为消息头得扩展，属性指定一些消息头没有包括的附加信息，比如可以在属性里指定消息选择器。（它们允许开发者添加有关消息的不透明附加信息。）

​	作用：

​		如果需要除消息头字段以外的值，那么可以使用消息属性

​		识别/去重/重点标注等操作非常有用的方法

​		它们还用于暴露消息选择器在消息过滤时使用的数据。

## 4.3JMS的可靠性

### 1.持久性（PERSISTENT）

（1）参数说明

```java
//非持久化:当服务器宕机，消息不存在。
messageProducer.setDeliveryMode(DeliveryMode.NON_ PERSISTENT);

//持久化:当服务器宕机，消息依然存在。
messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
```

（2）Queue(默认是持久的)：这是队列的的默认传送模式，此模式保证这些消息只被传送工次和成功使用一次。对于这些消息，可靠性是优先考虑的因素。可靠性的另一个重要方面是确保持久性消息传送至目标后，消息服务在向消费者传送它们之前不会丢失这些消息。

（3）Topic（持久化的）

1. 一定要先运行一次消费者，等于向MQ注册，类似我订阅了这个主题。	
2. 然后再运行生产者发送信息，此时,无论消费者是否在线，都会接收到，不在线的话，下次连接的时候， 会把没有收过的消息都接收下来。

### 2.事务

**事务偏生产者，签收偏消费者**

producer提交事务时

- false
  - 只要执行send,就进入到队列中。
  - 关闭事务，那第2个签收参数的设置需要有效
- true
  - 先执行send再执行commit，消息才被真正的提交到队列中。
  - 消息需要批量发送，需要缓冲区处理

### 3.签收（Acknowledge）

- 非事务状态时
  - 自动签收（默认）：Session.AUTO_ACKNOWLEDGE
  - 手动签收：Session.CLIENT_ACKNOWLEDGE
    - 客戶端调用acknowledge方法手动签收：message.acknowledge();
- 事务状态时：以事务的提交方式为标准，签收方式失效

事务与签收

- 在事务性会话中，当一个事务被成功提交则消息被自动签收。如果事务回滚，则消息会被再次传送。
- 非事务性会话中，消息何时被确认取决于创建会话时的应答模式(acknowledgementmode)

## 4.4 总结

### 1.点对点总结

​	点对点模型是基于队列的，生产者发消息到队列，消费者从队列接收消息，队列的存在使得消息的异步传输成为可能。

​	1.如果在Session关闭时有部分消息已被收到但还没有被签收(acknowledged)， 那当消费者下次连接到相同的队列时，这些消息还会被再次接收

​	2:队列可以长久地保存消息直到消费者收到消息。消费者不需要因为担心消息会丢失而时刻和队列保持激活的连接状态，充分体现了**异步传输模式**的优势

### 2.发布订阅总结

​	JMSPub/Sub模型定义了如何向一个内容节点发布和订阅消息，这些节点被称作topic

​	主题可以被认为是消息的传输中介，发布者(publisher)发布消息到主题，订阅者(subscribe)从主题订阅消息。

​	主题使得消息订阅者和消息发布者保持互相独立，不需要接触即可保证消息的传送。

​	**1.非持久订阅**

​	非持久订阅只有当客户端处于激活状态，也就是和MQ保持连接状态才能收到发送到某个主题的消息。

​	如果消费者处于离线状态，生产者发送的主题消息将会丢失作废，消费者永远不会收到。

​	**2.持久订阅**

​	客户端首先向MQ注册一个自己的身份ID识别号，当这个客户端处于离线时，生产者会为这个ID保存所有发送到主题的消息，当客户再次连接到MQ时会根据消费者的ID得到所有当自己处于离线时发送到主题的消息。

​	非持久订阅状态下，不能恢复或重新派送一个未签收的消息。

​	持久订阅才能恢复或重新派送一个未签收的消息。



# 5、ActiveMQ的Broker

## 5.1 Broker介绍

​	相当于一个ActiveM服务器实例

​	说白了，Broker其实就是实现了用代码的形式启动ActiveMQ将MQ嵌入到Java代码中，以便随时用随时启动，在用的时候再去启动这样能节省了资源，也保证了可靠性。

​	用ActiveMQ Broker作为独立的消息服务器来构建JAVA应用。

​	ActiveMQ也支持在vm中通信基于嵌入式的broker,能够无缝的集成其它java应用

## 5.2 嵌入式Broker

**1.pom.xml**

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.3</version>
</dependency>
```

**2.EmbedBroker**

```java
public class EmbedBroker{
	public static void main(String[] args)throws Exception{
		//ActiveMQ也支持在vm中通信基于嵌入式的broker
		BrokerService brokerService = new BrokerService();
		brokerService.setUseJmx(true);
		brokerService.addConnector("tcp://localhost:61616");
		brokerService.start();
    }
}

```

# 6、Spring整合ActiveMQ

### 1.Maven修改（添加Spring支持JMS的包）

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.3</version>
</dependency>

<!--activemq对JMS的支持，整合Spring和Activemq-->
<!-- https://mvnrepository.com/artifact/org.springframework/spring-jms -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>

<!--activemq所需要的pool包配置-->
<!-- https://mvnrepository.com/artifact/org.apache.activemq/activemq-pool -->
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-pool</artifactId>
    <version>5.15.12</version>
</dependency>

<!--Spring-AOP等相关jar不多阐述-->
```

### 2.Spring配置文件

```xml
<!-- 开启包的自动扫描-->
<context: component-scan base-package="com.hanhan.activemq"/>

<!--配置生产者-->
<bean id="jmsFactory"
class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
	<property name="connectionFactory">
		<!-- 真正可以产生Connection的ConnectionFactory，由对应的JMS服务厂商提供-->
		<bean class= "org.apache.activemq.ActiveMQConnectionFactory" >
			<property name="brokerURL" value="tcp://192.168 .111.136:61616"/>
		</bean>
	</property>
	<property name= "maxConnections" value="100"></ property>
</bean>

<!--这个是队列目的地，点对点的-->
<bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
	<constructor-arg index="0" value="spring-active-queue"/>
</bean>

<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等-->
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
	<property name="connectionFactory" ref="jmsFactory"/>
	<property name="defaultDestination" ref="destinationQueue"/>
	<property name="messageConverter">
		<bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
	</property>
</bean>
```

### 3.队列

生产者

```java
@Service
public class SpringMQ_Produce{

    @Autowired
	private ImsTemplate jmsTemplate;

    public static void main(string[] args){
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
		
        SpringMQ_Produce produce = (SpringMQ_Produce)ctx.getBean("springMQ_Produce");
		
        produce.jmsTemplate.send((session) -> {
			TextMessage textMessage = session.createTextMessage("**spring整合case..");
			return textMessage;
		});
		
        System. out .print1n("send task over");
    }
}
```

消费者

```java
@Service
public class SpringMQ_Consumer{

    @Autowired
	private ImsTemplate jmsTemplate;

    public static void main(string[] args){
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
		
        SpringMQ_Consumer consumer=(SpringMQ_Consumer)ctx.getBean("springMQ_Consumer");
        
		String retValue = (String)consumer.jmsTemplate.receiveAndConvert();
		
        System.out.println("消费者收到的消息:"+retValue);	
    }
}
```

### 4.主题

只需要修改部分配置文件

```xml
<!-- 这个是主题-->
<bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
<constructor-arg index="0" value=" spring-active-topic" />
</bean>|

<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等-->
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
	<property name="connectionFactory" ref="jmsFactory"/>
	<property name="defaultDestination" ref="destinationTopic"/><!--只用改这的ref-->
	<property name="messageConverter">
		<bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
	</property>
</bean>
```



### 5.不启动消费者，通过配置监听完成

**只需要启动生产者，消费者不用启动，自动会监听记录**

1.修改部分Spring配置文件

```xml
<!--配置监听程序-->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessagelistenerContainer">
	<property name="connectionFactory" ref="jmsFactory"/>
	<property name="destination" ref="destinationTopic"/>
	<!-- public class MyMessageL istener implements MessageListener-->
	<property name="messagelistener" ref= "myMessageListener"/>
</bean>
```

2.编写一个类实现消息监听

```java
@Component
public class MyMessageListener implements MessageListener{
	@Override
	public void onMessage(Message message){
		if(null != message && message instanceof TextMessage){
			TextMessage textMessage = (TextMessage) message;
			try{
       			System.out.println(textMessage.getText());
			} catch (JMSException e) {
				e. printStackTrace();
			}
		}
	}
}
```



# 7、SpringBoot整合ActiveMQ

## 7.1 队列

### 7.1.1 生产者

1. pom.xml
2. application.yml
3. 配置bean
4. 生产者
5. 主启动类
6. 测试单元

1.pom.xml

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-activemq -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>
```

2.application.yml

```yaml
spring:
	activemq:
		broker-url: tcp://192.168.111.136:61616 #自己的MQ服务器地址，用自己的
		user: admin
		password: admin
	jms:
		pub-sub-domain: false	# false = Queue   true = Topic

#自己定义队列名称
myqueue: boot-activemq-queue
```

3.配置bean

```java
@Component
@EnableJms
public class ConfigBean{
	
    @Value("${myqueue}")
	private String myQueue;
	
	@Bean
	public Queue queue(){
        return new ActiveMQQueue(myQueue);
    }		
}
```

4.生产者

```java
@Component
public class Queue_Produce{

    @Autowired
	private JmsMessagingTemplate jmsMessagingTemplate;
	@Autowired
	private Queue queue;
    
	public void produceMsg(){
        jmsMessagingTemplate.convertAndSend(queue,"一条消息");
		System.out.println("produceMsg task is over");
	}
}
```

5.主启动类

```java
@SpringBootApplication
public class MainApp_Produce{
	public static void main(String[] args){
		SpringApplication.run(MainApp_Produce.class,args);
	}
}
```

6.测试单元

```java
@SpringBootTest(classes = MainApp_Produce.class)
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
public class TestActiveMQ{

    @Resource
	private Queue_Produce queue_produce;
	
    @Test
	public void testSend() throws Exception{
        queue_produce.produceMsg();
    }
}
```

### 7.1.2 生产者之定时投递

1.修改生产者（新增定时投递方法）

```java
//添加定时投递方法

//间隔时间3秒钟定投
@scheduled(fixedDelay = 3000)
public void produceMsgScheduled(){
    jmsMessagingTemplate. convertAndSend(queue,"一条消息");
    System.out.println("send ok");
}
```

2.修改主启动类（直接启动主启动类，间隔发送）

```java
//添加注解
@EnableScheduling
```

### 7.1.3 消费者

​	大部分步骤同生产者，这里只阐述不一样的

1.消费者

```java
@Component
public class Queue_consumer{
	@JmsListener(destination = "${myqueue}")
	public void receive(TextMessage textMessage) throws JMSException{
        system.out.println("消费者收到消息:"+textMessage.getText());
    }		
}
```



## 7.2 主题

### 7.2.1 生产者

部分与队列相同，只阐述不同点

1.application.yml

```yaml
pub-sub-domain: true	# false = Queue   true = Topic

#自己定义的队列名称
myTopic: boot-activemq-topic
```

2.配置bean

```java
@Component
public class ConfigBean{
	
    @Value("${myTopic}")
	private String topicName;
	
    @Bean
	public Topic topic(){
		return new ActiveMQTopic(topicName);
	}
}
```

3.生产者

```java
@Component
public class Topic_Produce{

    @Autowired
	private JmsMessagingTemplate jmsMessagingTemplate; 
	@Autowired
	private Topic topic;

    @Scheduled(fixedDelay = 3000)
	public void produceTopic(){
		jmsMessagingTemplate.convertAndsend(topic,"主题消息");
    }
}
```

### 7.2.2 消费者

与上大致相同，不再阐述

# 8、ActiveMQ的传输协议

## 8.1 网络协议的种类

| 协议    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| TCP[重] | 默认的协议，性能相对可以                                     |
| NIO[重] | 基于TCP协议之.上的，进行了扩展和优化，具有更好的扩展性       |
| UDP     | 性能比TCP更好，但是不具有可靠性                              |
| SSL     | 安全链接                                                     |
| HTTP    | 基于HTTP或者HTTPS                                            |
| VM      | VM本身不是协议，当客户端和代理在同一个Java虚拟机(VM)中运行时,他们之间需要通信,但不想占用网络通道，而是直接通信，可以使用该方式 |

## 8.2 使用NIO传输协议

activemq默认支持5种协议（openwire为默认的协议）

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\8.1.jpg)

1.修改activemq.xml（添加nio）

```xml
<transportConnector name="nio" uri="nio://0.0.0.0:61618?trace=true"/>
```

2.修改生产者与消费者配置的代码（端口：61618）

```yaml
broker-url: nio://192.168.111.136:61618 #自己的MQ服务器地址
```

## 8.3 NIO加强

​	URI格式头以"nio”开头， 表示这个端口使用以TCP协议为基础的NIO网络I0模型。但是这样的设置方式，只能使这个端口支持Openwire协议。

​	问题：怎么既让这个端口支持NIO网络IO模型，又让它支持多个协议呢?

​	解决：使用auto关键字，使用"+"符号来为端口设置多种特性（NIO加强）



1.在activemq.xml（添加auto+nio）

```xml
<transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61608?maximumConnections=1000&amp;
                wireFormat.maxFrameSize=104857600&amp;
                org.apache.activemq.transport.nio.SelectorManager.corePoolSize=20&amp;
                org.apache.activemq.transport.nio.SelectorManager.maximumPoolSize=50" />
```

2.修改生产者与消费者配置的代码(端口：61608)

```yaml
broker-url: nio://192.168.111.136:61608 #自己的MQ服务器地址
```

# 9、ActiveMQ的消息存储和持久化

## 9.1 MQ持久化介绍

​	为了避免意外宕机以后丢失信息，需要做到重启后可以恢复消息队列，消息系统一般都会采用持久化机制。ActiveMQ的消息持久化机制有JDBC，AMQ，KahaDB 和LevelDB，无论使用哪种持久化方式，消息的存储逻辑都是一致的。

​	就是在发送者将消息发送出去后，消息中心首先将消息存储到本地数据文件、内存数据库或者远程数据库等再试图将消息发送给接收者，成功则将消息从存储中删除，失败则继续尝试发送。

​	消息中心启动以后首先要检查指定的存储位置，如果有未发送成功的消息，则需要把消息发送出去。

## 9.2 MQ持久化机制的种类

- AMQ Message Store(了解):基于文件的存储方式，以前的默认消息存储方式
- KahaDB消息存储(默认)：基于日志文件的存储方式，现在的默认的存储方式
- JDBC消息存储：基于JDBC的存储方式
- LevelDB消息存储(了解)：基于文件的存储方式，使用LevelDB的索引方式
- JDBC Message store with ActiveMQ Journal：JDBC存储的加强,加入高速缓存机制，减少数据库读写负担
- Replicated LevelDB Store：基于LevelDB和Zookeeper的数据复制方式，用于Master-slave方式的首选数据复制方案。

## 9.3 KahaDB的存储原理

​	消息存储使用**一个事务日志**和仅仅用**一个索引文件**来存储它所有的地址。

​	kahadb在消息保存目录中有4类文件和一个lock

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\9.1.png)

- db-<Number> .log：KahaDB存储消息到预定义大小的数据记录文件中，文件命名为db-<Number>.log。当数据文件己满时，一个新的文件会随之创建，number数值也会随之递增，它随着消息数量的增多。当不再有引用到数据文件中的任何消息时，文件会被删除或归档。
- db.data：该文件包含了持久化的BTree索引，索引了消息数据记录中的消息,它是消息的索引文件，本质上是B-Tree (B树)，使用B-Tree作为索引指向db-<Number> .log里面存储的消息。
- db.free：当前db.data文件里哪些页面是空闲的，文件具体内容是所有空闲页的ID
- db.redo：用来进行消息恢复,如果KahaDB消息存储在强制退出后启动，用于恢复BTree索引。
- lock：文件锁，表示当前获得kahadb读写权限的broker。

## 9.4 JDBC配置mysql

1.添加mysql数据库的驱动包到lib文件夹（mysql-connector-java-5.1.38. jar）
2.修改activemq.xml配置文件

```xml
<!--将默认的kahaDB换成jdbcPersistenceAdapter-->
<persistenceAdapter>
	<jdbcPersistenceAdapter dataSource="#mysql-ds" createTablesOnStartup="true" />
</persistenceAdapter>

<!--
dataSource指定将要引用的持久化数据库的bean名称，
createTablesOnStartup是否在启动的时候创建数据表，默认值是true,
-->

```

3.在activemq.xml中添加数据库连接池配置

```xml
<!--插入位置</broker>与<import resource="jetty.xml">之间-->

<bean id="mysql-ds" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value=" com.mysql.jdbc.Driver"/>
	<property name="url" value="jdbc:mysql://localhost:3306/activemq?relaxAutoCommit=true"/>
	<property name="username" value="root"/>
	<property name="password" value="123456" />
	<property name="maxTotal" value="200"/>
	<property name="poolPreparedStatements" value="true" />
</bean>

```

4.创建数据库和表的说明

​	创建数据库

```sql
CREATE DATABASE activemq;
```

​	重启activemq服务后会默认生成三张表

- ACTIVEMQ_ MSGS：消息表
- ACTIVEMQ_ ACKS：存储订阅关系
- ACTIVEMQ_ LOCK：在集群环境中才有作用，这个表用于记录哪个Broker是Master Broker

(1)ACTIVEMQ_ MSGS表的说明

| 列          | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| ID          | 自增的数据库主键                                             |
| CONTAINER   | 消息的Destination                                            |
| MSGID_ PROD | 消息发送者的主键                                             |
| MSG_ SEQ    | 是发送消息的顺序，MSGID_ PROD+MSG_SEQ 可以组成JMS的MessagelD |
| EXPIRATION  | 消息的过期时间，存储的是从1970-01-01到现在的毫秒数           |
| MSG         | 消息本体的Java序列化对象的二进制数据                         |
| PRIORITY    | 优先级，从0-9，数值越大优先级越高                            |

(2)ACTIVEMQ_ ACKS表的说明

| 列              | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| CONTAINER       | 消息的Destination                                            |
| SUB_DEST        | 如果是使用Static集群，这个字段会有集群其他系统的信息         |
| CLIENT_ ID      | 每个订阅者都必须有一个唯一的客户端ID用以区分                 |
| SUB_ NAME       | 订阅者名称                                                   |
| SELECTOR        | 选择器，可以选择只消费满足条件的消息。条件可以用自定义属性实现，可支持多属性AND和OR操作 |
| LAST_ ACKED_ ID | 记录消费过的消息的ID                                         |

(3)ACTIVEMQ_ LOCK表的说明

| 列         | 含义               |
| ---------- | ------------------ |
| ID         | 自增的数据库主键   |
| BrokerName | 拥有锁的Broker名称 |

5.开启消息持久化

```java
//只有开启了消息持久化，消息才会进入数据库
//在生产者代码中添加
messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
```

6.注意

- 记得需要使用到的相关jar文件放置到ActiveMQ安装路径下的lib目录。mysql-jdbc驱动的jar包和对应的数据库连接池jar包
- 在jdbcPersistenceAdapter标签中设置了createTablesOnStartup属性为true时在第一次启动ActiveMQ时， ActiveMQ服务 节点会自动创建所需要的数据表。启动完成后可以去掉这个属性，或者更改create TablesOnStartup属性为false.
- “java.lang.llegalStateException: BeanFactory not itilized or already closed”这是因为您的操作系统的机器名中有“_”符号。请更改机器名并且重启后即可解决问题。

## 9.5 JDBC的加强

​	ActiveMQ Journal,使用**高速缓存写入技术**，大大提高了性能。
​	当消费者的消费速度能够及时跟上生产者消息的生产速度时，journal文件能够大大减少需要写入到DB中的消息。
​	eg：生产者生产了1000条消息，这1000条消息会保存到journal|文件，如果消费者的消费速度很快的情况下，在journal文 件还没有同步到DB之前，消费者已经消费了90%的以上的消息，那么这个时候只需要同步剩余的10%的消息到DBp如果消费者的消费速度很慢，这个时候journal文件可以使消息以批量方式写到DB。

```xml
<!--修改activemq.xml，使用journalPersistenceAdapterFactory覆盖jdbcPersistenceAdapter-->
<persistenceFactory>
	< journalPersistenceAdapterFactory
		journalLogFiles="4"
		journalLogFileSize="32768"
		useJournal="true"
		useQuickJournal="true"
		dataSource="#mysql-ds"
		dataDirectory="activemq-data"/>
</persistenceFactory>
```

# 10、ActiveMQ多节点集群

​	基于ZooKeeper和LevelDB搭建ActiveMQ集群。集群仅提供主备方式的高可用集群功能，避免单点故障。

​	三种集群方式：基于sharedFileSystem共享文件系统(KahaDB）、基于JDBC、基于可复制的LevelDB

​	下面采用zookeeper+Replicated LevelDB Store(可复制的LevelDB)

## 10.1 集群原理图

![](G:\个人数据\笔记\NoteBook\消息中间件MQ\img\10.1.png)

原理说明:
	使用ZooKeeper集群注册所有的ActiveMQ Broker但只有其中的一个Broker可以提供服务它将被视为Master，其他的Broker处于待机状态被视为Slave。
	如果Master因故障而不能提供服务ZooKeeper会从Slave中选举出一个Broker充当Master。Slave连接Master并同步他们的存储状态，Slave不接受客户端连接。所有的存储操作都将被复制到连接至Master的Slaves。
	如果Master宕机得到了最新更新的Slave会成为Master。故障节点在恢复后会重新加入到集群中并连接Master进入Slave模式。
	所有需要同步的消息操作都将等待存储状态被复制到其他法定节点的操作完成才能完成。

## 10.2 部署规划和步骤

1. 配置环境（JDK，activemq等）
2. 关闭防火墙
3. 具备zookeeper集群并可以成功启动
4. 建立集群部署规划列表
5. 创建3台集群目录
6. 修改管理控制台端口
7. hostname名字映射
8. ActiveMQ集群配置
9. 修改各节点的消息端口
10. 按顺序启动3各ActiveMQ节点
11. zookeeper集群的节点状态说明

**4.建立集群部署规划表**

| 主机            | Zookeeper集群端口 | AMQ集群bind端口            | AMQ消息tcp端口 | 管理控制台端口 | AMQ节点安装目录       |
| --------------- | ----------------- | -------------------------- | -------------- | -------------- | --------------------- |
| 192.168.111.136 | 2191              | bind="tcp://0.0.0.0:63631" | 61616          | 8161           | /mq_cluster/mq_node01 |
| 192.168.111.136 | 2192              | bind="tcp://0.0.0.0:63632" | 61617          | 8162           | /mq_cluster/mq_node02 |
| 192.168.111.136 | 2193              | bind="tcp://0.0.0.0:63633" | 61618          | 8163           | /mq_cluster/mq_node03 |

**5.创建3台集群目录**

```shell
mkdir /mq_cluster/

cd /mq_cluster/

cp -r /opt/apache-activemq-5.15.9 mq_node01

cp -r mq_ node01 mq_ node02

cp -r mq_ node01 mq_ node03
```

**6.修改管理控制台端口（修改conf/jetty.xml）**

```xml
<!--只需要修改node2和node3即可-->
<!--修改jetty.xml中jettyPort中port的value-->
<bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
	<!-- the default port number for the web console -->
	<property name= "host" value="0.0.0.0"/>
	<property name="port" value="8162"/>
</bean>
```

**7.hostname名字映射（映射的名字随意）**

```shell
vim /etc/hosts

#自己的IP 地址映射 例如
192.168.111.136 lfuser
```

**8.ActiveMQ集群配置(修改activemq.xml)**

(1)设置三个节点的BrokerName，要求一致

```xml
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="mybroker" dataDirectory="${activemq.data}">
```

(2)设置持久化配置

```xml
<!--将默认的kahaDB覆盖-->
<persistenceAdapter>
	<replicatedLevelDB
		directory="${activemq.data}/leveldb"
		replicas="3"
		bind="tcp://0.0.0.0:63631"
		zkAddress="localhost:2191,localhost:2192,localhost:2193"
		hostname="lfuser"
		sync="local_disk"
		zkPath="/activemq/leveldb-stores"/>
</persistenceAdapter>
```

**9.修改各节点的消息端口(修改activemq.xml)**

```xml
<transportConnector name="openwire" uri="tcp://0.0.0.0:61617?maximumConnections=1000&amp; wireFormat.maxFrameSize=104857600"/>
```



使用集群后，生产者和消费者的代码改动

```java
public static final String ACTIVEMQ_URL="failover:(tcp://192.168.111.136:61616, tcp://192.168.111.136:61617,tcp://192.168.111.136:61618)?randomize=false";

```



# 11、高级特性和大厂常考重点

## 11.1 异步投递与其回调函数

​	ActiveMQ默认使用异步发送的模式:除非明确指定使用同步发送的方式或者**在未使用事务的前提下发送持久化的消息**，这两种情况都是同步发送的。
​	如果你没有使用事务且发送的是持久化的消息，每一次发送都是同步发送的且会阻塞producer直到broker返回一个确认，表示消息已经被安全的持久化到磁盘。确认机制提供了消息安全的保障，但同时会阻塞客户端带来了很大的延时。**但异步消息并不保证能发送成功**

​	在快生产、慢消费的情况下使用

**1.设置异步投递**

```java
//在生产者中，创建ActiveMQConnectionFactory对象后，执行以下方法
activeMQConnectionFactory.setuseAsyncSend(true);
```

**2.使用回调函数确认异步发送成功**

```java
//在生产者代码中使用Asynccallback
public class JmsProduce{
    public static final string ACTIVEMQ_URL="tcp://192.168.111.136:61616";
    public static final string QUEUE_NAME="queue02";
    
    public static void main(String[] args) throws JMSException{
       
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnection actory(ACTIVEMQ_URL);
        activeMQConnectionFactory.setuseAsyncSend(true);
        
        Connection connection = activeMQconnectionFactory.createConnection();
        connection.start();
        
        
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
    	
        Queue queue = session.createQueue(QUEUE_NAME);
       
        //MessageProducer messageProducer = session.createProducer(queue);
         //生产者的类使用ActiveMQMessageProducer
        ActiveMQMessageProducer activeMQMessageProducer =(ActiveMQMessageProducer)session.createProducer(queue);

        TextMessage message=null;
        for(inti=1;i<=3;i++){
			message = session.createTextMessage("message--"+i);
            //加入ID用来确认消息是否发送成功
			message.setJMSMessageID(UUID.randomUUID().tostring()+"--id");
			string msgID = message.getIMSMessageID();
            //编写回调函数确认消息是否发送成功
			activeMQMessageProducer.send(message, new Asynccallback(){
                @override
				public void onSuccess(){
                    System.out.println(msgID+"has been ok send");
                }
				
                @Override
				public void onException(JMSException exception){
                    System.out.println(msgID+"fail to send to mq");
                }
            });
        }
		//9、关闭资源
		messageProducer.close();
		session.close();
		connection.close();
		System.out.println("*****消息发布到MQ完成");
    }
}
```

## 11.2延迟投递和定时投递

四大属性

| 属性名               | 类型   | 含义               |
| -------------------- | ------ | ------------------ |
| AMQ_SCHEDULED_DELAY  | long   | 延迟投递的时间     |
| AMQ_SCHEDULED_PERIOD | long   | 重复投递的时间间隔 |
| AMQ_SCHEDULED_REPEAT | int    | 重复投递次数       |
| AMQ_SCHEDULED_CRON   | String | Cron表达式         |

**1.修改activemq.xml配置**

```xml
<!--将schedulerSupport设置为true-->
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="mybroker" dataDirectory="${activemq.data}" schedulerSupport="true">
```

**2.修改生产者代码**

```java
//大部分代码和上述生产者代码相同，只写不同部分
public class JmsProduce{
    public static final string ACTIVEMQ_URL="tcp://192.168.111.136:61616";
    public static final string QUEUE_NAME="queue03";
    
    public static void main(String[] args) throws JMSException{
       //...
       
        long delay = 3*1000;	//延迟投递的时间
		long period = 4*1000;	//重复投递的时间间隔
		int repeat = 5;			//重复投递次数
        
        for(inti=1;i<=3;i++){
			
			TextMessage message = session.createTextMessage("msg---" + i);
			
            //添加属性
            message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY,delay);
			message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD,period) ;
			message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT,repeat);

            
			messageProducer.send(message);
        }
		//...
    }
}
```

## 11.3 消费重试机制

**1.具体哪些情况回引起消息重发**

- Client用了transactions且在session中调用了rollback()
- Client用了transactions且在调用commit()之前关闭或者没有commit
- Client在CLIENT_ ACKNOWLEDGE的传递模式下，在session中 调用了recover()

2.消息重发时间间隔和重发次数：默认间隔：1，次数：6（第一次不算，重发6次，总共发7次）

3.**有毒消息**（PoisonACK）：一个消息被redelivedred超过默认的最大重发次数(默认6次)时，消费端会给MQ发送一个” poison ack"表示这个消息有毒，告诉broker不要再发了。这个时候broker会把这个消息放到DLQ(死信队列)。

4.在消费者中自定义消息重发次数

```java
RedeliveryPolicy redeliveryPolicy = new RedeliveryPolicy();
redeliveryPolicy.setMaximumRedeliveries(3);//设置重发次数为3
```

**5.整合Spring时自定义消息重发机制属性**

```xml
<!-- 定义ReDelivery(重发机制)机制-->
<bean id="activeMQRedeliveryPolicy" class="org.apache.activemq.RedeliveryPolicy">
    
	<!--是否在每次尝试重新发送失败后,增长这个等待时间-->
	<property name="useExponentialBack0ff" value="true"></property>
    
	<!--重发次数,默认为6次这里设置为3次-->
	<property name="maximumRedeliveries" value="3"></property>
    
	<!--重发时间间隔,默认为1秒-->
	<property name="initialRedeliveryDelay" value="1000"></property>
    
	<!--第一次失败后重新发送之前等待500毫秒,第二次失败再等待500*2毫秒,这里的2就是value -->
	<property name="backOffMultiplier" value="2"></property>
    
	<!--最大传送延迟，只在useExponentialBack0ff. 为true时有效(V5.5) ,
	假设首次重连间隔为10ms倍数为2，那么第二次重连时间间隔为20ms，第三次重连时间间隔为40ms，
	当重连时间间隔大的最大重连时间间隔时，以后每次 重连时间间隔都为最大重连时间间隔。-->
	<property name="maximumRedeliveryDelay" value=" 1000" ></property>
    
</bean>

<!--创建连接工厂-->
<bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionbactory">
	<property name="brokerURL" value="tcp://localhost:61616" ></property>
   
    <!--引用重发机制-->
	<property name="redeliveryPolicy" ref="activeMQRedeliveryPolicy"/> 
</bean>
```

6.消息重发机制的具体属性说明(了解)

- collisionAvoidanceFactor：设置防止冲突范围的正负百分比，只有启用useCollisionAvoidance参数时才生效。也就是在延迟时间上再加一个时间波动范围。默认值为0.15
-  maximumRedeliveries：最大重传次数，达到最大重连次数后抛出异常。为-1时不限制次数，为0时表示不进行重传。默认值为6。
- maximumRedeliveryDelay：最大传送延迟，只在useExponentialBackOff为true时有效(V5.5)，假设首次重连间隔为10ms，倍数为2，那么第二次重连时间间隔为20ms，第三次重连时间间隔为40ms，当重连时间间隔大的最大重连时间间隔时，以后每次重连时间间隔都为最大重连时间间隔。默认为-1。
- initialRedeliveryDelay：初始重发延迟时间，默认1000L
- redeliveryDelay：重发延迟时间，当initialRedeliveryDelay=0时生效， 默认1000L
- useCollisionAvoidance：启用防止冲突功能，默认false
- useExponentialBackOff：启用指数倍数递增的方式增加延迟时间，默认false
- backOffMultiplier：重连时间间隔递增倍数，只有值大于1和启用useExponentialBackOff参数时才生效。默认是5

## 11.4 防止重复调用

​	网络延迟传输中，会造成进行MQ重试中，在重试过程中，可能会造成重复消费。

​	如果消息是做数据库的插入操作，给这个消息做一个唯一主键， 那么就算出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。
​	

如果上面情况还不行，准备一个第三服务方来做消费记录。以redis为例， 给消息分配一个全局id，只要消费过该消息，将<id,message>以K-V形式写入redis。那消费者开始消费前，先去redis中查询有没消费记录即可。