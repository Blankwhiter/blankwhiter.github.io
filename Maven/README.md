# maven scope属性说明 以及传递性

# 一、scope属性

依赖范围控制哪些依赖在哪些classpath 中可用，哪些依赖包含在一个应用中。

![maven scope属性说明](images/image-202010121357001.png)


## compile （编译）
compile是默认的范围；如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath中可用，同时它们也会被打包。

## provided （已提供）
provided 依赖只有在当JDK 或者一个容器已提供该依赖之后才使用。例如， 如果你开发了一个web 应用，你可能在编译 classpath 中需要可用的Servlet API 来编译一个servlet，但是你不会想要在打包好的WAR 中包含这个Servlet API；这个Servlet API JAR 由你的应用服务器或者servlet 容器提供。已提供范围的依赖在编译classpath （不是运行时）可用。它们不是传递性的，也不会被打包。

## runtime （运行时）
runtime 依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现。

## test （测试）
test范围依赖 在一般的编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。

## system （系统）
system范围依赖与provided类似，但是你必须显式的提供一个对于本地系统中JAR文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构建应该是一直可用的，Maven 也不会在仓库中去寻找它。如果你将一个依赖范围设置成系统范围，你必须同时提供一个systemPath元素。注意该范围是不推荐使用的（建议尽量去从公共或定制的 Maven 仓库中引用依赖）。示例如下：

```
<project>
  <dependencies>
    <dependency>
      <groupId>sun.jdk</groupId>
      <artifactId>tools</artifactId>
      <version>1.5.0</version>
      <scope>system</scope>
      <systemPath>${java.home}/../lib/tools.jar</systemPath>
    </dependency>
    ...
  </dependencies>
</project>
```

## import(导入)
import仅支持在<dependencyManagement>中的类型依赖项上。它表示要在指定的POM <dependencyManagement>部分中用有效的依赖关系列表替换的依赖关系。该scope类型的依赖项实际上不会参与限制依赖项的可传递性。


给个scope类型对不同环境的情况：
![scope情况表格](images/image-202010121357002.png)



# 二、scope的依赖传递
A–>B–>C。当前项目为A，A依赖于B，B依赖于C，A与C的依赖关系？
当B对C的依赖的scope是test或者provided，则A不依赖C。
当B对C的依赖是scope是runtime或者compile，则A依赖C。且传递依赖的scope的规则：如果A对B的依赖是compile，那么A对C的依赖和B对C的依赖相同，否则和A对B的依赖保持一致。	

![scope的依赖传递](images/image-202010121357003.png)


# 三、scope为import的使用
前面说过该类型作用于只在dependencyManagement内使用生效，它可以用来管理模块依赖，说白了就是针对包含了一系列子依赖进的模块导入到当前项目中进行管理使用，而不是把需要用到的依赖一个一个的加入到项目中进行管理，可以理解为多继承模式。比如在一些场景中：我们只是想单纯加入springboot模块的依赖，而不想将springboot作为父模块引入项目中，此时就可以使用import来处理。

一般我们会将springboot作为父模块引入到项目中，如下：
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

一个项目一般只能有一个父依赖模块，真实开发中，我们都会自定义自己的父模块，这样就会冲突了。所以我们可以使用import来将springboot做为依赖模块导入自己项目中：
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.demo</groupId>
    <artifactId>MyService</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    <description>demo springboot</description>
    <inceptionYear>2019</inceptionYear>

    <licenses>
        <license>
            <name>The Apache Software License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <modules>
        <module>service</module>
        <module>common</module>
        <module>util</module>
    </modules>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- 注入组件定义的第三方依赖 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.9.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
            ......
        </dependencies>
    </dependencyManagement>

    <!-- 远程仓库配置 -->
    <distributionManagement> 
        <repository> 
            <id>releases</id> 
            <url>http://ali/nexus/content/repositories/releases</url> 
        </repository> 
        <snapshotRepository> 
            <id>snapshots</id> 
            <url>http://ali/nexus/content/repositories/snapshots/nexus/content/repositories/snapshots</url> 
        </snapshotRepository> 
    </distributionManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!--使用-Dloader.path需要在打包的时候增加<layout>ZIP</layout>，不指定的话-Dloader.path不生效-->
                    <layout>ZIP</layout>
                    <!-- 指定该jar包启动时的主类[建议] -->
                    <mainClass>com.common.util.CommonUtilsApplication</mainClass>
                    <!--<includes>-->
                        <!--<!–依赖jar不打进项目jar包中–>-->
                        <!--<include>-->
                            <!--<groupId>nothing</groupId>-->
                            <!--<artifactId>nothing</artifactId>-->
                        <!--</include>-->
                    <!--</includes>-->
                    <!--配置的 classifier 表示可执行 jar 的名字，配置了这个之后，在插件执行 repackage 命令时，
                    就不会给 mvn package 所打成的 jar 重命名了,这样就可以被其他项目引用了，classifier命名的为可执行jar-->
                    <!--<classifier>myexec</classifier>-->
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
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <!-- 剔除spring-boot打包的org和BOOT-INF文件夹(用于子模块打包) -->
                    <!--<skip>true</skip>-->
                    <source>1.8</source>
                    <target>1.8</target>
                    <!--<encoding>UTF-8</encoding>-->
                </configuration>
            </plugin>
            <!--拷贝依赖到jar外面的lib目录-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>
                                ${project.build.directory}/lib
                            </outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>${plugin.assembly.version}</version>
                <configuration>
                    <finalName>myservice</finalName>
                    <descriptor>deploy/assembly.xml</descriptor>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```


上述就可以将springboot模块作为依赖导入到项目中，然后就可以继承自己的父模块了，如果要加入其它类似springboot这样的模块的话就和加入springboot一样，这样就可以使模块管理看起来更简洁了，也实现了多继承的效果。



# 四 maven 自动处理版本冲突原则
## 1.同pom配置文件下 没有传递依赖
在同一个pom下 同类包会后面声明的会覆盖前面的依赖。(一般不存在这种情况)
情况1：
```
		<dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.9</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.0.1</version>
        </dependency>
```

![情况1](images/image-202010121357004.png)

情况2：

```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.0.1</version>
        </dependency>

		<dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.9</version>
        </dependency>
```
![情况2](images/image-202010121357005.png)


## 2.存在依赖传递的情况，声明优先原则

![声明优先原则](images/image-202010121357006.png)

 图片说明：同一组件的不同版本深度相同的话，就会按声明的顺序判定。图中R的依赖声明顺序是A→B→C，所以会选择A所引用的X 1.0作为依赖。如果反过来的话，就会选择X 3.0。


### 测试1：
在pom.xml文件中引入两个模块 AOP 和 Messaging 的jar包坐标信息，先声明的是AOP模块，当AOP和Messaging的依赖包发生冲突时，项目只会引入AOP模块底下的冲突依赖包，Messaging的冲突依赖包则会被排除。
```
   		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>

 		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-messaging</artifactId>
            <version>4.1.7.RELEASE</version>
        </dependency>
     
```
![测试1](images/image-202010121357007.png)


### 测试2：
将AOP和Messaging的声明信息调换一下位置：
```
 		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-messaging</artifactId>
            <version>4.1.7.RELEASE</version>
        </dependency>

   		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>
     
```
![测试2](images/image-202010121357008.png)


## 3.存在依赖传递情况下，路径最短原则

![路径最短原则](images/image-202010121357009.png)
图片说明：根组件R传递依赖了两个版本不同的X。由于R→C→X这条路径比R→A→B→X要短，也就是说X 1.0比X 2.0距离R更近，所以会选择X 1.0作为依赖，X 2.0会被忽略。

### 测试：
```
 		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-messaging</artifactId>
            <version>4.1.7.RELEASE</version>
        </dependency>

   		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>
     
```
![测试](images/image-202010121357010.png)
项目直接引入spring-aop（4.3.7版本），项目引入的spring-messaging架包的依赖 spring-context，spring-context又依赖于spring-aop（4.1.7版本),那么项目肯定会优先引入AOP的4.3.7版本，因为这个依赖路径最短。

# 附注：

### 1.在maven中经常会使用<optional>true</optional>参数，如下：
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
</dependency>
```
此处的<optional>true</optional>的作用是让依赖只被当前项目使用，而不会在模块间进行传递依赖。

### 2.dependencies 与 dependencyManagement 区别

dependencies 即使在子项目中不写该依赖项，那么子项目仍然会从父项目中继承该依赖项（全部继承）
dependencyManagement 里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且 version 和 scope 都读取自父 pom; 另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。