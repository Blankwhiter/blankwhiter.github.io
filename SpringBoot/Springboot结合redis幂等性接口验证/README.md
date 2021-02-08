# Springboot接口幂等性

幂等性：就是对于用户发起的同一操作的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生副作用。
你比如支付，在用户购买商品后的支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行二次付款。那么不论是扣款还是流水记录都生成了两条记录。或者提交特殊订单时网络问题生成两笔订单。在原来我们是把数据操作放入事务中即可，发生错误立即回滚，但是再相应客户端的时候是有可能网络再次中断或者异常等。

常见的解决方案
1、唯一索引：防止新增脏数据
2、token机制：防止页面重复提交
3、悲观锁： 获取数据的时候加锁（锁表或锁行）
4、乐观锁：基于版本号version实现，在更新数据的那一刻校验数据
5、分布式锁:redis(jedis、redisson)或zookeeper实现

 1.生成key值工具类
 ```
import java.lang.reflect.Method;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

import com.alibaba.fastjson.JSON;

/**
 * IdempotentKeyUtil
 * 生成key值工具类
 */
public class IdempotentKeyUtil {

    /**
     * 对接口的参数进行处理生成固定key
     * @param method
     * @param args
     * @return
     */
    public static String  generate(Method method,Object... args) {
        StringBuilder stringBuilder = new StringBuilder(method.toString());
        for (Object  arg : args) {
            stringBuilder.append(toStrinhg(arg));
        }
        //进行md5等长加密
        return md5(stringBuilder.toString());
    }   
    /**
     * 使用jsonObject对数据进行toString,(保持数据一致性)
     * @param object
     * @return
     */
    public static String toStrinhg(Object obj){
        if( obj == null ){
            return "-";
        }
        return JSON.toJSONString(obj);
    }

    /**
     * 对数据进行MD5等长加密
     * @param str
     * @return
     */
    public static String md5(String str){
        StringBuilder stringBuilder = new StringBuilder();
        try {
            //选择MD5作为加密方式
            MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(str.getBytes());
            byte b[] =  mDigest.digest();
            int j = 0;
            for (int i = 0,max = b.length; i < max; i++) {
                j = b[i];
                if(j < 0 ){
                    i += 256;
                }else if(j < 16){
                    stringBuilder.append(0);
                }
                stringBuilder.append(Integer.toHexString(j));
            }
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
		}
        return stringBuilder.toString();
    }
}
```

2.自定义注解
```
import java.lang.annotation.*;

/**
 * 接口幂等性注解
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Idempotent {
    /**
     *  注解自定义redis的key的一部分
     */
    String key();
    /**
     * 过期时间
     */
    long expirMillis();
}
```

3.AOP对我们自定义注解进行拦截处理
```
import io.lettuce.core.SetArgs;
import io.lettuce.core.api.async.RedisAsyncCommands;
import io.lettuce.core.cluster.api.async.RedisAdvancedClusterAsyncCommands;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.stereotype.Component;
import java.lang.reflect.Method;
import java.util.Objects;

/**
 * 自定义接口幂等性切点
 */
@Component
@Aspect
@ConditionalOnClass(RedisTemplate.class)
public class IdempotentAspect {
    private static final String KEY_TEMPLATE = "idempotent_%S";

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    /**
     * 切点(自定义注解)
     */
    @Pointcut("@annotation(com.example.springboot.test.annotation.Idempotent)")
    public void executeIdempotent() {

    }

    /**
     * 切点业务
     *
     * @throws Throwable
     */
    @Around("executeIdempotent()")
    public Object around(ProceedingJoinPoint jPoint) throws Throwable {
        //获取当前方法信息
        Method method = ((MethodSignature) jPoint.getSignature()).getMethod();
        //获取注解
        Idempotent idempotent = method.getAnnotation(Idempotent.class);
        //生成Key
        String key = String.format(KEY_TEMPLATE, idempotent.key() + "_" + IdempotentKeyUtil.generate(method, jPoint.getArgs()));
        //https://segmentfault.com/a/1190000002870317 -- JedisCommands接口的分析
        //nxxx的值只能取NX或者XX，如果取NX，则只有当key不存在是才进行set，如果取XX，则只有当key已经存在时才进行set
        //expx expx的值只能取EX或者PX，代表数据过期时间的单位，EX代表秒，PX代表毫秒
        // key value nxxx(set规则) expx(取值规则) time(过期时间)
        // String redisRes = redisTemplate.execute((RedisCallback<String>) conn -> ((JedisCommands) conn.getNativeConnection()).set(key, key, "NX", "EX", idempotent.expirMillis()));
        // Jedis jedis = new Jedis("127.0.0.1",6379);
        // jedis.auth("xuzz");
        // jedis.select(0);
        // String redisRes = jedis.set(key, key,"NX","EX",idempotent.expirMillis());
        long millis = idempotent.expirMillis();
        String execute = redisTemplate.execute(new RedisCallback<String>() {
            @Override
            public String doInRedis(RedisConnection connection) throws DataAccessException {
                Object nativeConnection = connection.getNativeConnection();
                String status = null;
                RedisSerializer<String> stringRedisSerializer = (RedisSerializer<String>) redisTemplate.getKeySerializer();

                byte[] keyByte = stringRedisSerializer.serialize(key);
                //springboot 2.0以上的spring-data-redis 包默认使用 lettuce连接包
                //lettuce连接包，集群模式，ex为秒，px为毫秒
                if (nativeConnection instanceof RedisAdvancedClusterAsyncCommands) {
                    status = ((RedisAdvancedClusterAsyncCommands) nativeConnection)
                            .getStatefulConnection().sync()
                            .set(keyByte, keyByte, SetArgs.Builder.px(millis).nx());
                }
                //lettuce连接包，单机模式，ex为秒，px为毫秒
                if (nativeConnection instanceof RedisAsyncCommands) {
                    status = ((RedisAsyncCommands) nativeConnection)
                            .getStatefulConnection().sync()
                            .set(keyByte, keyByte, SetArgs.Builder.px(millis).nx());
                }
                return status;
            }
        });

        if (Objects.equals("OK", execute)) {
            return jPoint.proceed();
        }else {
            //在此处可以跑出自定义异常
            System.err.println("数据错误");
            return null;
        }
    }
}
```

4.service 接口
```
/**
 * TestService
 */
public interface TestService {
    /**
     * 数据测试
     */
    public void print(String params);
}
```

5.TestServiceImpl
```
import org.springframework.stereotype.Service;

/**
 * TestServiceImpl
 */
@Service
public class TestServiceImpl implements TestService {
	@Idempotent(key = "com.example.springboot.test.service.TestService",expirMillis = 100)
	    @Override
	    public void print(String params) {
	         System.out.println(params);
	    }
	}
```

6.pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springboot-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-test</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.4</version>
        </dependency>
    </dependencies>

    <build>
        <finalName>boot-test</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>
</project>
```

7.application.yml
```
spring
  #redis集群
  redis:
    host: 47.99.200.71
    port: 6379
    timeout: 20000
    database: 4
```

8.IdempotentController 
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * IdempotentController
 */
@RestController
public class IdempotentController {
    @Autowired
    private TestService testService;

    /**
     * 测试
     */
    @PostMapping("/test")
    public String pushTest(String params){
        for (int i = 0; i < 5; i++) {
            new Thread(()->{
                try {
                    testService.print(params);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }
        return params;
    }
}
```