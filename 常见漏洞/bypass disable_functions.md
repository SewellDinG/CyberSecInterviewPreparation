[PHP 突破 disable_functions 常用姿势以及使用 Fuzz 挖掘含内部系统调用的函数](https://www.anquanke.com/post/id/197745)

## bypass disable_functions

1. 黑名单 bypass

   众所周知，disable_functions 是基于黑名单来实现对某些函数使用的限制的，既然是黑名单有时候就难免会有漏网之鱼。

2. LD_PRELOAD & putenv() bypass disable_functions

   LD_PRELOAD 是一个 Unix 中比较特殊的环境变量，它允许你定义在程序运行前优先加载的动态链接库。原理就是劫持系统函数，使程序加载恶意动态链接库文件，从而执行系统命令等敏感操作。

   strace跟踪记录命令执行过程中调用的系统函数，找无参数的函数进行劫持，如geteuid()；

   真实：利用 GNU C 中的constructor构造函数或destructor析构函数，一个在main()之前执行，一个在main()或exit()之后执行。**在 PHP 运行过程中只要有新程序启动，在我们加载恶意动态链接库的条件下，便可以执行 .so 中的恶意代码。**

   如 PHP 中经典的 mail() 函数：在PHP中先使用 putenv() 函数设置LD_PRELOAD环境变量，在调用mail('','','','')函数触发。同理还有error_log('',1) 或者 mb_send_mail('','','') 和 imap_mail("1@a.com","0","1","2","3")（如果 PHP 开启了 imap 模块）。

   同理还有，如果 PHP 安装了 imagick 模块，则还可以使用 Imagick，其在遇到 MPEG format 的时候，会调用 ffmpeg 进程来处理，也会有新程序启动。

3. ImageMagick bypass disable_functions

4. PHP 5.x Shellshock Exploit (bypass disable_functions)

5. PHP-FPM/FastCGI bypass disable_functions

   FastCGI 是 CGI 协议的升级版，用于封装 webserver 发送给 php 解释器的数据，通过 PHP-FPM 程序按照 FastCGI 协议进行处理和解析数据，返回结果给 webserver。

   原理：[https://zhuanlan.zhihu.com/p/75114351](https://zhuanlan.zhihu.com/p/75114351)，php-fpm是一个fastcgi协议解析器，负责按照fastcgi的协议将TCP流解析成真正的数据。**PHP-FPM默认监听9000端口，我们可以自己构造fastcgi协议，和fpm进行通信，以此来bypass。**

   蚁剑：生成ext，攻击php-fpm执行ext，在目标机器本地开启一个新的php server，然后再用受限制的 webshell 转发请求到新开启的php server上去。新开启的php server不使用php.ini，从而bypass disable_functions。

6. Windows 系统组件 COM

   COM（Component Object Model）组件对象模型，是一种跨应用和语言共享二进制代码的方法。COM 可以作为 DLL 被本机程序载入也可以通过 DCOM 被远程进程调用。

   在 PHP 中通过 COM 对象的 exec() 方法即可bypass disable_functions。

7. PHP 5.2.3 win32std extension safe_mode and bypass disable_functions

8. FFI 绕过 disable_functions

   PHP7.4 的一个新特性 FFI（Foreign Function Interface），即外部函数接口，可以让我们在 PHP 中调用 C 代码。

9. Apache mod_cgi 修改 .htaccess 绕过限制

   条件苛刻，需要 Apache 开启 AllowOverride、开启 cgi_module、.htaccess 文件可写、cgi 程序可执行。

10. 利用 PHP bug Bypass disable_functions

11. PHP imap_open RCE 漏洞 （CVE-2018-19518）