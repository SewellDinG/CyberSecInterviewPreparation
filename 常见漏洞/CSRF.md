https://xz.aliyun.com/t/1243

https://xz.aliyun.com/t/7297

## 基本概念

CSRF（Cross-Site Request Forgery 跨站请求伪造，也被称为One Click Attack或Session Riding，XSRF），**攻击者盗用了你的身份，以你的名义发送恶意请求，对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作**，比如以你的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。

**CSRF 攻击依赖的是跨站请求会自动带上用户 cookie，进而可以伪造请求，代替用户执行敏感操作。**

寻找CSRF：找浏览器和服务器之间的交互，然后依次分析。

## 原理

![](https://img-blog.csdnimg.cn/20200225224153721.jpg)

1. 用户C打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
2. 在用户信息通过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
3. 用户未退出网站A之前，在同一浏览器中，打开一个标签页访问恶意网站B；
4. 恶意网站B接收到用户请求后，返回一些攻击性代码，并发出一个请求访问第三方站点A；
5. 浏览器在接收到这些攻击性代码后，根据恶意网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自恶意网站B的恶意代码被执行。

## 攻击

GET请求：

- 链接利用（a标签）
- iframe利用。可以设置iframe的style为display:none，以此来不显示iframe加载的内容。
- img标签利用。img标签内的内容会随着页面加载而被请求，以此src指向的位置会在页面加载过程中进行请求。
- background利用。可以利用CSS中background样式中的url来加载远程机器上的内容，从而对url中的内容发送HTTP请求。

POST请求：

- 模拟form表单，利用js自动提交，`<script type="text/javascript">document.csrf.submit();</script>`。

## 防御（各种方法对比，尤其是SameSite Cookie）

攻击条件：

- 登录受信任站点WebA，并在本地生成Cookie。
- 在不登出WebA的情况下，访问站点WebB。

但每次登出A不会回回清理cookie，也不会避免访问B，所以从攻击条件来看并没有合适的防御手段。

CSRF攻击特点：

- **攻击请求基本都是跨域的**。可以在服务端对HTTP请求头部的Referer字段进行检查。一般情况下，用户提交的都是站内的请求，其Referer中的来源地址应该是站内的地址。至关重要的一点是，**前端的JavaScript无法修改Referer字段**，这也是这种防御方法成立的条件。
- **伪造的请求**。我们来想一下，攻击者为什么能够伪造请求呢？换句话说，攻击者能够伪造请求的条件是什么呢？纵观之前我们伪造的所有请求，无一例外，**请求中所有参数的值都是我们可以预测的，如果出现了攻击者无法预测的参数值，那么将无法伪造请求，CSRF攻击也不会发生。**基于这一点，我们有了如下的防御方法：
  - \* 添加验证码。如果使用验证码，每次操作都需要用户进行互动，从而简单有效的防御了CSRF攻击。
  - \* 验证HTTP Referer请求头。它记录了该HTTP请求的来源地址，因此要防御CSRF攻击，服务端只需要对于每一个请求验证其Referer值即可。
  - \* 验证一次性令牌token（Anti-CSRF Token）。每当访问该页面时，服务端会根据时间戳、用户ID、随机串等因子生成一个随机的token值并传回到前端的表单中，当提交表单时，token会作为一个参数提交到服务端进行验证。在这个请求过程中，token的值也是攻击者无法预知的，而且由于同源策略的限制，攻击者也无法使用JavaScript获取其他域的token值，所以这种方法可以成功防御CSRF攻击，也是现在用的最多的防御方式。BUT：如果在同域下存在XSS漏洞，那么基于token的CSRF防御将很容易被击破。
  - 限制Session生命周期。

除此之外的防御方法：

- 自定义HTTP头字段或Cookie字段。
- 尽量采用POST类型传参，减少了请求被直接伪造的可能。

\*\* 以上都是老旧的方法，除此之外，可以利用**set-cookie中的SameSite属性**，被誉为CSRF终极解决办法：

- SameSite-Cookies是一种机制，用于定义Cookie如何跨域发送，这是谷歌开发的一种安全机制，目的是尝试阻止CSRF（Cross-site request forgery 跨站请求伪造）以及XSSI（Cross Site Script Inclusion (XSSI) 跨站脚本包含）攻击。
- 对于 SameSite=Strict 的 cookie：只有同站请求会携带此类 cookie。
- 对于 SameSite=None 的 cookie：同站请求和跨站请求都会携带此类 cookie。
- Lax 的行为介于 None 和 Strict 之间。对于 SameSite=Lax 的 cookie，除了同站请求会携带此类 cookie 之外，特定情况（安全的顶级域名跳转）的跨站请求也会携带此类 cookie。
- **2016年出现的SameSite属性，默认为None；在19年11月新提案IBC规定SameSite值默认是Lax。**同时，如果是指None，则需要在设置Secure属性，换句话说，只有在HTTPS的情况下才能使用None。
- [CSRF 漏洞的末日？关于 Cookie SameSite 那些你不得不知道的事](https://mp.weixin.qq.com/s?__biz=MzIwMDk1MjMyMg==&mid=2247484949&idx=1&sn=73f32260765596aa0fe773c755561308&chksm=96f41978a183906e0b4f21fddcbe2d19f667b6e6cf2bdb66160a744d161a7bac7b420acac005&mpshare=1&scene=1&srcid=&sharer_sharetime=1588122156973&sharer_shareid=a7d99c78943a626e64cade4860efb7d9#rd)

## 绕过

- 绕过正则表达式。DVWA中的CSRF Medium级会判断Referer，`if( eregi( $_SERVER[ 'SERVER_NAME' ], $_SERVER[ 'HTTP_REFERER' ] ) )`，但**eregi()**只是判断SERVER_NAME是否在HTTP_REFERER中，因此可以构造攻击页面进行绕过，文件名为SERVER_NAME.php，如192.168.1.3.html，页面内容`<img src="http://192.168.1.3/DVWA/vulnerabilities/csrf/?password_new=qwzf&password_conf=qwzf&Change=Change#" border="0" style="display:none;"/>`。
- 移除referer字段。
- 使用XSS辅助。DVWA中的CSRF High级添加了Anti-CSRF Token。暴力破解Token并不可行；由于浏览器普遍对跨域请求资源有访问控制，所以在恶意页面中运行js脚本取得目标页面的token也不可行。但是，可以利用XSS获取Token，然后再进行CSRF。
- 删除token参数或发送空token。应用可能会在token存在或者token参数不为空的时候才检查token的有效性。逻辑漏洞。
- 使用另一个session的CSRF token。应用可能只是检查token是否合法，但是不检查token是否确实归属于当前用户。逻辑漏洞。
- 更改请求方法。GET <-->POST。

## **CSRF与XSS的区别**
CSRF听起来像跨站脚本攻击(XSS)，但与XSS不同。XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站。

什么意思呢？XSS利用的是用户对指定网站的信任，CSRF利用是网站对用户浏览器的信任。

## Self-XSS + CSRF

