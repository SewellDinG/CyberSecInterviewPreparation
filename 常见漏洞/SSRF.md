## 原理

SSRF，Server-Side Request Forgery，服务端请求伪造，是一种由攻击者构造形成由服务器端发起请求的一个漏洞。一般情况下，SSRF 攻击的目标是从外网无法访问的内部系统。漏洞形成的原因大多是因为服务端提供了从其他服务器应用获取数据的功能且没有对目标地址作正确的过滤和限制。

- 利用一个可以发起网络请求的服务作为跳板绕过防火墙，进而攻击内部其他服务。
- 本质上是属于信息泄露，其中内网表述包括本地。
- 由于服务端提供了从其他服务器应用获取数据的功能，但没有对用户可控的目标地址做过滤与限制。
- PHP中的curl()，file_get_contents()，fsockopen()等函数。

## 攻击（利用）

- 对内网主机进行端口扫描，获取一些服务的 Banner 信息。
- 对内网 Web 应用进行指纹识别，通过访问默认文件实现。
- 攻击运行在内网的应用程序，攻击 redis、fastcgi、mysql等。
- 攻击内外网的 Web 应用，通过GET/POST传参就可以实现的攻击，如：st2、sqli等。
- http/s 协议，获取Web请求内容。
- file 协议 ，本地文件传输协议，用于读取本地文件，`file:///etc/passwd`。
- gopher 协议，文件搜集获取网络协议，出现在http协议之前，可以模拟GET/POST请求（换行使用%0d%0a，空白行%0a）。因此可以先截获GET/POST请求包，再构造成符合gopher协议的请求。gopher协议传输的数据默认会吞掉首字符，`gopher://localhost:2222/_hello%0agopher`需要在数据前加一个无用字符。在地址栏中利用要再URL编码一次。
- dict 协议，字典服务器协议，是基于查询响应的TCP协议。dict协议有个特性：`dict://serverip:port/cmd:p1:p2` 向服务器的端口请求 cmd p1 p2，并在末尾自动补上rn(CRLF)。因此在漏洞没有屏蔽回显的情况下，可以利用dict协议获取敏感信息。

补充：基于 TCP Stream 且不做交互的点都可以进行攻击利用，包括但不限于：

- HTTP GET/POST -> Struts2、Web Vuls...
- Redis、Memcache、无认证MySQL
- PHP-FPM（FastCGI）
- SMTP
- Telnet
- 基于一个 TCP 包的 exploit
- FTP（不能实现上传下载文件，但是在有回显的情况下可用于爆破内网 FTP）


## 防御（修复）

- 限制协议为HTTP/S，`curl_setopt($ch, CURLOPT_PROTOCOLS, CURLPROTO_HTTP | CURLPROTO_HTTPS);`。
- 禁止301、302跳转，`删掉curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);`。
- 屏蔽回显，`curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);`。
- 设置URL白名单、IP白名单。
- 限制可请求的端口，防火墙策略。
- 统一错误信息，避免用户可以根据错误信息来判断远端服务器的端口状态
- 过滤返回信息，属于被动防御，压力较大。

## 绕过

URL组成：

![](https://github.com/SewellDinG/CyberSecInterviewPreparation/blob/master/常见漏洞/images/URL.png)

- 添加端口号：`http://127.0.0.1:80`
- 利用短网址：`https://dwz.cn/6ZSDAVpg`
- 利用解析URL绕过正则缺陷：`http://abc@127.0.0.1`
- 各种IP地址的进制转换：十进制转换、八进制转换、十六进制转换、不同进制组合转换
- 利用省略模式（Linux下）：`127.1  => 127.0.0.1`、`0 or http://0/ => 0.0.0.0`
- 通过非HTTP/S协议：dict://、file://、gopher://。
- 利用302 Redirect绕过：当url协议限制为http/s时，可以利用follow redirect特性，在VPS上构造临时(302)或永久(301)跳转在利用。
- 利用DNS解析（302？）：xip.io、xip.name
- 利用Enclosed alphanumerics：`ⓔⓧⓐⓜⓟⓛⓔ.ⓒⓞⓜ >>> http://example.com`
- 利用句号：`127。0。0。1 >>> 127.0.0.1`
- 利用IPv6：`http://[::]:80/  >>>  http://127.0.0.1`
- 利用 DNS Rebinding（DNS重新绑定攻击）：设置 DNS 服务器的 TTL 为0，来使得每次解析返回不同的 IP。当在校验 IP 的时候 DNS 解析返回合法的值，等后续重新请求内容的时候 DNS 解析返回内网 IP。

![](https://github.com/SewellDinG/CyberSecInterviewPreparation/blob/master/常见漏洞/images/DNSRebinding.png)

补充：

### 1、更改IP地址写法

例如192.168.0.1

```
(1)、8进制格式：0300.0250.0.1
(2)、16进制格式：0xC0.0xA8.0.1

(3)、10进制整数格式：3232235521
(4)、16进制整数格式：0xC0A80001
```

还有一种特殊的省略模式，例如10.0.0.1这个IP可以写成10.1

```
1.  http://0/
2.  http://127.1/
3. 利用ipv6绕过，http://[::1]/
4.  http://127.0.0.1./
```

### 2、利用302跳转

（1）、利用xip.io、xip.name。

```
10.0.0.1.xip.io 10.0.0.1
www.10.0.0.1.xip.io 10.0.0.1
mysite.10.0.0.1.xip.io 10.0.0.1
foo.bar.10.0.0.1.xip.io 10.0.0.1

10.0.0.1.xip.name  resolves to  10.0.0.1
www.10.0.0.2.xip.name  resolves to  10.0.0.2
foo.10.0.0.3.xip.name  resolves to  10.0.0.3
bar.baz.10.0.0.4.xip.name  resolves to  10.0.0.4

这里的服务原理为DNS+302
[Sewell]: ~
➜  nslookup hhhhh.hhhh.118.24.122.217.xip.io
Server:		211.137.191.26
Address:	211.137.191.26#53

Non-authoritative answer:
Name:	hhhhh.hhhh.118.24.122.217.xip.io
Address: 118.24.122.217
```

（2）、由于上述方法中包含了10.0.0.1这种内网IP地址，可能会被正则表达式过滤掉，我们可以通过用短地址的方式绕过。

（3）、限制协议时，可以利用follow redirect特性，在VPS上构造临时(302)或永久(301)跳转在利用。

### 3、DNS rebinding

DNS重绑定可以利用于ssrf绕过 ，bypass 同源策略等

我们需要一个域名，并且将这个域名的解析指定到我们自己的DNS Server，在我们的可控的DNS Server上编写解析服务，设置TTL时间为0。

```
服务器端获得URL参数，进行第一次DNS解析，获得了一个非内网的IP
对于获得的IP进行判断，发现为非黑名单IP，则通过验证
服务器端对于URL进行访问，由于DNS服务器设置的TTL为0，所以再次进行DNS解析，这一次DNS服务器返回的是内网地址。
由于已经绕过验证，所以服务器端返回访问内网资源的结果。
```

## 局限性

经过测试发现 Gopher 的以下几点局限性：

- 大部分 PHP 并不会开启 fopen 的 gopher wrapper
- file_get_contents 的 gopher 协议不能 URLencode
- file_get_contents 关于 Gopher 的 302 跳转有 bug，导致利用失败
- PHP 的 curl 默认不 follow 302 跳转
- curl/libcurl 7.43  gopher 协议存在 bug（%00 截断），经测试 7.49 可用

## 工具

[SSRFmap](https://github.com/swisskyrepo/SSRFmap)：将burp导出的请求包作为输入，并选取一个参数进行SSRF Fuzz，支持需要服务。

[Gopherus](https://github.com/tarunkant/Gopherus)：针对不同服务生成不同gopher请求。

## 实例

- 代码审计。三次绕过补丁。自动更新文章的爬虫，可控制url参数。[1](https://xz.aliyun.com/t/7256)
- 使用xray的反连平台来利用302和dns rebinding。[2](https://xray.cool/xray/#/scenario/reverse_server_ssrf)
