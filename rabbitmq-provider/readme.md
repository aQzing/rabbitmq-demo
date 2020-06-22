安装：https://blog.csdn.net/L_PHP_J/article/details/95935743
RabbitMQ在linux服务器上安装（阿里云centos7.5）
置顶 2019-07-14 22:39:20 李_壮士 阅读数 14更多
分类专栏： RabbitMQ
版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
本文链接：https://blog.csdn.net/L_PHP_J/article/details/95935743
abbitMQ在linux服务器上安装（阿里云centos7.5）
1、安装Erlang语言运行环境
因为RabbitMQ是用Erlang语言编写，所以要先安装Erlang运行环境
下载地址：http://www.erlang.org/downloads ，根据系统选择下载，如下图：
在这里插入图片描述
将安装包上传至服务器，执行如下命令进行安装：
 cd /tmp
 mkdir -p /usr/local/erlang
 tar -xzvf otp_src_22.0.tar.gz
 cd otp_src_20.1
 ./configure --prefix=/usr/local/erlang --with-ssl --enable-threads --enable-smp-support --enable-kernel-poll  --enable-hipe --without-javac
 make -j8
 make install
1
2
3
4
5
6
7
设置环境变量
 vim /etc/profile
 #在末尾加入以下内容：
 set erlang environment
 export PAHT=$PATH:/usr/local/erlang/bin
 #使环境变量生效
 source /etc/profile
 #查看erlang是否安装成功
 erl
 #出现如下信息表示成功
 Erlang R16B03-1 (erts-5.10.4) [source] [64-bit] [async-threads:10] [hipe] [kernel-poll:false]
 Eshell V5.10.4  (abort with ^G)
1
2
3
4
5
6
7
8
9
10
11
由于网络问题，在官网下载安装包速度很慢，还可以使用yum进行安装，速度快步骤少~（如果能连外网个人推荐）
 yum -y install erlang
1
2、安装RabbitMQ
下载地址：https://www.rabbitmq.com/install-rpm.html#downloads ，根据系统选择下载
在这里插入图片描述- 将安装包上转至服务器，执行以下命令进行安装：
 yum install -y rabbitmq-server-3.7.16-1.el7.noarch.rpm
 #检查RabbitMQ是否安装成功
 rabbitmqctl status
 #设置开机启动(即设置为守护线程)
  chkconfig rabbitmq-server on
  #启动mq
  service rabbitmq-server start                     
  #查看rabbitmq状态
  systemctl status rabbitmq-server.service    
  #停止mq服务
  rabbitmqctl stop 
1
2
3
4
5
6
7
8
9
10
11
如果官网安装包下载太慢也可以直接通过yum命令直接安装（个人推荐）
  yum -y install rabbitmq-server
1
通过下面命令安装RabbitMQ管理界面
 #安装管理界面
 rabbitmq-plugins enable rabbitmq_management
 #如果成功会有如下信息
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management
Plugin configuration has changed. Restart RabbitMQ for changes to take effect.
[root@iZm5ebi13eostn9rmsxkdbZ local]# rabbitmqctl stop
Stopping and halting node rabbit@iZm5ebi13eostn9rmsxkdbZ ...
...done.
1
2
3
4
5
6
7
8
9
10
11
12
13
14
可以通过http:#ip#:15672进行访问，如果是阿里云需要在安全组配置里面新增15672（管理界面端口）和5672（rabbitmq server端口）端口的访问权限
在这里插入图片描述

管理账号
默认登陆账号是guest，密码是guest，可以通过如下命令进行管理账号
 #创建账号
  rabbitmqctl add_user root 123456
 #设置用户角色
  rabbitmqctl set_user_tags root administrator
 #设置用户权限
  rabbitmqctl set_permissions -p "/" root ".*" ".*" ".*"
 #设置完成后可以查看当前用户和角色(需要开启服务)
  rabbitmqctl list_users
 #修改密码
  rabbitmqctl change_password guest guest123
 #删除账号
  rabbitmqctl delete_user guest




----------------------------------------------
https://blog.csdn.net/qq_35387940/article/details/100514134
Springboot 整合RabbitMq ，用心看完这一篇就够了
置顶 2019-09-03 23:21:11 小目标青年 阅读数 5715更多
分类专栏： 跟我一起玩转 SpringBoot  RabbitMq
版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。
本文链接：https://blog.csdn.net/qq_35387940/article/details/100514134
该篇文章内容较多，包括有rabbitMq相关的一些简单理论介绍，provider消息推送实例，consumer消息消费实例，Direct、Topic、Fanout的使用，消息回调、手动确认等。 （但是关于rabbitMq的安装，就不介绍了）
 

在安装完rabbitMq后，输入http://ip:15672/ ，是可以看到一个简单后台管理界面的。


在这个界面里面我们可以做些什么？
可以手动创建虚拟host，创建用户，分配权限，创建交换机，创建队列等等，还有查看队列消息，消费效率，推送效率等等。

以上这些管理界面的操作在这篇暂时不做扩展描述，我想着重后面实例里会使用到。

首先先介绍一个简单的一个消息推送到接收的流程，提供一个简单的图：
  

JCccc-RabbitMq
RabbitMq -JCccc
黄色的圈圈就是我们的消息推送服务，将消息推送到 中间方框里面也就是 rabbitMq的服务器，然后经过服务器里面的交换机、队列等各种关系（后面会详细讲）将数据处理入列后，最终右边的蓝色圈圈消费者获取对应监听的消息。

常用的交换机有以下三种，因为消费者是从队列获取信息的，队列是绑定交换机的（一般），所以对应的消息推送/接收模式也会有以下几种：

Direct Exchange 

直连型交换机，根据消息携带的路由键将消息投递给对应队列。

大致流程，有一个队列绑定到一个直连交换机上，同时赋予一个路由键 routing key 。
然后当一个消息携带着路由值为X，这个消息通过生产者发送给交换机时，交换机就会根据这个路由值X去寻找绑定值也是X的队列。

Fanout Exchange

扇型交换机，这个交换机没有路由键概念，就算你绑了路由键也是无视的。 这个交换机在接收到消息后，会直接转发到绑定到它上面的所有队列。

Topic Exchange

主题交换机，这个交换机其实跟直连交换机流程差不多，但是它的特点就是在它的路由键和绑定键之间是有规则的。
简单地介绍下规则：

*  (星号) 用来表示一个单词 (必须出现的)
#  (井号) 用来表示任意数量（零个或多个）单词
通配的绑定键是跟队列进行绑定的，举个小例子
队列Q1 绑定键为 *.TT.*          队列Q2绑定键为  TT.#
如果一条消息携带的路由键为 A.TT.B，那么队列Q1将会收到；
如果一条消息携带的路由键为TT.AA.BB，那么队列Q2将会收到；

主题交换机是非常强大的，为啥这么膨胀？
当一个队列的绑定键为 "#"（井号） 的时候，这个队列将会无视消息的路由键，接收所有的消息。
当 * (星号) 和 # (井号) 这两个特殊字符都未在绑定键中出现的时候，此时主题交换机就拥有的直连交换机的行为。
所以主题交换机也就实现了扇形交换机的功能，和直连交换机的功能。

另外还有 Header Exchange 头交换机 ，Default Exchange 默认交换机，Dead Letter Exchange 死信交换机，这几个该篇不做暂讲述。

好了，一些简单的介绍到这里为止，  接下来我们来一起编码。
本次实例教程需要创建2个springboot项目，一个 rabbitmq-provider （生产者），一个rabbitmq-consumer（消费者）。

首先创建 rabbitmq-provider，

pom.xml里用到的jar依赖：

        <!--rabbitmq-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
然后application.yml：

server:
  port: 8021
spring:
  #给项目来个名字
  application:
    name: rabbitmq-provider
  #配置rabbitMq 服务器
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
    #虚拟host 可以不设置,使用server默认host
    virtual-host: JCcccHost
接着我们先使用下direct exchange(直连型交换机),创建DirectRabbitConfig.java：

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Configuration
public class DirectRabbitConfig {
 
    //队列 起名：TestDirectQueue
    @Bean
    public Queue TestDirectQueue() {
        return new Queue("TestDirectQueue",true);  //true 是否持久 
    }
 
    //Direct交换机 起名：TestDirectExchange
    @Bean
    DirectExchange TestDirectExchange() {
        return new DirectExchange("TestDirectExchange");
    }
 
    //绑定  将队列和交换机绑定, 并设置用于匹配键：TestDirectRouting
    @Bean
    Binding bindingDirect() {
        return BindingBuilder.bind(TestDirectQueue()).to(TestDirectExchange()).with("TestDirectRouting");
    }
}
然后写个简单的接口进行消息推送（根据需求也可以改为定时任务等等，具体看需求），SendMessageController.java：

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@RestController
public class SendMessageController {
 
    @Autowired
    RabbitTemplate rabbitTemplate;  //使用RabbitTemplate,这提供了接收/发送等等方法
 
    @GetMapping("/sendDirectMessage")
    public String sendDirectMessage() {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "test message, hello!";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String,Object> map=new HashMap<>();
        map.put("messageId",messageId);
        map.put("messageData",messageData);
        map.put("createTime",createTime);
        //将消息携带绑定键值：TestDirectRouting 发送到交换机TestDirectExchange
        rabbitTemplate.convertAndSend("TestDirectExchange", "TestDirectRouting", map);
        return "ok";
    }
 
 
}
把rabbitmq-provider项目运行，调用下接口:


因为我们目前还没弄消费者 rabbitmq-consumer，消息没有被消费的，我们去rabbitMq管理页面看看，是否推送成功：


再看看队列（界面上的各个英文项代表什么意思，可以自己查查哈，对理解还是有帮助的）：



很好，消息已经推送到rabbitMq服务器上面了。

接下来，创建rabbitmq-consumer项目：

pom.xml里的jar依赖：

        <!--rabbitmq-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
然后是 application.yml：

 
server:
  port: 8022
spring:
  #给项目来个名字
  application:
    name: rabbitmq-consumer
  #配置rabbitMq 服务器
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
    #虚拟host 可以不设置,使用server默认host
    virtual-host: JCcccHost
然后一样，创建DirectRabbitConfig.java：

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Configuration
public class DirectRabbitConfig {
 
    //队列 起名：TestDirectQueue
    @Bean
    public Queue TestDirectQueue() {
        return new Queue("TestDirectQueue",true);
    }
 
    //Direct交换机 起名：TestDirectExchange
    @Bean
    DirectExchange TestDirectExchange() {
        return new DirectExchange("TestDirectExchange");
    }
 
    //绑定  将队列和交换机绑定, 并设置用于匹配键：TestDirectRouting
    @Bean
    Binding bindingDirect() {
        return BindingBuilder.bind(TestDirectQueue()).to(TestDirectExchange()).with("TestDirectRouting");
    }
}
然后是创建消息接收监听类，DirectReceiver.java：

@Component
@RabbitListener(queues = "TestDirectQueue")//监听的队列名称 TestDirectQueue
public class DirectReceiver {
 
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("DirectReceiver消费者收到消息  : " + testMessage.toString());
    }
 
}
然后将rabbitmq-consumer项目运行起来，可以看到把之前推送的那条消息消费下来了：



然后可以再继续调用rabbitmq-provider项目的推送消息接口，可以看到消费者即时消费消息：



 

 

接着，我们使用Topic Exchange 主题交换机。

在rabbitmq-provider项目里面创建TopicRabbitConfig.java：

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
 
@Configuration
public class TopicRabbitConfig {
    //绑定键
    public final static String man = "topic.man";
    public final static String woman = "topic.woman";
 
    @Bean
    public Queue firstQueue() {
        return new Queue(TopicRabbitConfig.man);
    }
 
    @Bean
    public Queue secondQueue() {
        return new Queue(TopicRabbitConfig.woman);
    }
 
    @Bean
    TopicExchange exchange() {
        return new TopicExchange("topicExchange");
    }
 
 
    //将firstQueue和topicExchange绑定,而且绑定的键值为topic.man
    //这样只要是消息携带的路由键是topic.man,才会分发到该队列
    @Bean
    Binding bindingExchangeMessage() {
        return BindingBuilder.bind(firstQueue()).to(exchange()).with(man);
    }
 
    //将secondQueue和topicExchange绑定,而且绑定的键值为用上通配路由键规则topic.#
    // 这样只要是消息携带的路由键是以topic.开头,都会分发到该队列
    @Bean
    Binding bindingExchangeMessage2() {
        return BindingBuilder.bind(secondQueue()).to(exchange()).with("topic.#");
    }
 
}
然后添加多2个接口，用于推送消息到主题交换机：

    @GetMapping("/sendTopicMessage1")
    public String sendTopicMessage1() {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "message: M A N ";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String, Object> manMap = new HashMap<>();
        manMap.put("messageId", messageId);
        manMap.put("messageData", messageData);
        manMap.put("createTime", createTime);
        rabbitTemplate.convertAndSend("topicExchange", "topic.man", manMap);
        return "ok";
    }
 
    @GetMapping("/sendTopicMessage2")
    public String sendTopicMessage2() {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "message: woman is all ";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String, Object> womanMap = new HashMap<>();
        womanMap.put("messageId", messageId);
        womanMap.put("messageData", messageData);
        womanMap.put("createTime", createTime);
        rabbitTemplate.convertAndSend("topicExchange", "topic.woman", womanMap);
        return "ok";
    }
}
生产者这边已经完事，先不急着运行，在rabbitmq-consumer项目上，创建TopicManReceiver.java：

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.util.Map;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Component
@RabbitListener(queues = "topic.man")
public class TopicManReceiver {
 
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("TopicManReceiver消费者收到消息  : " + testMessage.toString());
    }
}
再创建一个TopicTotalReceiver.java：

package com.elegant.rabbitmqconsumer.receiver;
 
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.util.Map;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
 
@Component
@RabbitListener(queues = "topic.woman")
public class TopicTotalReceiver {
 
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("TopicTotalReceiver消费者收到消息  : " + testMessage.toString());
    }
}
同样，加主题交换机的相关配置，TopicRabbitConfig.java：

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
 
@Configuration
public class TopicRabbitConfig {
    //绑定键
    public final static String man = "topic.man";
    public final static String woman = "topic.woman";
 
    @Bean
    public Queue firstQueue() {
        return new Queue(TopicRabbitConfig.man);
    }
 
    @Bean
    public Queue secondQueue() {
        return new Queue(TopicRabbitConfig.woman);
    }
 
    @Bean
    TopicExchange exchange() {
        return new TopicExchange("topicExchange");
    }
 
 
    //将firstQueue和topicExchange绑定,而且绑定的键值为topic.man
    //这样只要是消息携带的路由键是topic.man,才会分发到该队列
    @Bean
    Binding bindingExchangeMessage() {
        return BindingBuilder.bind(firstQueue()).to(exchange()).with(man);
    }
 
    //将secondQueue和topicExchange绑定,而且绑定的键值为用上通配路由键规则topic.#
    // 这样只要是消息携带的路由键是以topic.开头,都会分发到该队列
    @Bean
    Binding bindingExchangeMessage2() {
        return BindingBuilder.bind(secondQueue()).to(exchange()).with("topic.#");
    }
 
}

然后把rabbitmq-provider，rabbitmq-consumer两个项目都跑起来，先调用/sendTopicMessage1  接口：



然后看消费者rabbitmq-consumer的控制台输出情况：
TopicManReceiver监听队列1，绑定键为：topic.man
TopicTotalReceiver监听队列2，绑定键为：topic.#
而当前推送的消息，携带的路由键为：topic.man  

所以可以看到两个监听消费者receiver都成功消费到了消息，因为这两个recevier监听的队列的绑定键都能与这条消息携带的路由键匹配上。



接下来调用接口/sendTopicMessage2:



然后看消费者rabbitmq-consumer的控制台输出情况：
TopicManReceiver监听队列1，绑定键为：topic.man
TopicTotalReceiver监听队列2，绑定键为：topic.#
而当前推送的消息，携带的路由键为：topic.woman

所以可以看到两个监听消费者只有TopicTotalReceiver成功消费到了消息。



 

接下来是使用Fanout Exchang 扇型交换机。

同样地，先在rabbitmq-provIder项目上创建FanoutRabbitConfig.java：

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
 
@Configuration
public class FanoutRabbitConfig {
 
    /**
     *  创建三个队列 ：fanout.A   fanout.B  fanout.C
     *  将三个队列都绑定在交换机 fanoutExchange 上
     *  因为是扇型交换机, 路由键无需配置,配置也不起作用
     */
 
 
    @Bean
    public Queue queueA() {
        return new Queue("fanout.A");
    }
 
    @Bean
    public Queue queueB() {
        return new Queue("fanout.B");
    }
 
    @Bean
    public Queue queueC() {
        return new Queue("fanout.C");
    }
 
    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }
 
    @Bean
    Binding bindingExchangeA() {
        return BindingBuilder.bind(queueA()).to(fanoutExchange());
    }
 
    @Bean
    Binding bindingExchangeB() {
        return BindingBuilder.bind(queueB()).to(fanoutExchange());
    }
 
    @Bean
    Binding bindingExchangeC() {
        return BindingBuilder.bind(queueC()).to(fanoutExchange());
    }
}
然后是写一个接口用于推送消息，
 

    @GetMapping("/sendFanoutMessage")
    public String sendFanoutMessage() {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "message: testFanoutMessage ";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String, Object> map = new HashMap<>();
        map.put("messageId", messageId);
        map.put("messageData", messageData);
        map.put("createTime", createTime);
        rabbitTemplate.convertAndSend("fanoutExchange", null, map);
        return "ok";
    }
接着在rabbitmq-consumer项目里加上消息消费类，

FanoutReceiverA.java：

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.util.Map;
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Component
@RabbitListener(queues = "fanout.A")
public class FanoutReceiverA {
 
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("FanoutReceiverA消费者收到消息  : " +testMessage.toString());
    }
 
}
FanoutReceiverB.java：

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.util.Map;
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Component
@RabbitListener(queues = "fanout.B")
public class FanoutReceiverB {
 
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("FanoutReceiverB消费者收到消息  : " +testMessage.toString());
    }
 
}
FanoutReceiverC.java：

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;
import java.util.Map;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Component
@RabbitListener(queues = "fanout.C")
public class FanoutReceiverC {
 
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("FanoutReceiverC消费者收到消息  : " +testMessage.toString());
    }
 
}
然后加上扇型交换机的配置类，FanoutRabbitConfig.java：

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Configuration
public class FanoutRabbitConfig {
 
    /**
     *  创建三个队列 ：fanout.A   fanout.B  fanout.C
     *  将三个队列都绑定在交换机 fanoutExchange 上
     *  因为是扇型交换机, 路由键无需配置,配置也不起作用
     */
 
 
    @Bean
    public Queue queueA() {
        return new Queue("fanout.A");
    }
 
    @Bean
    public Queue queueB() {
        return new Queue("fanout.B");
    }
 
    @Bean
    public Queue queueC() {
        return new Queue("fanout.C");
    }
 
    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }
 
    @Bean
    Binding bindingExchangeA() {
        return BindingBuilder.bind(queueA()).to(fanoutExchange());
    }
 
    @Bean
    Binding bindingExchangeB() {
        return BindingBuilder.bind(queueB()).to(fanoutExchange());
    }
 
    @Bean
    Binding bindingExchangeC() {
        return BindingBuilder.bind(queueC()).to(fanoutExchange());
    }
}
最后将rabbitmq-provider和rabbitmq-consumer项目都跑起来，调用下接口/sendFanoutMessage ：



然后看看rabbitmq-consumer项目的控制台情况：



可以看到只要发送到 fanoutExchange 这个扇型交换机的消息， 三个队列都绑定这个交换机，所以三个消息接收类都监听到了这条消息。


到了这里其实三个常用的交换机的使用我们已经完毕了，那么接下来我们继续讲讲消息的回调，其实就是消息确认（生产者推送消息成功，消费这接收消息成功）。
 

在rabbitmq-provider项目的application.yml文件上，加上消息确认的配置项后：
 

server:
  port: 8021
spring:
  #给项目来个名字
  application:
    name: rabbitmq-provider
  #配置rabbitMq 服务器
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
    #虚拟host 可以不设置,使用server默认host
    virtual-host: JCcccHost
    #消息确认配置项
 
    #确认消息已发送到交换机(Exchange)
    publisher-confirms: true
    #确认消息已发送到队列(Queue)
    publisher-returns: true
然后是配置相关的消息确认回调函数，RabbitConfig.java：

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Configuration
public class RabbitConfig {
 
    @Bean
    public RabbitTemplate createRabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate();
        rabbitTemplate.setConnectionFactory(connectionFactory);
        //设置开启Mandatory,才能触发回调函数,无论消息推送结果怎么样都强制调用回调函数
        rabbitTemplate.setMandatory(true);
 
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                System.out.println("ConfirmCallback:     "+"相关数据："+correlationData);
                System.out.println("ConfirmCallback:     "+"确认情况："+ack);
                System.out.println("ConfirmCallback:     "+"原因："+cause);
            }
        });
 
        rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
            @Override
            public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
                System.out.println("ReturnCallback:     "+"消息："+message);
                System.out.println("ReturnCallback:     "+"回应码："+replyCode);
                System.out.println("ReturnCallback:     "+"回应信息："+replyText);
                System.out.println("ReturnCallback:     "+"交换机："+exchange);
                System.out.println("ReturnCallback:     "+"路由键："+routingKey);
            }
        });
 
        return rabbitTemplate;
    }
 
}
到这里，生产者推送消息的消息确认调用回调函数已经完毕。
可以看到上面写了两个回调函数，一个叫 ConfirmCallback ，一个叫 RetrunCallback；
那么以上这两种回调函数都是在什么情况会触发呢？

先从总体的情况分析，推送消息存在四种情况：

①消息推送到server，但是在server里找不到交换机
②消息推送到server，找到交换机了，但是没找到队列
③消息推送到sever，交换机和队列啥都没找到
④消息推送成功

那么我先写几个接口来分别测试和认证下以上4种情况，消息确认触发回调函数的情况：

①消息推送到server，但是在server里找不到交换机
写个测试接口，把消息推送到名为‘non-existent-exchange’的交换机上（这个交换机是没有创建没有配置的）：

    @GetMapping("/TestMessageAck")
    public String TestMessageAck() {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "message: non-existent-exchange test message ";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String, Object> map = new HashMap<>();
        map.put("messageId", messageId);
        map.put("messageData", messageData);
        map.put("createTime", createTime);
        rabbitTemplate.convertAndSend("non-existent-exchange", "TestDirectRouting", map);
        return "ok";
    }
调用接口，查看rabbitmq-provuder项目的控制台输出情况（原因里面有说，没有找到交换机'non-existent-exchange'）：

2019-09-04 09:37:45.197 ERROR 8172 --- [ 127.0.0.1:5672] o.s.a.r.c.CachingConnectionFactory       : Channel shutdown: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'non-existent-exchange' in vhost 'JCcccHost', class-id=60, method-id=40)
ConfirmCallback:     相关数据：null
ConfirmCallback:     确认情况：false
ConfirmCallback:     原因：channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'non-existent-exchange' in vhost 'JCcccHost', class-id=60, method-id=40)
    结论： ①这种情况触发的是 ConfirmCallback 回调函数。

 ②消息推送到server，找到交换机了，但是没找到队列  
这种情况就是需要新增一个交换机，但是不给这个交换机绑定队列，我来简单地在DirectRabitConfig里面新增一个直连交换机，名叫‘lonelyDirectExchange’，但没给它做任何绑定配置操作：

    @Bean
    DirectExchange lonelyDirectExchange() {
        return new DirectExchange("lonelyDirectExchange");
    }
然后写个测试接口，把消息推送到名为‘lonelyDirectExchange’的交换机上（这个交换机是没有任何队列配置的）：

    @GetMapping("/TestMessageAck2")
    public String TestMessageAck2() {
        String messageId = String.valueOf(UUID.randomUUID());
        String messageData = "message: lonelyDirectExchange test message ";
        String createTime = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        Map<String, Object> map = new HashMap<>();
        map.put("messageId", messageId);
        map.put("messageData", messageData);
        map.put("createTime", createTime);
        rabbitTemplate.convertAndSend("lonelyDirectExchange", "TestDirectRouting", map);
        return "ok";
    }
调用接口，查看rabbitmq-provuder项目的控制台输出情况：

ReturnCallback:     消息：(Body:'{createTime=2019-09-04 09:48:01, messageId=563077d9-0a77-4c27-8794-ecfb183eac80, messageData=message: lonelyDirectExchange test message }' MessageProperties [headers={}, contentType=application/x-java-serialized-object, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0])
ReturnCallback:     回应码：312
ReturnCallback:     回应信息：NO_ROUTE
ReturnCallback:     交换机：lonelyDirectExchange
ReturnCallback:     路由键：TestDirectRouting
ConfirmCallback:     相关数据：null
ConfirmCallback:     确认情况：true
ConfirmCallback:     原因：null
可以看到这种情况，两个函数都被调用了；
这种情况下，消息是推送成功到服务器了的，所以ConfirmCallback对消息确认情况是true；
而在RetrunCallback回调函数的打印参数里面可以看到，消息是推送到了交换机成功了，但是在路由分发给队列的时候，找不到队列，所以报了错误 NO_ROUTE 。
  结论：②这种情况触发的是 ConfirmCallback和RetrunCallback两个回调函数。

③消息推送到sever，交换机和队列啥都没找到 
这种情况其实一看就觉得跟①很像，没错 ，③和①情况回调是一致的，所以不做结果说明了。
  结论： ③这种情况触发的是 ConfirmCallback 回调函数。

 ④消息推送成功
那么测试下，按照正常调用之前消息推送的接口就行，就调用下 /sendFanoutMessage接口，可以看到控制台输出：

ConfirmCallback:     相关数据：null
ConfirmCallback:     确认情况：true
ConfirmCallback:     原因：null
结论： ④这种情况触发的是 ConfirmCallback 回调函数。


以上是生产者推送消息的消息确认 回调函数的使用介绍（可以在回调函数根据需求做对应的扩展或者业务数据处理）。

接下来我们继续， 消费者接收到消息的消息确认机制。

和生产者的消息确认机制不同，因为消息接收本来就是在监听消息，符合条件的消息就会消费下来。
所以，消息接收的确认机制主要存在三种模式：

①自动确认， 这也是默认的消息确认情况。   AcknowledgeMode.NONE
RabbitMQ成功将消息发出（即将消息成功写入TCP Socket）中立即认为本次投递已经被正确处理，不管消费者端是否成功处理本次投递。
所以这种情况如果消费端消费逻辑抛出异常，也就是消费端没有处理成功这条消息，那么就相当于丢失了消息。
一般这种情况我们都是使用try catch捕捉异常后，打印日志用于追踪数据，这样找出对应数据再做后续处理。

② 不确认， 这个不做介绍
③ 手动确认 ， 这个比较关键，也是我们配置接收消息确认机制时，多数选择的模式。
消费者收到消息后，手动调用basic.ack/basic.nack/basic.reject后，RabbitMQ收到这些消息后，才认为本次投递成功。
basic.ack用于肯定确认 
basic.nack用于否定确认（注意：这是AMQP 0-9-1的RabbitMQ扩展） 
basic.reject用于否定确认，但与basic.nack相比有一个限制:一次只能拒绝单条消息 
消费者端以上的3个方法都表示消息已经被正确投递，但是basic.ack表示消息已经被正确处理，但是basic.nack,basic.reject表示没有被正确处理，但是RabbitMQ中仍然需要删除这条消息。 


看了上面这么多介绍，接下来我们一起配置下，看看一般的消息接收 手动确认是怎么样的。
​​​​​​

之前介绍用了很多个交换，现在我们就先给直连型交换机添加消息接收确认机制，
新建MessageListenerConfig.java上添加代码相关的配置代码（可以看到注释掉的代码，就是给扇型交换机配置消息确认，只用在这个里面继续添加对应的队列和对应的接收类即可，当然对应的接收类也需要跟后面介绍一样加上对应方法）：

import com.elegant.rabbitmqconsumer.receiver.DirectReceiver;
import com.elegant.rabbitmqconsumer.receiver.FanoutReceiverA;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/4
 * @Description :
 **/
@Configuration
public class MessageListenerConfig {
 
    @Autowired
    private CachingConnectionFactory connectionFactory;
    @Autowired
    private DirectReceiver directReceiver;//Direct消息接收处理类
//    @Autowired
//    FanoutReceiverA fanoutReceiverA;//Fanout消息接收处理类A
    @Autowired
    DirectRabbitConfig directRabbitConfig;
//    @Autowired
//    FanoutRabbitConfig fanoutRabbitConfig;
    @Bean
    public SimpleMessageListenerContainer simpleMessageListenerContainer() {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        container.setConcurrentConsumers(1);
        container.setMaxConcurrentConsumers(1);
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL); // RabbitMQ默认是自动确认，这里改为手动确认消息
        container.setQueues(directRabbitConfig.TestDirectQueue());
        container.setMessageListener(directReceiver);
//        container.addQueues(fanoutRabbitConfig.queueA());
//        container.setMessageListener(fanoutReceiverA);
        return container;
    }
}

然后在直连型交换机的消息接收处理类上需要添加相关的消息手动确认代码DirectReceiver.java：

import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.listener.api.ChannelAwareMessageListener;
import org.springframework.stereotype.Component;
 
import java.util.HashMap;
import java.util.Map;
 
 
/**
 * @Author : JCccc
 * @CreateTime : 2019/9/3
 * @Description :
 **/
@Component
@RabbitListener(queues = "TestDirectQueue")//监听的队列名称 TestDirectQueue
public class DirectReceiver implements ChannelAwareMessageListener {
 
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            //因为传递消息的时候用的map传递,所以将Map从Message内取出需要做些处理
            String msg = message.toString();
            String[] msgArray = msg.split("'");//可以点进Message里面看源码,单引号直接的数据就是我们的map消息数据
            Map<String, String> msgMap = mapStringToMap(msgArray[1].trim());
            String messageId=msgMap.get("messageId");
            String messageData=msgMap.get("messageData");
            String createTime=msgMap.get("createTime");
            System.out.println("messageId:"+messageId+"  messageData:"+messageData+"  createTime:"+createTime);
            channel.basicAck(deliveryTag, true);
//			channel.basicReject(deliveryTag, true);//为true会重新放回队列
        } catch (Exception e) {
            channel.basicReject(deliveryTag, false);
            e.printStackTrace();
        }
    }
 
    //{key=value,key=value,key=value} 格式转换成map
    private Map<String, String> mapStringToMap(String str) {
        str = str.substring(1, str.length() - 1);
        String[] strs = str.split(",");
        Map<String, String> map = new HashMap<String, String>();
        for (String string : strs) {
            String key = string.split("=")[0].trim();
            String value = string.split("=")[1];
            map.put(key, value);
        }
        return map;
    }
}
然后现在将rabbitmq-provider 、rabbitmq-consumer两个项目跑起来，调用下/sendDirectMessage接口往直连型交换机推送一条消息，看看监听到的消息结果：



手动的确认模式的投递效率略低于自动，但是可以弥补自动确认模式的不足，更加准确地去记录消息消费情况。

那么如果需要有些消息接收类设置自动确认，有些消息接收类设置手动确认的话，那只需要将需要设置手动确认的相关队列加到之前的MessageListenerConfig的SimpleMessageListenerContainer里面就行。
PS： 我这个就是扇型交换机A、B、C里面仅仅对FanoutReceiverA设置了手动确认，推送一条消息，情况如下：


 