# SecurityManager应用场景
　　
>当运行未知的Java程序的时候，该程序可能有恶意代码（删除系统文件、重启系统等），为了防止运行恶意代码对系统产生影响，需要对运行的代码的权限进行控制，这时候就要启用Java安全管理器。

# 一、默认配置文件
　　默认的安全管理器配置文件是 `$JAVA_HOME/jre/lib/security/java.policy`，即当未指定配置文件时，将会使用该配置。内容如下：
```
// Standard extensions get all permissions by default

grant codeBase "file:${{java.ext.dirs}}/*" {
        permission java.security.AllPermission;
};

// default permissions granted to all domains

grant {
        // Allows any thread to stop itself using the java.lang.Thread.stop()
        // method that takes no argument.
        // Note that this permission is granted by default only to remain
        // backwards compatible.
        // It is strongly recommended that you either remove this permission
        // from this policy file or further restrict it to code sources
        // that you specify, because Thread.stop() is potentially unsafe.
        // See the API specification of java.lang.Thread.stop() for more
        // information.
        permission java.lang.RuntimePermission "stopThread";

        // allows anyone to listen on dynamic ports
        permission java.net.SocketPermission "localhost:0", "listen";

        // "standard" properies that can be read by anyone

        permission java.util.PropertyPermission "java.version", "read";
        permission java.util.PropertyPermission "java.vendor", "read";
        permission java.util.PropertyPermission "java.vendor.url", "read";
        permission java.util.PropertyPermission "java.class.version", "read";
        permission java.util.PropertyPermission "os.name", "read";
        permission java.util.PropertyPermission "os.version", "read";
        permission java.util.PropertyPermission "os.arch", "read";
        permission java.util.PropertyPermission "file.separator", "read";
        permission java.util.PropertyPermission "path.separator", "read";
        permission java.util.PropertyPermission "line.separator", "read";

        permission java.util.PropertyPermission "java.specification.version", "read";
        permission java.util.PropertyPermission "java.specification.vendor", "read";
        permission java.util.PropertyPermission "java.specification.name", "read";

        permission java.util.PropertyPermission "java.vm.specification.version", "read";
        permission java.util.PropertyPermission "java.vm.specification.vendor", "read";
        permission java.util.PropertyPermission "java.vm.specification.name", "read";
        permission java.util.PropertyPermission "java.vm.version", "read";
        permission java.util.PropertyPermission "java.vm.vendor", "read";
        permission java.util.PropertyPermission "java.vm.name", "read";
};

```



权限(对应java类)分为以下类别：
- 文件 (java.io.FilePermission)
- 套接字 (java.net.SocketPermission) 
- 网络 (java.net.NetPermission)
- 安全性 (java.security.SecurityPermission)
- 运行时 (java.lang.RuntimePermission)
- 属性 (java.util.PropertyPermission)
- AWT  (java.awt.AWTPermission)
- 反射 (java.lang.reflect.ReflectPermission)
- 可序列化(java.io.SerializablePermission)

除前两个（FilePermission 和 SocketPermission）类以外的所有类都是`java.security.BasicPermission` 的子类，而 `java.security.BasicPermission` 类又是顶级权限类`java.security.Permission` 的抽象子类。`BasicPermission` 定义了所有权限所需的功能，这些功能的名称遵从分层属性命名惯例（例如“exitVM”、“setFactory”、“queuePrintJob”等等）。在名称的末尾可能出现一个星号，前面是“.”或星号，这表示通配符匹配。例如：“a.*”、“*”是有效的，而“*a”或“a*b”是无效的。

`FilePermission` 和 `SocketPermission` 是顶级权限类 (java.security.Permission) 的子类。像这些命名语法比 `BasicPermission` 所用的语法更为复杂的类都直接是 `Permission` 的子类，而不是`BasicPermission` 的子类。例如，对于 `java.io.FilePermission` 对象而言，权限名就是文件（或目录）的路径名。

某些权限类具有一个“动作”列表，告知允许对象所执行的动作。例如，对于`java.io.FilePermission` 对象，动作列表（如“读、写”）指定了允许对指定文件（或指定目录中的文件）执行哪些动作。

其他权限类是“指定的”权限 - 有名称但没有动作列表的类；您也许有指定的权限，也许没有。

注：还有一个暗指所有权限的 `java.security.AllPermission` 权限。该权限是为了简化系统管理员的工作而存在的，因为管理员可能需要执行很多需要所有（或许多）权限的任务。





# 二、启动关闭安全管理器
启动安全管理有两种方式，建议使用启动参数方式。

## 2.1 启动参数方式
　　启动程序的时候通过附加参数启动安全管理器：
```
-Djava.security.manager
```
　　若要同时指定配置文件的位置那么示例如下：
```
-Djava.security.manager -Djava.security.policy="E:/java.policy"
```

## 2.2 编码方式启动
　　也可以通过编码方式启动，不过不建议：
```
System.setSecurityManager(new SecurityManager());
```
　　通过参数启动，本质上也是通过编码启动，不过参数启动使用灵活，项目启动源码如下（sun.misc.Launcher）：
```
// Finally, install a security manager if requested
String s = System.getProperty("java.security.manager");
if (s != null) {
    SecurityManager sm = null;
    if ("".equals(s) || "default".equals(s)) {
        sm = new java.lang.SecurityManager();
    } else {
        try {
            sm = (SecurityManager)loader.loadClass(s).newInstance();
        } catch (IllegalAccessException e) {
        } catch (InstantiationException e) {
        } catch (ClassNotFoundException e) {
        } catch (ClassCastException e) {
        }
    }
    if (sm != null) {
        System.setSecurityManager(sm);
    } else {
        throw new InternalError(
            "Could not create SecurityManager: " + s);
    }
}
```
　可以发现将会创建一个默认的SecurityManager；

## 2.3 关闭安全管理器
```
SecurityManager sm=System.getSecurityManager();
if(sm!=null){
    System.setSecurityManager(null);
}
```


## 2.4 自定义安全管理器
自定义类继承SecurityManager
定义一个文件读取检测 （需要用密码检测）：

```
import java.io.BufferedReader;
import java.io.FileDescriptor;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * 文件读取检测 （需要用密码检测）
 */
public class PasswordSecurityManager extends SecurityManager {
    private String password;

    public PasswordSecurityManager(String password) {
        super();
        this.password = password;
    }
    private boolean accessOK() {
        int c;
        BufferedReader dis = new BufferedReader(new InputStreamReader(System.in));
        String response;
        System.out.println("What's the secret password?");
        try {
            response = dis.readLine();
            if (response.equals(password))
                return true;
            else
                return false;
        } catch (IOException e) {
            return false;
        }
    }
    @Override
    public void checkRead(FileDescriptor filedescriptor) {
        if (!accessOK())
            throw new SecurityException("Not a Chance!");
    }
    @Override
    public void checkRead(String filename) {
        if (!accessOK())
            throw new SecurityException("No Way!");
    }
    @Override
    public void checkRead(String filename, Object executionContext) {
        if (!accessOK())
            throw new SecurityException("Forget It!");
    }

}
```


```
        System.setSecurityManager(new PasswordSecurityManager("123"));
        System.out.println("开始检测");
        FileReader fileReader = new FileReader("d://log.txt");
        System.out.println("检测通过");
```

# 三、问题解决
　当出现关于安全管理的报错的时候，基本有两种方式来解决。

## 3.1 取消安全管理器
　　一般情况下都是无意启动安全管理器，所以这时候只需要把安全管理器进行关闭，去掉启动参数即可。

## 3.2 增加相应权限
　　若因为没有权限报错，则报错信息中会有请求的权限和请求什么权限，如下：
```
Exception in thread "main" java.security.AccessControlException: access denied (java.io.FilePermission E:\pack\a\a.txt write)
```
　　上面例子，请求资源`E:\pack\a\a.txt`，的`FilePermission`的写权限没有，因此被拒绝。

　　也可以开放所有权限：
```
grant { 
    permission java.security.AllPermission;
};
```




# 附录
## 1.配置文件说明
1.1 配置基本原则
　　在启用安全管理器的时候，配置遵循以下基本原则：
	- 没有配置的权限表示没有。
	- 只能配置有什么权限，不能配置禁止做什么。
	- 同一种权限可多次配置，取并集。
	- 统一资源的多种权限可用逗号分割。
1.2 默认配置文件解释
　第一部分授权：
```
grant codeBase "file:${{java.ext.dirs}}/*" {
    permission java.security.AllPermission;
};
```
　　授权基于路径在"file:${{java.ext.dirs}}/*"的class和jar包，所有权限。

　　第二部分授权：
```
grant { 
    permission java.lang.RuntimePermission "stopThread";
    ……   
}
```
　　这是细粒度的授权，对某些资源的操作进行授权。具体不再解释，可以查看javadoc。如RuntimePermission的可授权操作经查看javadoc如下：
<table summary="permission target name, 
  what the target allows,and associated risks" border="1" cellpadding="5">
<tbody>
<tr><th>权限目标名称</th><th>权限所允许的操作</th><th>允许此权限所带来的风险</th></tr>
<tr>
<td>createClassLoader</td>
<td>创建类加载器</td>
<td>授予该权限极其危险。能够实例化自己的类加载器的恶意应用程序可能会在系统中装载自己的恶意类。这些新加载的类可能被类加载器置于任意保护域中，从而自动将该域的权限授予这些类。</td>
</tr>
<tr>
<td>getClassLoader</td>
<td>类加载器的获取（即调用类的类加载器）</td>
<td>这将授予攻击者得到具体类的加载器的权限。这很危险，由于攻击者能够访问类的类加载器，所以攻击者能够加载其他可用于该类加载器的类。通常攻击者不具备这些类的访问权限。</td>
</tr>
<tr>
<td>setContextClassLoader</td>
<td>线程使用的上下文类加载器的设置</td>
<td>在需要查找可能不存在于系统类加载器中的资源时，系统代码和扩展部分会使用上下文类加载器。授予 setContextClassLoader 权限将允许代码改变特定线程（包括系统线程）使用的上下文类加载器。</td>
</tr>
<tr>
<td>enableContextClassLoaderOverride</td>
<td>线程上下文类加载器方法的子类实现</td>
<td>在需要查找可能不存在于系统类加载器中的资源时，系统代码和扩展部分会使用上下文类加载器。授予 enableContextClassLoaderOverride 权限将允许线程的子类重写某些方法，这些方法用于得到或设置特定线程的上下文类加载器。</td>
</tr>
<tr>
<td>setSecurityManager</td>
<td>设置安全管理器（可能会替换现有的）</td>
<td>安全管理器是允许应用程序实现安全策略的类。授予 setSecurityManager 权限将通过安装一个不同的、可能限制更少的安全管理器，来允许代码改变所用的安全管理器，因此可跳过原有安全管理器所强制执行的某些检查。</td>
</tr>
<tr>
<td>createSecurityManager</td>
<td>创建新的安全管理器</td>
<td>授予代码对受保护的、敏感方法的访问权，可能会泄露有关其他类或执行堆栈的信息。</td>
</tr>
<tr>
<td>getenv.{variable name}</td>
<td>读取指定环境变量的值</td>
<td>此权限允许代码读取特定环境变量的值或确定它是否存在。如果该变量含有机密数据，则这项授权是很危险的。</td>
</tr>
<tr>
<td>exitVM.{exit status}</td>
<td>暂停带有指定退出状态的 Java 虚拟机</td>
<td>此权限允许攻击者通过自动强制暂停虚拟机来发起一次拒绝服务攻击。注意：自动为那些从应用程序类路径加载的全部代码授予 "exitVM.*" 权限，从而使这些应用程序能够自行中止。此外，"exitVM" 权限等于 "exitVM.*"。</td>
</tr>
<tr>
<td>shutdownHooks</td>
<td>虚拟机关闭钩子 (hook) 的注册与取消</td>
<td>此权限允许攻击者注册一个妨碍虚拟机正常关闭的恶意关闭钩子 (hook)。</td>
</tr>
<tr>
<td>setFactory</td>
<td>设置由 ServerSocket 或 Socket 使用的套接字工厂，或 URL 使用的流处理程序工厂</td>
<td>此权限允许代码设置套接字、服务器套接字、流处理程序或 RMI 套接字工厂的实际实现。攻击者可能设置错误的实现，从而破坏数据流。</td>
</tr>
<tr>
<td>setIO</td>
<td>System.out、System.in 和 System.err 的设置</td>
<td>此权限允许改变标准系统流的值。攻击者可以改变 System.in 来监视和窃取用户输入，或将 System.err 设置为 "null" OutputStream，从而隐藏发送到 System.err 的所有错误信息。</td>
</tr>
<tr>
<td>modifyThread</td>
<td>修改线程，例如通过调用线程的&nbsp;<tt>interrupt</tt>、<tt>stop</tt>、<tt>suspend</tt>、<tt>resume</tt>、<tt>setDaemon</tt>、<tt>setPriority</tt>、<tt>setName</tt>&nbsp;和&nbsp;<tt>setUncaughtExceptionHandler</tt>&nbsp;方法</td>
<td>此权限允许攻击者修改系统中任意线程的行为。</td>
</tr>
<tr>
<td>stopThread</td>
<td>通过调用线程的&nbsp;<code>stop</code>&nbsp;方法停止线程</td>
<td>如果系统已授予代码访问该线程的权限，则此权限允许代码停止系统中的任何线程。此权限会造成一定的危险，因为该代码可能通过中止现有的线程来破坏系统。</td>
</tr>
<tr>
<td>modifyThreadGroup</td>
<td>修改线程组，例如通过调用 ThreadGroup 的&nbsp;<code>destroy</code>、<code>getParent</code>、<code>resume</code>、<code>setDaemon</code>、<code>setMaxPriority</code>、<code>stop</code>&nbsp;和&nbsp;<code>suspend</code>&nbsp;方法</td>
<td>此权限允许攻击者创建线程组并设置它们的运行优先级。</td>
</tr>
<tr>
<td>getProtectionDomain</td>
<td>获取类的 ProtectionDomain</td>
<td>此权限允许代码获得特定代码源的安全策略信息。虽然获得安全策略信息并不足以危及系统安全，但这确实会给攻击者提供了能够更好地定位攻击目标的其他信息，例如本地文件名称等。</td>
</tr>
<tr>
<td>getFileSystemAttributes</td>
<td>获取文件系统属性</td>
<td>此权限允许代码获得文件系统信息（如调用者可用的磁盘使用量或磁盘空间）。这存在潜在危险，因为它泄露了关于系统硬件配置的信息以及一些关于调用者写入文件特权的信息。</td>
</tr>
<tr>
<td>readFileDescriptor</td>
<td>读取文件描述符</td>
<td>此权限允许代码读取与文件描述符读取相关的特定文件。如果该文件包含机密数据，则此操作非常危险。</td>
</tr>
<tr>
<td>writeFileDescriptor</td>
<td>写入文件描述符</td>
<td>此权限允许代码写入与描述符相关的特定文件。此权限很危险，因为它可能允许恶意代码传播病毒，或者至少也会填满整个磁盘。</td>
</tr>
<tr>
<td>loadLibrary.{库名}</td>
<td>动态链接指定的库</td>
<td>允许 applet 具有加载本机代码库的权限是危险的，因为 Java 安全架构并未设计成可以防止恶意行为，并且也无法在本机代码的级别上防止恶意行为。</td>
</tr>
<tr>
<td>accessClassInPackage.{包名}</td>
<td>当类加载器调用 SecurityManager 的<code>checkPackageAccess</code>方法时，通过类加载器的&nbsp;<code>loadClass</code>&nbsp;方法访问指定的包</td>
<td>此权限允许代码访问它们通常无法访问的那些包中的类。恶意代码可能利用这些类帮助它们实现破坏系统安全的企图。</td>
</tr>
<tr>
<td>defineClassInPackage.{包名}</td>
<td>当类加载器调用 SecurityManager 的&nbsp;<code>checkPackageDefinition</code>&nbsp;方法时，通过类加载器的&nbsp;<code>defineClass</code>&nbsp;方法定义指定的包中的类。</td>
<td>此权限允许代码在特定包中定义类。这样做很危险，因为具有此权限的恶意代码可能在受信任的包中定义恶意类，比如&nbsp;<code>java.security</code>或&nbsp;<code>java.lang</code>。</td>
</tr>
<tr>
<td>accessDeclaredMembers</td>
<td>访问类的已声明成员</td>
<td>此权限允许代码查询类的公共、受保护、默认（包）访问和私有的字段和/或方法。尽管代码可以访问私有和受保护字段和方法名称，但它不能访问私有/受保护字段数据并且不能调用任何私有方法。此外，恶意代码可能使用该信息来更好地定位攻击目标。而且，它可以调用类中的任意公共方法和/或访问公共字段。如果代码不能用这些方法和字段将对象强制转换为类/接口，那么它通常无法调用这些方法和/或访问该字段，而这可能很危险。</td>
</tr>
<tr>
<td>queuePrintJob</td>
<td>打印作业请求的开始</td>
<td>这可能向打印机输出敏感信息，或者只是浪费纸张。</td>
</tr>
<tr>
<td>getStackTrace</td>
<td>获取另一个线程的堆栈追踪信息。</td>
<td>此权限允许获取另一个线程的堆栈追踪信息。此操作可能允许执行恶意代码监视线程并发现应用程序中的弱点。</td>
</tr>
<tr>
<td>setDefaultUncaughtExceptionHandler</td>
<td>在线程由于未捕获的异常而突然终止时，设置将要使用的默认处理程序</td>
<td>此权限允许攻击者注册恶意的未捕获异常处理程序，可能会妨碍线程的终止</td>
</tr>
<tr>
<td>Preferences</td>
<td>表示得到 java.util.prefs.Preferences 的访问权所需的权限。java.util.prefs.Preferences 实现了用户或系统的根，这反过来又允许获取或更新 Preferences 持久内部存储中的操作。</td>
<td>如果运行此代码的用户具有足够的读/写内部存储的 OS 特权，则此权限就允许用户读/写优先级内部存储。实际的内部存储可能位于传统的文件系统目录中或注册表中，这取决于平台 OS。</td>
</tr>
</tbody>
</table>

1.3 可配置项详解
　　当批量配置的时候，有三种模式：

- directory/ 表示directory目录下的所有.class文件，不包括.jar文件
- directory/* 表示directory目录下的所有的.class及.jar文件
- directory/- 表示directory目录下的所有的.class及.jar文件，包括子目录
　　可以通过${}来引用系统属性，如：
```
"file:${{java.ext.dirs}}/*"
```


#### 2.Permission类实现类结构
```
Permission (java.security)
    CryptoPermission (javax.crypto)
        CryptoAllPermission (javax.crypto)
    BasicPermission (java.security)
        SecureCookiePermission (com.sun.deploy.security)
        NetworkPermission (jdk.net)
        DelegationPermission (javax.security.auth.kerberos)
        BridgePermission (sun.corba)
        SubjectDelegationPermission (javax.management.remote)
        BufferSecretsPermission (com.oracle.nio)
        InquireSecContextPermission (com.sun.security.jgss)
        PropertyPermission (java.util)
        SerializablePermission (java.io)
        NetPermission (java.net)
        AWTPermission (java.awt)
        WebServicePermission (javax.xml.ws)
        AudioPermission (javax.sound.sampled)
        SSLPermission (com.sun.net.ssl)
        ReflectPermission (java.lang.reflect)
        SecurityPermission (java.security)
        SSLPermission (javax.net.ssl)
        JavaScriptPermission (sun.plugin.liveconnect)
        SQLPermission (java.sql)
        MBeanTrustPermission (javax.management)
        MBeanServerPermission (javax.management)
        LoggingPermission (java.util.logging)
        DynamicAccessPermission (com.sun.corba.se.impl.presentation.rmi)
        JAXBPermission (javax.xml.bind)
        RuntimePermission (java.lang)
        SnmpPermission (com.sun.jmx.snmp)
        ManagementPermission (java.lang.management)
        AuthPermission (javax.security.auth)
        LinkPermission (java.nio.file)
    UnresolvedPermission (java.security)
    PrivateCredentialPermission (javax.security.auth)
    URLPermission (java.net)
    SelfPermission in PolicyFile (sun.security.provider)
    MBeanPermission (javax.management)
    AllPermission (java.security)
    ExecOptionPermission (com.sun.rmi.rmid)
    CardPermission (javax.smartcardio)
    FilePermission (java.io)
    ServicePermission (javax.security.auth.kerberos)
    SocketPermission (java.net)
```