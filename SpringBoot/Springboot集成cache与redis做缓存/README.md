# Springboot redis cache

1.pom.xml集成环境
```
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
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.freemarker</groupId>
            <artifactId>freemarker</artifactId>
            <version>2.3.28</version>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

    </dependencies>

```

2.application.yml 配置
```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://47.99.200.71:3306/test?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowPublicKeyRetrieval=true
    username: root
    password: 123456
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    serialization:
      write-dates-as-timestamps: false

  #redis集群
  redis:
    host: 47.99.200.71
    port: 6379
    timeout: 20000
    database: 4
  cache:
    type: redis


mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    auto-mapping-behavior: full
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath*:mapper/**/*Mapper.xml

```

3.主类加入注解
```
@SpringBootApplication
@MapperScan("com.example.springboot.test.mapper")
@EnableCaching

```

4.redis配置 RedisConfig
```
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

/**
 * redis配置
 */
@Configuration
public class RedisConfig {
    /**
     * 缓存存活时间
     */
    private Duration timeToLive = Duration.ofMinutes(30);
    public void setTimeToLive(Duration timeToLive) {
        this.timeToLive = timeToLive;
    }

    /**
     * cache设置
     * @param factory
     * @return
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        //解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        // 配置序列化（解决乱码的问题）
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(timeToLive)
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }

    /**
     *  RedisTemplate配置
     */
    @Bean
    @SuppressWarnings("all")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(factory);
        //配置事务
        template.setEnableTransactionSupport(true);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

}

```


5.MybatisPlusConfig
```
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 配置分页插件
 *
 */
@Configuration
public class MybatisPlusConfig {

    /**
     * 分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}

```

6.User
```
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.extension.activerecord.Model;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.Accessors;

/**
 * <p>
 * 
 * </p>
 *
 * @author belongHuang
 * @since 2021-02-02
 */
@Data
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
public class User extends Model {

    private static final long serialVersionUID = 1L;

    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    private String name;

    private String email;

}

```
7.UserMapper

```
import com.example.springboot.test.model.User;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;

import java.util.List;

/**
 * <p>
 *  Mapper 接口
 * </p>
 *
 * @author belongHuang
 * @since 2021-02-02
 */
public interface UserMapper extends BaseMapper<User> {

}

```
8.IUserService
```
import com.example.springboot.test.model.User;
import com.baomidou.mybatisplus.extension.service.IService;

import java.util.List;

/**
 * <p>
 * 服务类
 * </p>
 *
 * @author belongHuang
 * @since 2021-02-02
 */
public interface IUserService extends IService<User> {
    List<User> findAllUser();

    User getOneUser(int id);

    int add(User user);

    int update(User user);

    int delete(int id);
}

```
9.UserServiceImpl
```
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.example.springboot.test.model.User;
import com.example.springboot.test.mapper.UserMapper;
import com.example.springboot.test.service.IUserService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.cache.annotation.Caching;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;

/**
 * <p>
 * 服务实现类
 * </p>
 *
 * @author belongHuang
 * @since 2021-02-02
 */
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {
    @Resource
    private UserMapper userMapper;

    @Cacheable(value = "user:list")
    @Override
    public List<User> findAllUser() {
        return userMapper.selectList(new QueryWrapper<>());
    }

    @Cacheable(value = "user:id", key = "#id")
    @Override
    public User getOneUser(int id) {
        return baseMapper.selectById(id);
    }


    @CacheEvict(value = "user:list", allEntries = true)
    @Override
    public int add(User user) {
        return userMapper.insert(user);
    }

    @Caching(
            evict = {
                    @CacheEvict(value = "user:id", key = "#user.id"),
                    @CacheEvict(value = "user:list", allEntries = true)
            }
    )
    @Override
    public int update(User user) {
        return baseMapper.updateById(user);
    }


    @Caching(
            evict = {
                    @CacheEvict(value = "user:id", key = "#id"),
                    @CacheEvict(value = "user:list", allEntries = true)
            }
    )
    @Override
    public int delete(int id) {
        return delete(id);
    }

}

```
10.UserController
```
import com.example.springboot.test.model.User;
import com.example.springboot.test.service.IUserService;
import com.example.springboot.test.service.impl.UserServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.List;

/**
 * <p>
 *  前端控制器
 * </p>
 *
 * @author belongHuang
 * @since 2021-02-02
 */
@RestController
@RequestMapping("/user")
public class UserController {

    @Resource
    private IUserService userService;

    @RequestMapping("/list")
    public List<User> list(){
        return userService.findAllUser();
    }

    @RequestMapping("/getUser")
    public User getUser(Integer id){
        return userService.getOneUser(id);
    }

    @RequestMapping("/updateUser")
    public int updateUser(User user){
        return userService.update(user);
    }

}

```

附录：

# 声明式缓存注解
Spring提供4个注解来声明缓存规则，如下表所示：

<table>
<thead>
<tr>
<th style="text-align:left">注解</th>
<th style="text-align:left">说明</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left">@Cacheable</td>
<td style="text-align:left">方法执行前先看缓存中是否有数据，如果有直接返回。如果没有就调用方法，并将方法返回值放入缓存</td>
</tr>
<tr>
<td style="text-align:left">@CachePut</td>
<td style="text-align:left">无论怎样都会执行方法，并将方法返回值放入缓存</td>
</tr>
<tr>
<td style="text-align:left">@CacheEvict</td>
<td style="text-align:left">将数据从缓存中删除</td>
</tr>
<tr>
<td style="text-align:left">@Caching</td>
<td style="text-align:left">可通过此注解组合多个注解策略在一个方法上面</td>
</tr>
</tbody>
</table>

@Cacheable 、@CachePut 、@CacheEvict都有value属性，指定要使用的缓存名称，而key属性指定缓存中存储的键。

@EnableCaching 开启缓存。

# @Cacheable
这个注解含义是方法结果会被放入缓存，并且一旦缓存后，下一次调用此方法，会通过key去查找缓存是否存在，如果存在就直接取缓存值，不再执行方法。

这个注解有几个参数值，定义如下
<table>
<thead>
<tr>
<th style="text-align:left">参数</th>
<th style="text-align:left">解释</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left">cacheNames</td>
<td style="text-align:left">缓存名称</td>
</tr>
<tr>
<td style="text-align:left">value</td>
<td style="text-align:left">缓存名称的别名</td>
</tr>
<tr>
<td style="text-align:left">condition</td>
<td style="text-align:left">Spring SpEL 表达式，用来确定是否缓存</td>
</tr>
<tr>
<td style="text-align:left">key</td>
<td style="text-align:left">SpEL 表达式，用来动态计算key</td>
</tr>
<tr>
<td style="text-align:left">keyGenerator</td>
<td style="text-align:left">Bean 名字，用来自定义key生成算法，跟key不能同时用</td>
</tr>
<tr>
<td style="text-align:left">unless</td>
<td style="text-align:left">SpEL 表达式，用来否决缓存，作用跟condition相反</td>
</tr>
<tr>
<td style="text-align:left">sync</td>
<td style="text-align:left">多线程同时访问时候进行同步</td>
</tr>
</tbody>
</table>
在计算key、condition或者unless的值得时候，可以使用到以下的特有的SpEL表达式
<table>
<thead>
<tr>
<th style="text-align:left">表达式</th>
<th style="text-align:left">解释</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left">#result</td>
<td style="text-align:left">表示方法的返回结果</td>
</tr>
<tr>
<td style="text-align:left">#root.method</td>
<td style="text-align:left">当前方法</td>
</tr>
<tr>
<td style="text-align:left">#root.target</td>
<td style="text-align:left">目标对象</td>
</tr>
<tr>
<td style="text-align:left">#root.caches</td>
<td style="text-align:left">被影响到的缓存列表</td>
</tr>
<tr>
<td style="text-align:left">#root.methodName</td>
<td style="text-align:left">方法名称简称</td>
</tr>
<tr>
<td style="text-align:left">#root.targetClass</td>
<td style="text-align:left">目标类</td>
</tr>
<tr>
<td style="text-align:left">#root.args[x]</td>
<td style="text-align:left">方法的第x个参数</td>
</tr>
</tbody>
</table>

# @CachePut
该注解在执行完方法后会触发一次缓存put操作，参数跟@Cacheable一致

# @CacheEvict
该注解在执行完方法后会触发一次缓存evict操作，参数除了@Cacheable里的外，还有个特殊的allEntries， 表示将清空缓存中所有的值。

 