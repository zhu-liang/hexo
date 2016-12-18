---
title: ActiveMQ之初使用
date: 2016-12-18 23:14:56
tags: Java
---
## 前言
最近要调试Java端代码，涉及到Java端代码，就不得不提系统间通信。于是周末试了试运行了一个ActiveMQ在本地，同时用了ActiveMQ官网的Java代码段分别实现了一个Producer和一个Consumer。

## 安装
本着边动手边理解的原则，先安装了ActiveMQ到本地，
http://activemq.apache.org/getting-started.html
上面是Apache ActiveMQ的官网，非常详细的指导。照着安装完成启动服务访问本机8161端口。可以进入Admin界面看到不同的消息提示。

在Eclipse中新建一个项目，
使用了ActiveMQ官网的示例代码
``` Java
package system_design;

import org.apache.activemq.ActiveMQConnectionFactory;
 
import javax.jms.Connection;
import javax.jms.DeliveryMode;
import javax.jms.Destination;
import javax.jms.ExceptionListener;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.MessageProducer;
import javax.jms.Session;
import javax.jms.TextMessage;
 
/**
 * Hello world!
 */
public class Producer {
 
    public static void main(String[] args) throws Exception {
        thread(new HelloWorldProducer(), true);
        thread(new HelloWorldConsumer(), true);
        Thread.sleep(1000);
    }
 
    public static void thread(Runnable runnable, boolean daemon) {
        Thread brokerThread = new Thread(runnable);
        brokerThread.setDaemon(daemon);
        brokerThread.start();
    }
 
    public static class HelloWorldProducer implements Runnable {
        public void run() {
            try {
                // Create a ConnectionFactory
                ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory("vm://localhost");
 
                // Create a Connection
                Connection connection = connectionFactory.createConnection();
                connection.start();
 
                // Create a Session
                Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
 
                // Create the destination (Topic or Queue)
                Destination destination = session.createQueue("TEST.FOO");
 
                // Create a MessageProducer from the Session to the Topic or Queue
                MessageProducer producer = session.createProducer(destination);
                producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
 
                int iLoop = 10;
                while(iLoop > 0) {
	                // Create a messages
	                String text = "Hello world! From: " + Thread.currentThread().getName() + " : " + this.hashCode();
	                TextMessage message = session.createTextMessage(text);
	 
	                // Tell the producer to send the message
	                System.out.println("Sent message: "+text + message.hashCode() + " : " + Thread.currentThread().getName());
	                producer.send(message);
	                iLoop--;
	            }
                // Clean up
                session.close();
                connection.close();
            	}
            catch (Exception e) {
                System.out.println("Caught: " + e);
                e.printStackTrace();
            }
        }
    }
 
    public static class HelloWorldConsumer implements Runnable, ExceptionListener {
        public void run() {
            try {
 
                // Create a ConnectionFactory
                ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory("vm://localhost");
 
                // Create a Connection
                Connection connection = connectionFactory.createConnection();
                connection.start();
 
                connection.setExceptionListener(this);
 
                // Create a Session
                Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
 
                // Create the destination (Topic or Queue)
                Destination destination = session.createQueue("TEST.FOO");
 
                // Create a MessageConsumer from the Session to the Topic or Queue
                MessageConsumer consumer = session.createConsumer(destination);
 
                // Wait for a message
                int iLoop = 20;
                while(iLoop > 0){
	                Message message = consumer.receive(1000);
	 
	                if (message instanceof TextMessage) {
	                    TextMessage textMessage = (TextMessage) message;
	                    String text = textMessage.getText();
	                    System.out.println("Received: " + text);
	                } else {
	                    System.out.println("Received: " + message);
	                }
	                iLoop--;
                }
                consumer.close();
                session.close();
                connection.close();
            } catch (Exception e) {
                System.out.println("Caught: " + e);
                e.printStackTrace();
            }
        }
 
        public synchronized void onException(JMSException ex) {
            System.out.println("JMS Exception occured.  Shutting down client.");
        }
    }
}
```
类名叫Producer，其中却同时有producer和consumer.

运行之后的结果如下：
```Bash
log4j:WARN No appenders could be found for logger (org.apache.activemq.broker.jmx.ManagementContext).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Sent message: Hello world! From: Thread-0 : 19218348011371468768 : Thread-0
Sent message: Hello world! From: Thread-0 : 19218348011508660406 : Thread-0
Sent message: Hello world! From: Thread-0 : 192183480163826692 : Thread-0
Sent message: Hello world! From: Thread-0 : 1921834801397203739 : Thread-0
Sent message: Hello world! From: Thread-0 : 1921834801833296893 : Thread-0
Sent message: Hello world! From: Thread-0 : 1921834801173646490 : Thread-0
Sent message: Hello world! From: Thread-0 : 19218348011713437982 : Thread-0
Sent message: Hello world! From: Thread-0 : 1921834801397771968 : Thread-0
Sent message: Hello world! From: Thread-0 : 19218348012117604154 : Thread-0
Sent message: Hello world! From: Thread-0 : 1921834801967979797 : Thread-0
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: Hello world! From: Thread-0 : 1921834801
Received: null
Received: null
Received: null
Received: null
Received: null
Received: null
Received: null
Received: null
Received: null
Received: null
```

系统间通信之ActiveMQ
参考CSDN一篇一看就知道作者花了精力写出来的博客。地址如下
http://blog.csdn.net/yinwenjie/article/details/50698695
作者从运行机制到优化方法及集群配置给出了详细的分析，实在是翔实有料的一个系列。

