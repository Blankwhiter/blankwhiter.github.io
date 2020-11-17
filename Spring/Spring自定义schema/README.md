# Spring自定义schema

步骤
- 1.编写java bean
- 2.编写xsd配置文件
- 3.编写spring.handlers和spring.schemas
- 4.编写applicationContext.xml
- 5.编写NamespaceHandler和BeanDefinitionParser

1.编写java bean = RpcRegistry
```
public class RpcRegistry {
    private String ipAddr;
    private String echoApiPort;
    private String protocol;

    public String getIpAddr() {
        return ipAddr;
    }

    public void setIpAddr(String ipAddr) {
        this.ipAddr = ipAddr;
    }

    public String getEchoApiPort() {
        return echoApiPort;
    }

    public void setEchoApiPort(String echoApiPort) {
        this.echoApiPort = echoApiPort;
    }

    public String getProtocol() {
        return protocol;
    }

    public void setProtocol(String protocol) {
        this.protocol = protocol;
    }
}

```

2.编写xsd配置文件 = /src/main/resources/META-INF/nettyrpc.xsd 文件
```
<?xml version="1.0" encoding="UTF-8" ?>
<xsd:schema xmlns="http://www.newlandframework.com/nettyrpc" xmlns:xsd="http://www.w3.org/2001/XMLSchema"
            xmlns:beans="http://www.springframework.org/schema/beans"
            targetNamespace="http://www.newlandframework.com/nettyrpc"
            elementFormDefault="qualified" attributeFormDefault="unqualified">
    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="registry">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType">
                    <xsd:attribute name="ipAddr" type="xsd:string" use="required"/>
                    <xsd:attribute name="echoApiPort" type="xsd:string" use="required"/>
                    <xsd:attribute name="protocol" type="xsd:string" use="required"/>
                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>

</xsd:schema>

```

3.1.编写spring.handlers = /src/main/resources/META-INF/spring.handlers (固定位置 固定文件)
```
http\://www.newlandframework.com/nettyrpc=com.exmple.test.handler.NettyRpcNamespaceHandler
```
3.2.编写spring.schemas = /src/main/resources/META-INF/spring.schemas (固定位置 固定文件)
```
http\://www.newlandframework.com/nettyrpc/nettyrpc.xsd=nettyrpc.xsd
```

4.编写applicationContext.xml = applicationContext.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:nettyrpc="http://www.newlandframework.com/nettyrpc" xsi:schemaLocation="
	http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
    http://www.newlandframework.com/nettyrpc http://www.newlandframework.com/nettyrpc/nettyrpc.xsd">

    <nettyrpc:registry id="testRpc"  ipAddr="192.168.0.41" echoApiPort="ip-port" protocol="tcp"></nettyrpc:registry>
</beans>

```
5.1.编写NamespaceHandler = NettyRpcNamespaceHandler
```
import com.exmple.test.parser.NettyRpcRegistryParser;
import com.google.common.io.CharStreams;
import org.springframework.beans.factory.xml.NamespaceHandlerSupport;
import org.springframework.core.io.ClassPathResource;
import java.io.IOException;
import java.io.InputStreamReader;

public class NettyRpcNamespaceHandler extends NamespaceHandlerSupport {

    @Override
    public void init() {
        registerBeanDefinitionParser("registry", new NettyRpcRegistryParser());
    }
}

```
5.2.编写BeanDefinitionParser  = NettyRpcRegistryParser
```
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.xml.AbstractSingleBeanDefinitionParser;
import org.w3c.dom.Element;

public class NettyRpcRegistryParser  extends AbstractSingleBeanDefinitionParser {

    @Override
    protected Class<?> getBeanClass(Element element) {
        return RpcRegistry.class ;
    }

    @Override
    protected void doParse(Element element, BeanDefinitionBuilder builder) {
        String ipAddr = element.getAttribute("ipAddr");
        String echoApiPort = element.getAttribute("echoApiPort");
        String protocol = element.getAttribute("protocol");
        builder.setLazyInit(false);
        builder.addPropertyValue("ipAddr",ipAddr);
        builder.addPropertyValue("echoApiPort",echoApiPort);
        builder.addPropertyValue("protocol",protocol);
    }

}

```
测试样例
```
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("rpc-invoke-config-server.xml");
        Object testRpc = context.getBean("testRpc");
    }
```