# 本地缓存-Ehcache3

> 集成到springboot

* pom.xml

```xml
<!--开启 cache 缓存 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```

* application.properties

```properties
# 不是ehcache
spring.cache.type=jcache
```

* 配置类EhcahceConfig

```java
package com.example.sjs.ehcache;

import org.ehcache.CacheManager;
import org.ehcache.config.builders.CacheConfigurationBuilder;
import org.ehcache.config.builders.CacheManagerBuilder;
import org.ehcache.config.builders.ExpiryPolicyBuilder;
import org.ehcache.config.builders.ResourcePoolsBuilder;
import org.ehcache.config.units.MemoryUnit;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.io.File;

/**
 * @auther lanmei
 * @date 2022/12/28
 */
@Configuration
@EnableCaching
public class EhcacheConfig {

    @Bean
    public CacheManager getCacheManager() {
        CacheManager build = CacheManagerBuilder.newCacheManagerBuilder()
                // 持久化路径
                .with(CacheManagerBuilder.persistence(new File("tempfiles/cache/")))
                // disk的persistent=true表明持久化
                .withCache("noExpiry", CacheConfigurationBuilder.newCacheConfigurationBuilder(String.class,
                                String.class, ResourcePoolsBuilder
                                        .heap(100).offheap(10, MemoryUnit.MB).disk(500, MemoryUnit.MB, true))
                        // 永不过期
                        .withExpiry(ExpiryPolicyBuilder.noExpiration())
                        .build())
                .build(true);
        return build;
    }
}
```

* 测试类

```java
@Test
void getCache() {
    Cache<String, String> noExpiry = cacheManager.getCache("noExpiry", String.class, String.class);
    noExpiry.put("lanmei", "永不过期================================================！");
    String noExpriy = noExpiry.get("lanmei");
    System.out.println(noExpriy);
    noExpiry.clear();
    String noExpriy2 = noExpiry.get("lanmei");
    System.out.println(noExpriy2);
}
```

官网示例

```java
@Test
void ehcahe() {
    CacheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder()
        .withCache("preConfigured",
                   CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
                                                                          ResourcePoolsBuilder.heap(100))
                   .build())
        .build(true);

    Cache<Long, String> preConfigured
        = cacheManager.getCache("preConfigured", Long.class, String.class);

    Cache<Long, String> myCache = cacheManager.createCache("myCache",
                                                           CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class,
                                                                                                                  ResourcePoolsBuilder.heap(100))
                                                           .withExpiry(ExpiryPolicyBuilder.noExpiration())
                                                           .build());

    myCache.put(1L, "da one!");
    String value = myCache.get(1L);
    System.out.println(value);

    cacheManager.close();

}

```



> 层组合

如果需要使用多个层，则需要遵循一定的规则。规则：

**必须有heap层**
disk和clusterd不能同时存在
层的大小应该采用金字塔的方式，即金字塔上的层比下的层使用更少的内存

根据规则进行以下有效配置：

* heap + offheap

* heap + offheap + disk

* heap + offheap + clustered

* heap + disk

* heap + clustered

  对于多层，put、get的顺序：

当将一个值放入缓存时，它直接最低层。比如heap + offheap + disk，直接会存储在disk层。
当获取一个值，从最高层获取，如果没有继续向下一层获取，一旦获取到，会向上层推送，同时上层存储该值。

> 淘汰策略（3种）

* LFU（least frequently used）：最不频繁使用。以次数维度统计

* LRU（least recently used）：最近最少使用（默认）。以时间纬度统计

* FIFO（first in first out）：先进先出。数据结构上使用Queue来实现

> 过期策略（3种） 

* no expiry：永不过期
* time-to-live：创建后一段时间后过期
* time-to-idle：访问后一段时间过期

Java示例

```java
		CacheConfiguration<String, String> configuration = CacheConfigurationBuilder
				.newCacheConfigurationBuilder(String.class, String.class, ResourcePoolsBuilder.heap(10))
				.withExpiry(ExpiryPolicyBuilder.timeToIdleExpiration(Duration.ofSeconds(300))).build();
```

支持实现ExpiryPolicy接口来自定义过期策略。

 除了heap层，可以存储对象；其他所有层都只能通过二进制形式表示，所以需要对对象进行序列化和反序列化 

# 异步@Async

可以处理日志

注解`@EnableAsync`

异步方法所在类`@Component`。也可以使用 @Controller、@RestController、@Service、@Configuration等注解，加入到Ioc容器里。 

异步方法上加上`@Async`注解

```java
@Slf4j
@Component
@EnableAsync
public class AsyncFactory {

    @Async
    public void test() {
        log.info("异步输出");
    }
}
```

```java
@Autowired
private AsyncFactory asyncFactory;

@Test
void testAsync() throws InterruptedException {
    asyncFactory.test();
    System.out.println("你好==============");
}
```

> @Async注解失效情况

* 异步方法使用static修饰

* 调用方法和异步方法在一个类

> Future回调

 如果我想异步执行，同时想获取所有异步执行的结果，那么这个时候就需要采用Future。 

异步方法

```java
@Component
@Slf4j
public class FutureTask {
    
    @Async
    public Future<String> taskOne() throws Exception {
        //执行内容同上，省略
        return new AsyncResult<>("1完成");
    }
    @Async
    public Future<String> taskTwo() throws Exception {
        //执行内容同上，省略
        return new AsyncResult<>("2完成");
    }

    @Async
    public Future<String> taskThere() throws Exception {
        //执行内容同上，省略
        return new AsyncResult<>("执行任务3完成");
    }
}
```

调用方法

```java
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
@EnableAsync
public class FutureTaskTest {

    @Autowired
    private FutureTask futureTask;

    @Test
    public void runAsync() throws Exception {
        long start = System.currentTimeMillis();
        Future<String> taskOne = futureTask.taskOne();
        Future<String> taskTwo = futureTask.taskTwo();
        Future<String> taskThere = futureTask.taskThere();

        while (true) {
            if (taskOne.isDone() && taskTwo.isDone() && taskThere.isDone()) {
                log.info("任务1返回结果={},任务2返回结果={},任务3返回结果={},", taskOne.get(), taskTwo.get(), taskThere.get());
                break;
            }
        }
        long end = System.currentTimeMillis();
        log.info("总任务执行结束,总耗时={} 毫秒", end - start);
    }
}
```

> @Async+自定义线程实现异步任务

如果不自定义异步方法的线程池默认使用SimpleAsyncTaskExecutor线程池。

SimpleAsyncTaskExecutor：不是真的线程池，这个类不重用线程，每次调用都会创建一个新的线程。并发大的时候会产生严重的性能问题。‘

Spring也更加推荐我们开发者使用ThreadPoolTaskExecutor类来创建线程池。

阿里规范要求不能使用`Executors`

```bash
线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors返回的线程池对象的弊端如下：
1）FixedThreadPool和SingleThreadPool:
  允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。
2）CachedThreadPool:
  允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。
```

```java
Positive example 1：
    //org.apache.commons.lang3.concurrent.BasicThreadFactory
    ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
       
Positive example 2：
    ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
        .setNameFormat("demo-pool-%d").build();

    //Common Thread Pool
    ExecutorService pool = new ThreadPoolExecutor(5, 200,
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

    pool.execute(()-> System.out.println(Thread.currentThread().getName()));
    pool.shutdown();//gracefully shutdown

Positive example 3：
    <bean id="userThreadPool"
        class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        <property name="corePoolSize" value="10" />
        <property name="maxPoolSize" value="100" />
        <property name="queueCapacity" value="2000" />

    <property name="threadFactory" value= threadFactory />
        <property name="rejectedExecutionHandler">
            <ref local="rejectedExecutionHandler" />
        </property>
    </bean>
    //in code
    userThreadPool.execute(thread);
```



自定义线程池

spring方式

```java
@Configuration
public class ExecutorAsyncConfig {
    
    @Bean(name = "newAsyncExecutor")
    public Executor newAsync() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //设置核心线程数
        taskExecutor.setCorePoolSize(10);
        // 线程池维护线程的最大数量，只有在缓冲队列满了以后才会申请超过核心线程数的线程
        taskExecutor.setMaxPoolSize(100);
        //缓存队列
        taskExecutor.setQueueCapacity(50);
        //允许的空闲时间，当超过了核心线程数之外的线程在空闲时间到达之后会被销毁
        taskExecutor.setKeepAliveSeconds(200);
        //异步方法内部线程名称
        taskExecutor.setThreadNamePrefix("my-xiaoxiao-AsyncExecutor-");
        //拒绝策略
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        taskExecutor.initialize();
        return taskExecutor;
    }
}
```

示例代码

```java
@Component
@Slf4j
public class FutureExecutorTask {

    @Async("newAsyncExecutor")
    public Future<String> taskOne() throws Exception {
        log.info("任务1线程名称 = {}", Thread.currentThread().getName());
        return new AsyncResult<>("1完成");
    }
    @Async("newAsyncExecutor")
    public Future<String> taskTwo() throws Exception {
        log.info("任务2线程名称 = {}", Thread.currentThread().getName());
        return new AsyncResult<>("2完成");
    }

    @Async
    public Future<String> taskThere() throws Exception {
        log.info("任务3线程名称 = {}", Thread.currentThread().getName());
        return new AsyncResult<>("执行任务3完成");
    }
}
```

> CompletableFuture实现异步任务

推荐这种方式来实现异步,它不需要在启动类上加@EnableAsync注解，也不需要在方法上加@Async注解,它实现更加优雅，而且CompletableFuture功能更加强大。

示例

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest
public class CompletableTest {

    @Autowired
    private FutureExecutorTask dmoTask;
    
    @Test
    public void testCompletableThenRunAsync() throws Exception {
        long startTime = System.currentTimeMillis();

        CompletableFuture<Void> cp1 = CompletableFuture.runAsync(() -> {
            //任务1
            dmoTask.taskOne();
        });
        CompletableFuture<Void> cp2 = CompletableFuture.runAsync(() -> {
            //任务2
            dmoTask.taskTwo();
        });
        CompletableFuture<Void> cp3 = CompletableFuture.runAsync(() -> {
            //任务3
            dmoTask.taskThere();
        });
        
        cp1.get();
        cp2.get();
        cp3.get();
        //模拟主程序耗时时间
        System.out.println("总共用时" + (System.currentTimeMillis() - startTime) + "ms");
    }
}
```



# 打包跳过test

```xml
<properties>
	<java.version>8</java.version>
	<skipTests>true</skipTests>
</properties>
```

# jar包瘦身

 jar包同级目录下lib

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <mainClass>com.lanmei.SjsApplication</mainClass>
            <layout>ZIP</layout>
            <includes>
                <!--排除lib包，nothing任何依赖项都不进行打包-->
                <include>
                    <groupId>nothing</groupId>
                    <artifactId>nothing</artifactId>
                </include>
            </includes>
            <excludes>
                <exclude>
                    <groupId>org.projectlombok</groupId>
                    <artifactId>lombok</artifactId>
                </exclude>
            </excludes>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>repackage</goal>
                </goals>
            </execution>
        </executions>
    </plugin>

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
            <execution>
                <id>copy-dependencies</id>
                <phase>package</phase>
                <goals>
                    <goal>copy-dependencies</goal>
                </goals>
                <configuration>
                    <type>jar</type>
                    <includeTypes>jar</includeTypes>
                    <includeScope>runtime</includeScope>
                    <!--指定拷贝依赖项存放的目录位置-->
                    <outputDirectory>${project.build.directory}/lib</outputDirectory>
                    <overWriteReleases>false</overWriteReleases>
                    <overWriteSnapshots>false</overWriteSnapshots>
                    <overWriteIfNewer>true</overWriteIfNewer>
                </configuration>
            </execution>
        </executions>
    </plugin>

    <!--指定加载外部jar位置-->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
            <archive>
                <manifest>
                    <addClasspath>true</addClasspath>
                    <!--外部jar位置，这里表示从同级目录的lib包下加载-->
                    <classpathPrefix>./lib</classpathPrefix>
                    <mainClass>com.lanmei.SjsApplication</mainClass>
                </manifest>
            </archive>
        </configuration>
    </plugin>

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
            <!--打包跳过test-->
            <skip>true</skip>
        </configuration>
    </plugin>
</plugins>
```



# shardingsphere-jdbc

官网中文地址：https://shardingsphere.apache.org/document/current/cn/overview

官网示例为mysql，根据官网配置，基本上没有问题。

查看官网适配了哪写数据库，可查看`BranchDatabaseType`接口被哪写类实现，目前官网支持如下数据库

<img src="../../doc/static/images/SpringBoot/1693655148822.png" alt="1693655148822" style="zoom:50%;" />

* Oracle。虽然被适配，但是有些坑；分片配置中，例如，yaml文件中，数据库和表名要大写

## DM8示例

本例以`达梦8数据库`为例，展示没有被适配的数据库如何使用，使用yaml配置

springboot2.2.13、mybatis-plus5.3.2、dm8

* 引入maven依赖

  ```xml
  <dependency>
      <groupId>org.apache.shardingsphere</groupId>
      <artifactId>shardingsphere-jdbc-core</artifactId>
      <version>5.3.2</version>
  </dependency>
  
   <!-- yaml解析，没有这个会报错-->
  <dependency>
      <groupId>org.yaml</groupId>
      <artifactId>snakeyaml</artifactId>
      <version>1.33</version>
  </dependency>
  
  ```

  注意：缺少`snakeyaml`，会出现如下错误

  <img src="../../doc/static/images/SpringBoot/1693655071686.png" alt="1693655071686" style="zoom:50%;" />

### 配置文件

  application.yml

  ```yaml
  spring:
    main:
      #允许定义重名的bean对象覆盖原有的bean（datasource覆盖）
      allow-bean-definition-overriding: true
  
    datasource:
      driver-class-name: org.apache.shardingsphere.driver.ShardingSphereDriver
      url: jdbc:shardingsphere:classpath:config-sharding.yaml
  ```

  config-sharding.yaml

  ```yaml
  
  
  dataSources:
    ds2023:
      dataSourceClassName: com.zaxxer.hikari.HikariDataSource
      driverClassName: dm.jdbc.driver.DmDriver
      username: CETC15_FXBZ_CZ_IOT
      password: CETC15_FXBZ_CZ_IOT
      jdbcUrl: jdbc:dm://127.0.0.1:5236?schema=CETC15_FXBZ_CZ_IOT_2023
    ds2024:
      dataSourceClassName: com.zaxxer.hikari.HikariDataSource
      driverClassName: dm.jdbc.driver.DmDriver
      username: CETC15_FXBZ_CZ_IOT
      password: CETC15_FXBZ_CZ_IOT
      jdbcUrl: jdbc:dm://127.0.0.1:5236?schema=CETC15_FXBZ_CZ_IOT_2024
    ds2025:
      dataSourceClassName: com.zaxxer.hikari.HikariDataSource
      driverClassName: dm.jdbc.driver.DmDriver
      username: CETC15_FXBZ_CZ_IOT
      password: CETC15_FXBZ_CZ_IOT
      jdbcUrl: jdbc:dm://127.0.0.1:5236?schema=CETC15_FXBZ_CZ_IOT_2025
  rules:
    #  - !SINGLE
    #    tables:
    #      - "*.*"
    - !SHARDING
      tables:
        ORDER_INFO:
          actualDataNodes: ds$->{2023..2025}.ORDER_INFO_$->{1..12}
          databaseStrategy:
            standard:
              shardingAlgorithmName: db-sharding-algorithm
              shardingColumn: CREATE_TIME
          tableStrategy:
            standard:
              shardingAlgorithmName: tb-sharding-algorithm
              shardingColumn: CREATE_TIME
      shardingAlgorithms:
        db-sharding-algorithm:
          props:
            strategy: standard
            algorithmClassName: com.caso.shardingjdbc.properities.DbShardingAlgorithm
          type: CLASS_BASED
        tb-sharding-algorithm:
          props:
            strategy: standard
            algorithmClassName: com.caso.shardingjdbc.properities.TableShardingAlgorithmByMonth
          type: CLASS_BASED
  props:
    sql-show: true
  
  ```

### 分片规则，按日期分片，重写

  * 数据库，按年

    ```java
    package com.caso.shardingjdbc.properities;
    
    import com.caso.shardingjdbc.utils.DateTimeFormatUtil;
    import com.google.common.collect.Range;
    import lombok.extern.slf4j.Slf4j;
    import org.apache.shardingsphere.sharding.api.sharding.standard.PreciseShardingValue;
    import org.apache.shardingsphere.sharding.api.sharding.standard.RangeShardingValue;
    import org.apache.shardingsphere.sharding.api.sharding.standard.StandardShardingAlgorithm;
    import org.springframework.stereotype.Component;
    
    import java.time.LocalDate;
    import java.time.format.DateTimeFormatter;
    import java.util.*;
    
    /**
     * 数据库分片算法
     *
     * @author wangzk
     */
    @Slf4j
    @Component(value = "dbShardingAlgorithm")
    public class DbShardingAlgorithm implements StandardShardingAlgorithm<String> {
        /**
         * 分片算法的属性配置
         */
        private final Properties properties = new Properties();
    
        /**
         * 分片键参数格式化
         */
        private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    
        /**
         * 根据精确分片值进行分片操作，返回分片后的目标库名
         *
         * @param collection           所有的目标库名集合
         * @param preciseShardingValue 精确分片值对象，包含了逻辑库名、分片键和分片值
         * @return 分片后的目标库名
         */
        @Override
        public String doSharding(final Collection collection, final PreciseShardingValue preciseShardingValue) {
            String timeStr = preciseShardingValue.getValue().toString();
            String formatterStr = DateTimeFormatUtil.getFormatterStr(timeStr, true);
            int year = LocalDate.parse(formatterStr, FORMATTER).getYear();
            String target = "";
            for (Object o : collection) {
                if (o.toString().contains(String.valueOf(year))) {
                    target = o.toString();
                }
            }
            return target;
        }
    
        /**
         * 根据范围分片值进行分片操作，返回分片后的目标库名集合
         *
         * @param collection         所有的目标库名集合
         * @param rangeShardingValue 范围分片值对象，包含了逻辑库名、分片键和分片范围
         * @return 分片后的目标库名集合
         */
        @Override
        public Collection<String> doSharding(final Collection<String> collection,
                                             final RangeShardingValue<String> rangeShardingValue) {
            Range<String> valueRange = rangeShardingValue.getValueRange();
            LocalDate startDay = LocalDate.parse(valueRange.lowerEndpoint(), FORMATTER);
            LocalDate endDay = LocalDate.parse(valueRange.upperEndpoint(), FORMATTER);
    
            Set<String> res = new HashSet<>();
            while (!startDay.isAfter(endDay)) {
                int year = startDay.getYear();
                for (Object o : collection) {
                    if (o.toString().contains(String.valueOf(year))) {
                        res.add(o.toString());
                    }
                }
                // 获取增加后的日期
                startDay = startDay.plusYears(1);
            }
    
            return res;
        }
    
        /**
         * 获取分片算法的类型
         *
         * @return 分片算法的类型
         */
        @Override
        public String getType() {
            return "DB_SHARDING_BY_YEAR";
        }
    
        /**
         * 获取分片算法的属性配置
         *
         * @return 分片算法的属性配置
         */
    
    
        /**
         * 初始化分片算法和属性配置
         *
         * @param properties 分片算法的属性配置
         */
        @Override
        public void init(Properties properties) {
            log.info("初始化按年分库算法 -> properties:{}", properties.toString());
        }
    }
    
    ```

  * 表，按月

    ```java
    package com.caso.shardingjdbc.properities;
    
    import com.caso.shardingjdbc.utils.DateTimeFormatUtil;
    import com.google.common.collect.Range;
    import lombok.extern.slf4j.Slf4j;
    import org.apache.shardingsphere.sharding.api.sharding.standard.PreciseShardingValue;
    import org.apache.shardingsphere.sharding.api.sharding.standard.RangeShardingValue;
    import org.apache.shardingsphere.sharding.api.sharding.standard.StandardShardingAlgorithm;
    import org.springframework.stereotype.Component;
    
    import java.time.LocalDate;
    import java.time.format.DateTimeFormatter;
    import java.util.ArrayList;
    import java.util.Collection;
    import java.util.List;
    import java.util.Properties;
    
    /**
     * @auther lanmei
     * @date 2022/8/24
     */
    @Slf4j
    @Component(value = "tableShardingAlgorithmByMonth")
    public class TableShardingAlgorithmByMonth implements StandardShardingAlgorithm<String> {
    
        /**
         * 分片键参数格式化
         */
        private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    
        /**
         * 分片算法的属性配置
         */
        private final Properties properties = new Properties();
    
    
        @Override
        public String doSharding(final Collection collection, final PreciseShardingValue preciseShardingValue) {
            String timeStr = preciseShardingValue.getValue().toString();
            String formatterStr = DateTimeFormatUtil.getFormatterStr(timeStr, true);
            LocalDate date = LocalDate.parse(formatterStr, FORMATTER);
            int monthValue = date.getMonthValue();
            return preciseShardingValue.getLogicTableName().toUpperCase() + "_" + monthValue;
        }
    
        @Override
        public Collection<String> doSharding(Collection<String> collection, RangeShardingValue<String> rangeShardingValue) {
            String tableName = rangeShardingValue.getLogicTableName().toUpperCase();
            Range<String> valueRange = rangeShardingValue.getValueRange();
    
            LocalDate startDate = LocalDate.parse(valueRange.lowerEndpoint(), FORMATTER);
            LocalDate endDate = LocalDate.parse(valueRange.upperEndpoint(), FORMATTER);
    
            List<String> resultList = new ArrayList<>();
            while (!startDate.isAfter(endDate)) {
                int quarter = startDate.getMonthValue();
                resultList.add(tableName + "_" + quarter);
                startDate = startDate.plusDays(1);
            }
    
            return resultList;
        }
    
    
    
        @Override
        public void init(Properties properties) {
            log.info("初始化按月分表算法 -> properties:{}", properties.toString());
        }
    }
    
    ```

### 新增达梦SPI

  官方提供SPI机制适配其他数据库。所以使用SPI适配达梦。2个类`DMDatabaseType`和`DMDataSourceMetaData`，在resources\META-INF\services目录下新建org.apache.shardingsphere.infra.database.type.DatabaseType文件，没有任何文件后缀，内容填写`com.caso.shardingjdbc.databasetype.DMDatabaseType`即，`DMDatabaseType`的全路径。（如何确定文件的名称，找到`MySQLDatabaseType`类，利用idea，找到引用处，可以看到org.apache.shardingsphere.infra.database.type.DatabaseType文件，即新建文件名）

  * DMDatabaseType

    ```java
    package com.caso.shardingjdbc.databasetype;
    
    import org.apache.shardingsphere.infra.database.type.BranchDatabaseType;
    import org.apache.shardingsphere.infra.database.type.DatabaseType;
    import org.apache.shardingsphere.infra.util.spi.type.typed.TypedSPILoader;
    import org.apache.shardingsphere.sql.parser.sql.common.enums.QuoteCharacter;
    
    import java.util.Collection;
    import java.util.Collections;
    import java.util.Map;
    
    /**
     *
     * 目前是简单实现,使用了MySQL的处理逻辑,扩展BranchDatabaseType,实现达梦数据库的DMDatabaseType.
     * 完善功能是实现org.apache.shardingsphere.spi.database.DatabaseType接口
     * DatabaseTypes的静态方法,通过SPI加载,需要在 META-INF/services/org.apache.shardingsphere.spi.database.DatabaseType中配置全路径
     *
     * @author springrain
     */
    public final class DMDatabaseType implements BranchDatabaseType {
    
    
        @Override
        public QuoteCharacter getQuoteCharacter() {
            return QuoteCharacter.QUOTE;
        }
    
        @Override
        public Collection<String> getJdbcUrlPrefixes() {
            return Collections.singleton(String.format("jdbc:%s:", getType().toLowerCase()));
        }
    
        @Override
        public DMDataSourceMetaData getDataSourceMetaData(final String url, final String username) {
            return new DMDataSourceMetaData(url);
        }
    
        @Override
        public DatabaseType getTrunkDatabaseType() {
            return TypedSPILoader.getService(DatabaseType.class, "MySQL");
        }
    
        @Override
        public Map<String, Collection<String>> getSystemDatabaseSchemaMap() {
            return Collections.emptyMap();
        }
    
        @Override
        public Collection<String> getSystemSchemas() {
            return Collections.emptyList();
        }
    
        @Override
        public String getType() {
            return "DM";
        }
    }
    
    ```

  * DMDataSourceMetaData

    ```java
    package com.caso.shardingjdbc.databasetype;
    
    import com.google.common.base.Strings;
    import lombok.Generated;
    import lombok.Getter;
    import org.apache.shardingsphere.infra.database.metadata.DataSourceMetaData;
    
    import java.util.Properties;
    import java.util.regex.Matcher;
    import java.util.regex.Pattern;
    
    /**
     * @auther lanmei
     * @date 2023/8/26
     */
    @Getter
    public class DMDataSourceMetaData implements DataSourceMetaData {
    
        private static final int DEFAULT_PORT = 5236;
        private static final Pattern THIN_URL_PATTERN = Pattern.compile("jdbc:dm://([\\w\\-\\.]+):?([0-9]*)\\?schema=([\\w\\-\\.]+)", Pattern.CASE_INSENSITIVE);
        private final Pattern pattern = Pattern.compile("jdbc:dm://([\\w\\-\\.]+):?([0-9]*)\\?schema=([\\w\\-\\.]+)", Pattern.CASE_INSENSITIVE);
        private final String hostname;
        private final int port;
        private final String catalog;
        private final String schema;
    
        public DMDataSourceMetaData(String url) {
            Matcher matcher = THIN_URL_PATTERN.matcher(url);
    
            if (matcher.find()) {
                this.hostname = matcher.group(1);
                this.port = Strings.isNullOrEmpty(matcher.group(2)) ? DEFAULT_PORT : Integer.valueOf(matcher.group(2));
                this.catalog = matcher.group(3);
    
                this.schema = null;
            } else {
                System.out.println("No match found.");
                this.hostname = null;
                this.port = 0;
                this.catalog = null;
                this.schema = null;
    
            }
    
    
    
    
        }
    
        public Properties getQueryProperties() {
            return new Properties();
        }
    
        public Properties getDefaultQueryProperties() {
            return new Properties();
        }
    
        @Generated
        public String getHostname() {
            return this.hostname;
        }
    
        @Generated
        public int getPort() {
            return this.port;
        }
    
        @Generated
        public String getCatalog() {
            return this.catalog;
        }
    
        @Generated
        public String getSchema() {
            return this.schema;
        }
    }
    
    ```

  * org.apache.shardingsphere.infra.database.type.DatabaseType

    ```java
    com.caso.shardingjdbc.databasetype.DMDatabaseType
    ```

## 注意事项

分片使用限制：https://shardingsphere.apache.org/document/current/cn/features/sharding/limitation/







# MQTT

> 测试环境搭建https://blog.csdn.net/u010632165/article/details/132145926
>
> * mosquitto
> * MQTTX

> Qos分析https://baijiahao.baidu.com/s?id=1758337893245255496&wfr=spider&for=pc

 MQTT定义了三个QoS等级，分别为： 

* QoS 0，最多交付一次。

* QoS 1，至少交付一次。

* QoS 2，只交付一次。

  使用QoS 0可能丢失消息，使用QoS 1可以保证收到消息，但消息可能重复，使用QoS 2可以保证消息既不丢失也不重复。QoS等级从低到高，不仅意味着消息可靠性的提升，也意味着传输复杂程度的提升。





> 重新连接的一些坑https://www.cnblogs.com/shihuc/p/11552406.html

*  cleanSession =true；通过 connectonLost 重新连接，也无法继续订阅 （我的paho是1.2.0版本，EMQX：V3.1.1） 
*  cleanSession =true &&  autoconnect =false；通过 connectonLost 重新连接，可以连接，可以重新订阅，但是cpu占用很高
* cleanSession =false &&  autoconnect =false；可以连接成功，并且可以重新订阅，cpu正常