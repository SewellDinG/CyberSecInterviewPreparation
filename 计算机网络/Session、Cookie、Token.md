Ref：[https://segmentfault.com/a/1190000017831088](https://segmentfault.com/a/1190000017831088)

## Session、Cookie、Token？

- session存储于服务器，可以理解为一个状态列表，拥有一个唯一识别符号sessionId，通常存放于cookie中。服务器收到cookie后解析出sessionId，再去session列表中查找，才能找到相应session。依赖cookie。
- cookie类似一个令牌，装有sessionId，存储在客户端，浏览器通常会自动添加。
- token也类似一个令牌，无状态，用户信息都被加密到token中，服务器收到token后解密就可知道是哪个用户。需要开发者手动添加。因此可以抵抗CSRF。
- jwt只是一个跨域认证的方案