> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_33204709/article/details/133687613)

#### 目录

*   *   [一、背景](#_2)
    *   [二、Redis 的发布订阅](#Redis_12)
    *   *   [2.1 订阅单个频道](#21__16)
        *   *   [常用命令](#_27)
        *   [2.2 按规则（Pattern）订阅频道](#22_Pattern_43)
        *   [2.3 不推荐使用的原因](#23__81)
    *   [三、SpringBoot 实现发布订阅](#SpringBoot_92)
    *   *   [3.1 RedisUtil.java 发布类](#31_RedisUtiljava__94)
        *   *   [1）MessageDTO.java 实体类](#1MessageDTOjava__145)
            *   [2）发布测试](#2_179)
        *   [3.2 订阅实现方式一：实现 MessageListener 接口](#32_MessageListener_211)
        *   *   [1）RedisConfig.java 配置类](#1RedisConfigjava__213)
            *   [2）RedisMessageListener.java 监听类](#2RedisMessageListenerjava__277)
            *   [3）执行结果](#3_315)
        *   [3.2 订阅实现方式二：注入 MessageListenerAdapter 实例，反射调用](#32_MessageListenerAdapter_322)
        *   *   [1）RedisConfig.java 配置类](#1RedisConfigjava__324)
            *   [2）RedisMessageReceiver.java 监听类](#2RedisMessageReceiverjava__400)
            *   [3）执行结果](#3_432)

### 一、背景

[Redis](https://so.csdn.net/so/search?q=Redis&spm=1001.2101.3001.7020) 的 List 数据类型可以通过 `rpush` 和 `blpop` 命令实现[消息队列](https://so.csdn.net/so/search?q=%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97&spm=1001.2101.3001.7020)（先进后出），没有任何元素可以弹出的时候，连续会被阻塞。

但是**基于 List 实现的消息队列，不支持一对多的消息广播，相当于只有一个消费者。**

如果要实现一对多的消息广播，怎么办？

### 二、Redis 的发布订阅

> Redis 2.8 及以上版本实现了发布订阅的功能。

#### 2.1 订阅单个频道

首先思考一个问题：如果消息的生产者和消费者是不同的客户端，连接到同一个 Redis。通过什么对象把生产者和消费者关联起来呢？

在 **RabbitMQ** 里面叫 `Queue`，在 **Kafka** 里面叫 `Topic`，在 **Redis** 里面叫 `channel`（频道）。

**订阅者可以订阅一个或多个 channel。** 消息的发布者可以给指定的 channel 发布消息。只要消息到达了 channel，所有订阅了这个 channel 的订阅者都会收到这条消息。

![](https://img-blog.csdnimg.cn/6a25049f67cd43cfa126d7070990bd6e.png)

##### 常用命令

**订阅命令，一次可以订阅多个频道：**

**subscribe channel-1 channel-2**

**发布消息，一次只能在一个频道发布：**

**publish channel-1 hello**

**取消订阅（不能再订阅状态下使用）：**

**unsubscribe channel-1**

测试 -

#### 2.2 按规则（Pattern）订阅频道

支持 `?` 和 `*` 占位符：

*   `?`：代表一个字符。
*   `*`：代表 0 个或多个字符。

例如：现在有三个新闻频道:

*   运动新闻（news-sport）
*   音乐新闻（news-music）
*   天气新闻（news-weather）

![](https://img-blog.csdnimg.cn/a90507e913c249ae82fa4b9cdd27414d.png)

**消费端 1，订阅运动新闻：**

**psubscribe *sport**

**消费端 2，订阅所有新闻：**

**psubscribe news***

**消费端 3，订阅天气新闻：**

**psubscribe new-weather**

生产者，向三个频道分别发布三条消息，对应的订阅者能收到消息：

```
publish news-sport kobe
publish news-music jaychou
publish news-weather sunny

```

#### 2.3 不推荐使用的原因

*   **消息丢失：** Redis 的 Pub/Sub 模式不会对消息进行持久化，如果订阅者在消息发布之前未连接到 Redis 服务器，它们将无法接收到之前发布的消息。这意味着如果订阅者在消息发布之前断开连接或重新启动，它们将错过这些消息。
*   **内存占用：** 由于 Redis 将所有订阅者的订阅信息存储在内存中，当订阅者数量非常大时，可能会导致 Redis 服务器的内存占用过高。这会对 Redis 的性能和可伸缩性产生负面影响。
*   **阻塞问题：** 当订阅者在执行阻塞操作（例如阻塞式读取）时，它们将无法处理其他的 Redis 命令。这可能会导致性能问题，特别是在高并发环境中。
*   **无法保证消息传递顺序：** 在 Pub/Sub 模式中，消息的传递是异步的，并且无法保证消息的传递书匈奴。如果应用程序需要处理有序的消息，Pub/Sub 模式可能不适合。

一般来说，考虑到性能和持久化等因素，不建议使用 Redis 的发布订阅功能来实现 MQ。Redis 的一些内部机制用到了发布订阅功能。

### 三、SpringBoot 实现发布订阅

#### 3.1 RedisUtil.java 发布类

```
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import javax.annotation.Resource;
import java.util.Set;
import java.util.concurrent.TimeUnit;

/**
 * <p> @Title RedisUtil
 * <p> @Description Redis工具类
 *
 * @author ACGkaka
 * @date 2021/6/16 16:32
 */
@Slf4j
@Component
public class RedisUtil {

    @Qualifier("redisTemplate")
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * 向频道发布消息
     * @param channel   频道
     * @param message   消息
     * @return true成功 false失败
     */
    public boolean publish(String channel, Object message) {
        if (!StringUtils.hasText(channel)) {
            return false;
        }
        try {
            redisTemplate.convertAndSend(channel, message);
            log.info("发送消息成功，channel:{}, message:{}", channel, message);
            return true;
        } catch (Exception e) {
            log.error("发送消息失败，channel:{}, message:{}", channel, message, e);
        }
        return false;
    }

}

```

##### 1）MessageDTO.java 实体类

```
package com.demo.redis.listener;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MessageDTO implements Serializable {

    /**
     * 消息标题
     */
    private String title;

    /**
     * 消息内容
     */
    private String content;

    /**
     * 消息内容
     */
    private LocalDateTime createTime;
}

```

##### 2）发布测试

```
import com.demo.redis.listener.MessageDTO;
import com.demo.util.RedisUtil;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootRedisApplicationTests {

    @Autowired
    private RedisUtil redisUtil;

    @Test
    public void test1() {
        // 订阅主题
        final String TOPIC_NAME_1 = "TEST_TOPIC_1";
        final String TOPIC_NAME_2 = "TEST_TOPIC_2";
        // 发布消息
        MessageDTO dto = new MessageDTO("测试标题", "测试内容", LocalDateTime.now());
        redisUtil.publish(TOPIC_NAME_1, dto);
    }
}

```

#### 3.2 订阅实现方式一：实现 MessageListener 接口

##### 1）RedisConfig.java 配置类

```
import com.demo.redis.RedisCustomizeProperties;
import com.demo.redis.listener.RedisMessageListener;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.PatternTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.serializer.JdkSerializationRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.io.Serializable;

@Configuration
@EnableConfigurationProperties(RedisCustomizeProperties.class)
public class RedisConfig {

    /**
     * 配置RedisTemplate
     *
     * @param redisConnectionFactory 连接工厂
     * @return RedisTemplate
     */
    @Bean
    public RedisTemplate<String, Serializable> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<>();
        //设置key的存储方式为字符串
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //设置为value的存储方式为JDK二进制序列化方式，还有jackson序列化方式（Jackson2JsonRedisSerialize）
        redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
        //设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }

    /**
     * Redis消息监听器容器（实现方式一）
     *
     * @param redisConnectionFactory    连接工厂
     * @param listener                  消息监听器
     * @return Redis消息监听容器
     */
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory redisConnectionFactory,
                                                   RedisMessageListener listener) {
        // 订阅主题
        final String TOPIC_NAME_1 = "TEST_TOPIC_1";
        final String TOPIC_NAME_2 = "TEST_TOPIC_2";
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        // 设置连接工厂
        container.setConnectionFactory(redisConnectionFactory);
        // 订阅频道（可以添加多个）
        container.addMessageListener(listener, new PatternTopic(TOPIC_NAME_1));
        container.addMessageListener(listener, new PatternTopic((TOPIC_NAME_2)));

        return container;
    }
}

```

##### 2）RedisMessageListener.java 监听类

```
package com.demo.redis.listener;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.redis.connection.Message;
import org.springframework.data.redis.connection.MessageListener;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

@Slf4j
@Component
public class RedisMessageListener implements MessageListener {

    @Qualifier("redisTemplate")
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    @Override
    public void onMessage(Message message, byte[] pattern) {
        // 打印渠道
        log.info(">>>>>>>>>> 【INFO】订阅的channel：{}", new String(pattern));

        // 获取消息
        byte[] messageBody = message.getBody();
        // 序列化对象
        MessageDTO messageDTO = (MessageDTO) redisTemplate.getValueSerializer().deserialize(messageBody);

        // 打印消息
        log.info(">>>>>>>>>> 【INFO】收到的message：{}", messageDTO);
    }
}

```

##### 3）执行结果

执行 3.1 的发布测试，结果如下：

![](https://img-blog.csdnimg.cn/1232ac0bb2ac4796949a1a4283da0514.png)

#### 3.2 订阅实现方式二：注入 MessageListenerAdapter 实例，反射调用

##### 1）RedisConfig.java 配置类

```
import com.demo.redis.listener.RedisMessageReceiver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.listener.PatternTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;
import org.springframework.data.redis.serializer.JdkSerializationRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.io.Serializable;

@Configuration
public class RedisConfig {

    /**
     * 配置RedisTemplate
     *
     * @param redisConnectionFactory 连接工厂
     * @return RedisTemplate
     */
    @Bean
    public RedisTemplate<String, Serializable> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<>();
        //设置key的存储方式为字符串
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //设置为value的存储方式为JDK二进制序列化方式，还有jackson序列化方式（Jackson2JsonRedisSerialize）
        redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
        //设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }

    /**
     * Redis消息监听器容器（实现方式二）
     *
     * @param redisConnectionFactory    连接工厂
     * @param adapter                   消息监听器
     * @return Redis消息监听容器
     */
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory redisConnectionFactory,
                                                    MessageListenerAdapter adapter) {
        // 订阅主题
        final String TOPIC_NAME_1 = "TEST_TOPIC_1";
        final String TOPIC_NAME_2 = "TEST_TOPIC_2";
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        // 设置连接工厂
        container.setConnectionFactory(redisConnectionFactory);
        // 订阅频道（可以添加多个）
        container.addMessageListener(adapter, new PatternTopic(TOPIC_NAME_1));
        container.addMessageListener(adapter, new PatternTopic((TOPIC_NAME_2)));

        return container;
    }

    /**
     * 用于接收消息的消息接收器
     * @param receiver
     * @return
     */
    @Bean
    public MessageListenerAdapter listenerAdapter(RedisMessageReceiver receiver) {
        // receiveMessage 为反射调用，用于接收消息的方法名
        MessageListenerAdapter receiveMessage = new MessageListenerAdapter(receiver, "receiveMessage");
        receiveMessage.setSerializer(new JdkSerializationRedisSerializer());
        return receiveMessage;
    }

}

```

##### 2）RedisMessageReceiver.java 监听类

```
package com.demo.redis.listener;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

/**
 * <p> @Title RedisMessageReceiver
 * <p> @Description Redis消息接收器（实现方式二）
 *
 * @author ACGkaka
 * @date 2023/10/7 18:28
 */
@Slf4j
@Component
public class RedisMessageReceiver {

    /**
     * 接收消息（在 RedisConfig.java 中反射调用）
     */
    public void receiveMessage(MessageDTO messageDTO, String channel) {
        // 打印渠道
        log.info(">>>>>>>>>> 【INFO】订阅的channel：{}", channel);

        // 打印消息
        log.info(">>>>>>>>>> 【INFO】收到的message：{}", messageDTO);
    }
}

```

##### 3）执行结果

执行 3.1 的发布测试，结果如下：

![](https://img-blog.csdnimg.cn/8739f08f72b44133b25891dfa9d15b84.png)

整理完毕，完结撒花~ 🌻

参考地址：

1.Spring boot 整合 Redis 实现发布订阅（超详细），https://blog.csdn.net/BBQ__ZXB/article/details/124980860

2.springboot 中使用 redis 发布订阅，https://blog.csdn.net/H900302/article/details/113914979

3.Redis 的 Pub/Sub 为何不建议进行消息订阅，https://www.jianshu.com/p/3eff7425429a