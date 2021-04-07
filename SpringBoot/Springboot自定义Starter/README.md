# Springboot自定义starter

# 一、创建一个maven项目
项目包名版本
```
    <groupId>com.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>1.0</version>
```

Maven依赖如下：
```
 	  <dependencies>
        <!--封装Starter核心依赖  -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
        <!--非必需,该依赖作用是在使用IDEA编写配置文件有代码提示-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <version>2.1.3.RELEASE</version>
        </dependency>
        <!-- lombok 插件  -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.6</version>
            <optional>true</optional>
        </dependency>
        <!-- 日志框架 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.11.0</version>
        </dependency>

```    
spring-boot-configuration-processor不是必须的，它的作用是和编译时生成 spring-configuration-metadata.json，此文件主要给IDEA使用。
配置此JAR相关配置属性在 application.yml中，你可以用Ctrl+鼠标左键点击属性名，IDE会跳转到你配置此属性的类中，并且编写application.yml会有代码提示。

# 二、编写项目基础类
```
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
@Data
@ConfigurationProperties(prefix = "test")
public class TestProperties {
    private String url;
    private String weight;
}

```


```
@AllArgsConstructor
@Log4j2
public class TestService {
    private String url;
    private String weight;

    public String show(){
        if(this.url==null){
            log.error("url not config");
            return "";
        }
        if(this.weight==null){
            log.error("weight not config");
            return "";
        }
        return this.url+" - "+this.weight;
    }

}

```

# 三.编写Starter自动配置类
```
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import spring.boot.starter.test.prop.TestProperties;
import spring.boot.starter.test.service.TestService;

import javax.annotation.Resource;

@Configuration //标注为配置类 源码中可以看到 标注这个注解的类会被加载
@ConditionalOnClass(TestService.class) // 该注解意思是 有“”TestService“”类才起作用 通俗来说就是你不注该服务类，就不会去注入下面的类
@EnableConfigurationProperties(TestProperties.class) //开启配置文件
public class TestConfig {

    @Resource
    private TestProperties testProperties; //注入配置文件

    @Bean
    @ConditionalOnMissingBean  //该注解的意思是如果没有发现service的bean就执行新建一个bean
    public TestService testService(){
        return new TestService(testProperties.getUrl(),testProperties.getWeight());
    }
}

```

# 四.创建spring.factories文件
spring.factories该文件用来定义需要自动配置的类，SpringBoot启动时会进行对象的实例化，会通过加载类SpringFactoriesLoader加载该配置文件，将文件中的配置类加载到spring容器
在src/main/resources新建META-INF文件夹，在META-INF文件夹下新建spring.factories文件。配置内容如下:
```
#-------starter自动装配---------
org.springframework.boot.autoconfigure.EnableAutoConfiguration=spring.boot.starter.test.config.TestConfig
```

# 五.打包和测试
使用Maven插件,将项目打包安装到本地仓库(mvn clean install  功能)
在springboot项目下依赖
```
        <dependency>
            <groupId>com.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>1.0</version>
        </dependency>
```
配置application.yml
```
test:
  url: http://baidu.com
  weight: 9
```
使用包
```
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import spring.boot.starter.test.service.TestService;

import javax.annotation.Resource;

@RestController
@RequestMapping("/index")
public class IndexController {
    @Resource
    private TestService testService;

    @RequestMapping("/test")
    public String test() {
        System.out.println("this is test" + testService.show());
        return System.currentTimeMillis() + "";
    }

}

```