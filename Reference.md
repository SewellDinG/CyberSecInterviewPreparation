## 面经

- [navisec 安全面试经验汇总](https://www.yuque.com/exploit/job/ycizkv)：覆盖面广，较详细
- [d1nfinite 信息安全面试题汇总](https://github.com/d1nfinite/sec-interview)：只有题，有出现频率
- [渗透测试面试近期热门题](https://www.freebuf.com/vuls/228750.html)：一问一答
- [Kit4y 计算机方向-2021届同学们的2020春季实习面经](https://github.com/Kit4y/2020-Interview-experience)：真实面经，安全方向较少
- [tiaotiaolong 信息安全方面面试清单](https://github.com/tiaotiaolong/sec_interview_know_list)：只有题，知识清单
- [Leezj9671 渗透测试和安全面试经验](https://github.com/Leezj9671/Pentest_Interview)：个人整理
- [SecYouth 信息安全实习和校招的面经、真题和资料（旧）](https://github.com/SecYouth/sec-jobs)：信息安全实习/秋招群

## 计算机网络

- [TCP三次握手、四次挥手出现意外情况时，如何保证稳定可靠？](https://wemp.app/posts/c3938333-9bb5-4758-93b4-039107260a80)
- [TCP 为什么是三次握手，而不是两次或四次？](https://www.zhihu.com/question/24853633)
- [为什么 IPv6 难以取代 IPv4？](https://draveness.me//whys-the-design-ipv6-replacing-ipv4)
- [CS-Notes](https://cyc2018.github.io/CS-Notes)：算法、计算机操作系统、计算机网络、数据库
- [高匿代理和透明代理的区别](https://huangzy.cn/article/2019/6/gn-and-tm-proxy)
- [彻底弄懂session，cookie，token](https://segmentfault.com/a/1190000017831088)

## 操作系统与编程语言

- [对比学习：Golang VS Python3](https://juejin.im/post/5cd945d6e51d453d022cb65f)

## 常见漏洞

- [SSRF学习笔记](https://evi1.cn/post/ssrf)、[SSRF 学习记录 【详细】](https://hackmd.io/@Lhaihai/H1B8PJ9hX)、[浅析Redis中SSRF的利用](https://xz.aliyun.com/t/5665)、[浅析SSRF认证攻击Redis](https://www.smi1e.top/%e6%b5%85%e6%9e%90ssrf%e8%ae%a4%e8%af%81%e6%94%bb%e5%87%bbredis/)：基础，RESP数据格式，利用gopher攻击MySQL、FastCGI、Redis等服务，Weblogic、Discuz X3.2 SSRF
- [如何将代码审计变成CTF---面向github代码审计](https://xz.aliyun.com/t/7256)：SSRF bypass，代码审计，三次绕过补丁
- [经典写配置漏洞与几种变形](https://www.leavesongs.com/PENETRATION/thinking-about-config-file-arbitrary-write.html)：PHP，bypass正则
- [暗度陈仓：基于国内某云的 Domain Fronting 技术实践](https://www.anquanke.com/post/id/195011)：域前置漏洞，CDN，Cobalt Strike's C2 Profile
- [Blind XXE详解与Google CTF一道题分析](https://www.freebuf.com/vuls/207639.html)：XXE OOB实践
- [SameSite Cookie，防止 CSRF 攻击](https://www.cnblogs.com/ziyunfei/p/5637945.html)：再见，CSRF
- [CSRF 漏洞的末日？关于 Cookie SameSite 那些你不得不知道的事](https://mp.weixin.qq.com/s?__biz=MzIwMDk1MjMyMg==&mid=2247484949&idx=1&sn=73f32260765596aa0fe773c755561308&chksm=96f41978a183906e0b4f21fddcbe2d19f667b6e6cf2bdb66160a744d161a7bac7b420acac005&mpshare=1&scene=1&srcid=&sharer_sharetime=1588122156973&sharer_shareid=a7d99c78943a626e64cade4860efb7d9#rd)：从同源到同站，深入解析SameSite对CSRF影响
- [细数 redis 的几种 getshell 方法](https://paper.seebug.org/1169)：写文件、反序列化、主从复制、Lua RCE
- [Redis相关安全学习小记](https://mp.weixin.qq.com/s?__biz=MzIzOTE1ODczMg==&mid=2247484020&idx=1&sn=06db219408f093c65d252c506ad502df&chksm=e92f16d7de589fc1df6fea9ebba21db9e8f8e76a0db89887b6eb9fa6f07f8c61d34977a6405c&mpshare=1&scene=1&srcid=&sharer_sharetime=1590661447618&sharer_shareid=a7d99c78943a626e64cade4860efb7d9#rd)：CRLF操作、绕过webshell中的?问号
- [对 Redis 在 Windows 下利用方式思考](https://www.t00ls.net/thread-56522-1-1.html)：处理脏数据、DLL劫持、link劫持、写配置文件、覆写exe
- [PHP 突破 disable_functions 常用姿势以及使用 Fuzz 挖掘含内部系统调用的函数](https://www.anquanke.com/post/id/197745)：bypass disable_functions总结
- [Fastjson 反序列化漏洞自动化检测](https://zhuanlan.zhihu.com/p/99075925)：fastjson成因
- [Fastcgi协议分析 && PHP-FPM未授权访问漏洞](https://www.leavesongs.com/PENETRATION/fastcgi-and-php-fpm.html)：直接给FPM发包为什么能执行RCE

## 内网渗透

- [内网渗透基础知识](http://mang0.me/archis/7db24e65)：工作组、域环境相关基本概念
- [彻底理解Windows认证 - 议题解读](https://payloads.online/archivers/2018-11-30/1)：从本地认证，网络认证，域认证介绍了相关流程及相应技术
- [《内网安全攻防-渗透测试实战指南》](https://github.com/SewellDinG/Pentest-Notes)：内网渗透的步骤和流程，以及相应的技术操作

## 工具使用

- [Cobalt Strike](https://www.secpulse.com/newpage/author?author_id=18480)：原理解析，基础系列
- [Cobalt Strike from Snowming04](http://blog.leanote.com/cate/snowming/Cobalt-Strike)：技巧，进阶

## 经验相关

- [红方人员作战执行手册](https://github.com/klionsec/RedTeamer)：渗透流程，入口权限 => 内网搜集/探测 => 免杀提权[非必须] => 抓取登录凭证 => 跨平台横向 => 入口维持 => 数据回传 => 定期权限维护