---
title: 填坑：spring-data-redis（Jedis）
date: 2019-08-05 20:46:00
tags: java
---


>问题：  Cloud not get resource from pool

![图片1:异常](https://t1.picb.cc/uploads/2019/08/05/gOvyMj.png)


>原因


1.使用的组件
pom.xml

```xml

     <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.data/spring-data-redis -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>1.8.3.RELEASE</version><!--$NO-MVN-MAN-VER$-->
        </dependency>

```

2.配置

开启事务配置,使用过程，方法上未使用@Transactional的注解。

```xml
//开启事务配置
 <property name="enableTransactionSupport" value="true"></property>

......
 <!--redis操作模版,使用该对象可以操作redis  -->  
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate" >    
        <property name="connectionFactory" ref="redisConnectionFactorySentinel" />    
        <!--如果不配置Serializer，那么存储的时候缺省使用String，如果用User类型存储，那么会提示错误User can't cast to String！！  -->    
        <property name="keySerializer" >    
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />    
        </property>    
        <property name="valueSerializer" >    
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />    
        </property>    
        <property name="hashKeySerializer">    
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>    
        </property>    
        <property name="hashValueSerializer">    
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>    
        </property>    
        <!--开启事务  -->  
        <property name="enableTransactionSupport" value="true"></property>  
    </bean>
   
     
     ......

```

3.分析代码

```java
//代码实现

@Component
public class RedisUtil {
	@Autowired
	private RedisTemplate<String, Object> redisTemplate;
	
  /** 
   * 普通缓存获取 
   * @param key 键 
   * @return 值 
   */  
  public Object get(String key){  
      return key==null?null:redisTemplate.opsForValue().get(key);  
  } 
  
......
//跟踪源码

org.springframework.data.redis.core.RedisTemplate

 public <T> T execute(RedisCallback<T> action, boolean exposeConnection, boolean pipeline) {
        Assert.isTrue(this.initialized, "template not initialized; call afterPropertiesSet() before using it");
        Assert.notNull(action, "Callback object must not be null");
        RedisConnectionFactory factory = this.getConnectionFactory();
        RedisConnection conn = null;

        Object var11;
        try {
            if (this.enableTransactionSupport) {// 开启事务，此值为：true
                conn = RedisConnectionUtils.bindConnection(factory, this.enableTransactionSupport);
            } else {
                conn = RedisConnectionUtils.getConnection(factory);
            }

            boolean existingConnection = TransactionSynchronizationManager.hasResource(factory);
            RedisConnection connToUse = this.preProcessConnection(conn, existingConnection);
            boolean pipelineStatus = connToUse.isPipelined();
            if (pipeline && !pipelineStatus) {
                connToUse.openPipeline();
            }

            RedisConnection connToExpose = exposeConnection ? connToUse : this.createRedisConnectionProxy(connToUse);
            T result = action.doInRedis(connToExpose);
            if (pipeline && !pipelineStatus) {
                connToUse.closePipeline();
            }

            var11 = this.postProcessResult(result, connToUse, existingConnection);
        } finally {
            RedisConnectionUtils.releaseConnection(conn, factory);//调用释放连接的方法
        }

        return var11;
    }
    
.....

//跟踪释放连接的代码 

//事务开启 connHolder 不为null,get方法没有注解@Transactional，connHolder.isTransactionSyncronisationActive()方法返回false
//事务开启 isConnectionTransactional(conn, factory)为true，没有配置 @Transactional(readOnly = true)，TransactionSynchronizationManager.isCurrentTransactionReadOnly()为false
//经过层层判断，未进入任何一个if方法块中，也没调用任何回收redis连接的方法
  public static void releaseConnection(RedisConnection conn, RedisConnectionFactory factory) {
        if (conn != null) {
            RedisConnectionUtils.RedisConnectionHolder connHolder = (RedisConnectionUtils.RedisConnectionHolder)TransactionSynchronizationManager.getResource(factory);//事务开启 connHolder 不为null
            if (connHolder != null && connHolder.isTransactionSyncronisationActive()) {//条件不够，不执行
                if (log.isDebugEnabled()) {
                    log.debug("Redis Connection will be closed when transaction finished.");
                }

            } else {
                if (isConnectionTransactional(conn, factory) && TransactionSynchronizationManager.isCurrentTransactionReadOnly()) {//条件不够，不执行
                    unbindConnection(factory);
                } else if (!isConnectionTransactional(conn, factory)) {//条件不够，不执行
                    if (log.isDebugEnabled()) {
                        log.debug("Closing Redis Connection");
                    }

                    conn.close();
                }

            }
        }
    }



```


> 综上所述，当spring-data-redis 配置事务的时候，方法使用中不加注解@Transactional，会出现redis连接无法回收，资源池pool 资源耗尽，报异常：Cloud not get resource from pool



> 解决方法：声明两个RedisTemplate实例

两个RedisTemplate实例? 
- 支持事务：commands要么统一执行，要么都被清除，维护数据完整性； 
- 不支持事务，command立即执行，即时返回执行结果并且更高效；

```java
/** Sample Configuration **/
@Configuration
public class RedisTxContextConfiguration {
  @Bean
  public StringRedisTemplate redisTransactionTemplate() {
    StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory());
    // explicitly enable transaction support
    template.setEnableTransactionSupport(true);
    return template;
  }
 @Bean
  public StringRedisTemplate redisTemplate() {
    StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory());
    return template;
  }
}
```








