# （个人学习笔记）webservice

## 如何实现一个webservice

1. 协议层：`tcp/ip`
2. 应用层：`http`
3. SOAP：`http`+`xml`

## 分布式通信框架-webservice分析

### 什么是webservice？

> webservice也可以叫xml web service webservice, 轻量级的独立的通讯技术

1. 1. 基于web的服务：服务端提供的服务接口让客户端访问
   2. 跨语言、跨平台的整合方案
   3. webservice基于jdk提供的httpServer实现一个server服务

### 为什么使用webservice

**跨语言调用的解决方案**

### 什么时候去使用webservice

> 电商平台，订单的物流状态。 
>
>  .net实现的webservice服务接口

### webservice中的一些概念

**WSDl(web service definition language  webservice 定义语言)**

> webservice服务需要通过wsdl文件来说明自己有什么服务可以对外调用。并且有哪些方法、方法里面有哪些参数
>
> wsdl基于XML（可扩展标记语言）去定义的

1. 1. 对应一个.wsdl的文件类
   2. 定义了webservice的服务端和客户端应用进行交互的传递数据和相应数据格式和方式
   3. 一个webservice对应唯一一个wsdl文档

**SOAP（simple object access protocal简单对象访问协议）**

> http+xml
>
> webservice通过http协议发送和接收请求时， 发送的内容（请求报文）和接收的内容（响应报文）都是采用xml格式进行封装
>
> 这些特定的HTTP消息头和XML内容格式就是SOAP协议

1. 一种简单、基于HTTP和XML的协议
2. soap消息：请求和相应消息
3. http+xml报文

**SEI（webservice endpoint interface webservice的终端接口）**

> webservice 服务端用来处理请求的接口，也就是发布出去的接口

### 分析WSDL文档

**Types标签**

![image-20180829104501843](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829104501843.png)

**Schema标签**

![image-20180829104546304](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829104546304.png)

``` xml
<sayHello>
   <arg0>String</arg0>
</sayHello>

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:web="http://webservice.mic.vip.gupao.com/">
   <soapenv:Header/>
   <soapenv:Body>
      <web:sayHello>
         <!--Optional:-->
         <arg0>?</arg0>
      </web:sayHello>
   </soapenv:Body>
</soapenv:Envelope>

<sayHelloResponse>
   <return>string</return>
</sayHelloResponse>

<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
   <S:Body>
      <ns2:sayHelloResponse xmlns:ns2="http://webservice.mic.vip.gupao.com/">
         <return>Hello ,Mic,I'am 菲菲</return>
      </ns2:sayHelloResponse>
   </S:Body>
</S:Envelope>
```

**Message**:`定义了在通信中使用的消息的数据结构`

![image-20180829104638956](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829104638956.png)







**PortType**`定义服务端的SEI input/output表示输入/输出数据`

![image-20180829104750176](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829104750176.png)

**Binding标签**

1. type属性：引用portype <soap:binding style=”document”>
2. oneration：指定实现方法
3. input/output：表示输入和输出的数据类型

![image-20180829105023367](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829105023367.png)

**Service标签**

1. service：服务端的一个webservice的容器
2. name属性：指定客户端的容器类
3. address：当前webservice的请求地址

![image-20180829105005273](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829105005273.png)

### 实现交互过程

![image-20180829105118971](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829105118971.png)

![image-20180829105127565](/var/folders/hg/_bgwc8xs7vdcc09kmk7w2ttc0000gn/T/abnerworks.Typora/image-20180829105127565.png)

### 实现方式

#### Axis/Axis2

apache开源的webservice工具

#### cxf

Celtix+Xfire  应用比较广泛，因为集成了`spring`

#### Xfire

高性能的Webservice

HTTP+JSON（新的webservice）

HTTP+XML



`内容出处：咕泡学院MIC老师` 