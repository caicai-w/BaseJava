### 1.Tomcat架构

Coyota连接器

Catalina Servlet容器

#### 1.1Coyota

客户端通过Coyota和服务器建立连接，Coyota封装了底层的网络通信模块，就是socket，它把Socket输入转换为Request对象，再交给Catalina容器处理。

#### 1.2 Coyota中核心概念

EndPoint：是具体接受Socket的处理类，面向TCP/IP协议，有NIO，NIO2，ARP三种具体实现类。

Processer：负责构造Request，Response对象，通过Adapter，适配器模式，再把Request变成ServletRequest交给Catalina处理，它是对应用层的抽象，按照协议的不同提供了3种实现类：HTTP1.1，AJP，HTTP2.0。

ProtocalHandler：封装了EndPoint和Processer。

#### 1.3 Catalina

Catalina负责的是逻辑的处理，一个Catalina可以连接到多个连接器，Catalina下又有很多容器。有Engine、host、Context、Wrapper。

一个Engine下可以有多个Host，一个host就是说可以有不同的ip，一个host下又可以有多个Context，就是你的服务，一个Context下又可以有多个wapper，一个wapper就是一个Servlet。

