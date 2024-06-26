---
title: "rabbitmq工作原理"
date: 2024-04-23T20:46:01+08:00
draft: false
categories: ["rabbitmq"]
tags: ["rabbitmq"]
keywords: ["rabbitmq"]
---


# rabbitmq工作原理

## AMQP协议
    AMQP是一个提供统一消息服务的应用层标准协议，基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同开发语言等条件的限制。

    AMQP协议是一种二进制协议，提供客户端应用与消息中间件之间异步、安全、高效地交互。

    AMQP作为中间层服务，把消息生产和消费分割出来，当消息生产者出现异常，不影响消费者对消息的消费，当消费者异常时，生产者生产的消息可以存放到服务的内存或者磁盘，不以影响到消息的消费，同时，消息也可以基于路由的规则可以投递到指定的消费者消费。

    AMQP基于模块化通过Exchange和Message Queue两个组件组合实现消息路由分发

## 生产者和消费者
    生产者是生产消息的主体，消费者是消费消息的主体

    数据集成与系统解耦、异步处理与事件驱动、流量削峰、事务消息与分布式事务的最终一致等

    生产者生产一条消息丢给消息代理，消息代理根据投递规则将消息传送到消费者手上

## 交换机
    交换机就像是消息代理的路由器，负责拿到一个消息之后，根据确定的规则（路由键）将他路由给一个或零个队列、交换机具备多种路由模式。

    基于消息生产者和路由规则可以将消息投递到指定的Message Queue，交换机收到生产者投递的消息，基于路由规则及队列绑定关系匹配到投递对应的交换机或者队列进行分发，交换机不存储消息，只做转发

    直连交换机：根据路由键完全匹配的投递到对应的队列。

    扇形交换机：无视路由键，将消息进行拷贝，并路由到给绑定到它身上的所有队列，提供了一个广播的效果

    主题交换机： 根据路由键按模式匹配的投递到对应的队列

    交换机也具备自己的属性，可以定义自己的名字，是否持久化等选项

## 队列
    消息的暂存地，至少有一个消费者订阅了队列的话，消息会立即发送给这些订阅的消费者。但是如果消息到达了无人的订阅队列，消息会在队列中等待，等待有了消费者便进行分发

    Exchange和Message Queue之间存在绑定关系，消息到了Exchange后基于路由策略可以将消息投递到已绑定且符合路由策略的Message Queue。

    消息队列会将消息存储到内存或者磁盘中，并将这些消息按照一定顺序转发给一个或多个消费者，每个消息队列都是独立隔离的，相互不影响。

    消息队列具有不同的属性：私有，共享，持久化，临时，客户端定义或者服务端定义等，可以基于实际需求选择对应的类型

## 消息
    消息是信息的载体，也是MAQP协议的一个实体，消息包含以下两部分
    载荷：就是真正的信息，是你想要传输的任何内容，该部分内容对消息代理来说是透明的
    元消息：包含路由键、内容类型、编码、是否持久化等等消息属性，会被消息代理所解析，消息代理根据消息的属性对这条消息进行投递，存储等。这部分被消息代理所关心，而消费者对其是不关心的

## 信道
    网络信道，是建立在Connection连接之上的一种轻量级的连接。几乎所有的操作都在Chanel中进行，Channel是进行消息读写的通道，客户端可以建立对各Channel，每个Channel代表一个会话任务

    如果把Connection比作一条光纤电缆的话，那么Channel信道就比作成光纤电缆中的其中一束光纤。一个Connection上可以创建任意数量的Channel。

    大部分的业务操作是在Channel这个接口中完成的

    队列声明 queueDeclare
    交换机的声明 ExchangeDeclare
    队列的绑定 queueBind
    发布消息 basicPublish
    消费消息 basicConsume


## Rabbitmq 工作模式
一、简单模式

    生产者发送消息到队列，消费者进行消费，没有交换机

二、工作模式
    一个生产者发送消息到队列中，队列分配消息给不同的队列，队列接收到消息进行消费

三、订阅模式
    每个队列的消息都是一样的，生产者把消息发送给交换机，交换机把消息发送给消息绑定的队列，让消费者消费

四、路由模式
    根据路由键发送到不同的消息队列中

五、主题模式
    根据路由键分类，发送到不同的消息队列中