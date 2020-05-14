之前写过一个关于HTTPS的文章，最近又重新看了看，发现还有很多地方可以补充完善，正好最近也看了关于RSA加密算法的知识，正好一起完善一下。

分割线
--------

# 1.什么是HTTPS
HTTPS简单的说就是安全版的HTTP。
因为HTTP协议的数据都是明文进行传输的，所以对于一些敏感信息的传输就很不安全，为了安全传输敏感数据，网景公司设计了SSL（Secure Socket Layer），在HTTP的基础上添加了一个安全传输层，对所有的数据都加密后再进行传输，客户端和服务器端收到加密数据后按照之前约定好的秘钥解密。

# 2.HTTPS是如何保证安全的
HTTPS的安全性是建立在密码学的基础之上的，尤其是非对称加密算法RSA在HTTPS中起到了极为关键的作用。
## 2.1常见的加密算法
目前我们常见的加密算法主要分两种，对称加密和非对称加密。
* 对称加密，顾名思义就是加密和解密都是用一个同样的秘钥，它的优点就行加解密的速度很快，缺点就是尤其需要注意秘钥的传输，不能泄露
* 非对称加密，非对称加密有一对秘钥公钥和私钥。使用公钥加密，然后使用私钥解密。公钥可以公开的发送给任何人。使用公钥加密的内容，只有私钥可以解开。安全性比对称加密大大提高。缺点是和对称加密相比速度较慢，加解密耗费的计算资源较多。

在HTTPS中则是综合了这两种算法的优缺点。使用非对称加密完成秘钥的传递，然后使用对称秘钥进行数据加密和解密。

## HTTPS的交互过程
通过上面的描述，我们已经能大概知道HTTPS是使用加密算法在浏览器和服务器之前传递秘钥，然后再使用秘钥完成信息的加解密。所以这个秘钥是如何生成的，还有秘钥是如何在浏览器和服务器之间传递的就成了HTTPS的关键，下面我们来详细的了解一下这个过程。 

# 2.HTTPS证书
现在HTTPS基本已经成为了一个网站的标配。想要给一个网站添加对HTTPS的支持，就需要针对这个网站的域名申请证书。

 大家都知道要使用https，需要在网站的服务器上配置https证书（一般是nginx，或者tomcat），证书可以使用自己生成，也可以向专门的https证书提供商进行购买。这两种的区别是自己生成的证书是不被浏览器信任的,所以当访问的时候回提示不安全的网站，需要点击信任之后才能继续访问

![自己生成的](http://upload-images.jianshu.io/upload_images/2829175-8749dcdef5960ff5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而购买的https证书会提示安全
![DV,OV](http://upload-images.jianshu.io/upload_images/2829175-5c621e00597c6c00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![EV](http://upload-images.jianshu.io/upload_images/2829175-ef3241aa68eeb225.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是因为浏览器中预置了一些https证书提供商的证书，在浏览器获取到服务器的https证书进行验证的时候就知道这个https证书是可信的；而自己生成的证书，浏览器获取到之后无法进行验证是否可信，所以就给出不安全的提示。
下面对具体的一些知识点进行介绍
---
#### 1. 什么是https
 https简单的说就是安全版的http，因为http协议的数据都是明文进行传输的，所以对于一些敏感信息的传输就很不安全，为了安全传输敏感数据，网景公司设计了SSL（Secure Socket Layer），在http的基础上添加了一个安全传输层，对所有的数据都加密后再进行传输，客户端和服务器端收到加密数据后按照之前约定好的秘钥解密。


#### 2. 加密和解密
Https的发展和密码学的发展是分不开的。大家应该知道加密方式可以大体分为对称加密和非对称加密（反正我就知道这两种）
- 对称加密，就是加密和解密都是用同一个秘钥，这种方式优点就是速度快，缺点就是在管理和分配秘钥的时候不安全。
- 非对称加密算法，非对称加密有一个秘钥对，叫做公钥和私钥，私钥自己持有，公钥可以公开的发送给使用的人。使用公钥进行加密的信息，只有和其配对的私钥可以解开。目前常见的非对称加密算法是RSA，非对称的加密算法的优点是安全，因为他不需要把私钥暴露出去。
在正式的使用场景中一般都是对称加密和非对称加密结合使用，使用非对称加密完成秘钥的传递，然后使用对称秘钥进行数据加密和解密


#### 3. https证书的申请流程
1 在服务器上生成CSR文件（证书申请文件，内容包括证书公钥、使用的Hash算法、申请的域名、公司名称、职位等信息）
可以使用命令在服务器上生成；也可以使用线上的工具进行生成，线上的工具会把公钥加入到CSR文件中，并同时生成私钥。

![CSR文件内容](http://upload-images.jianshu.io/upload_images/2829175-31655affc6683c0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2 把CSR文件和其他可能的证件上传到CA认证机构，CA机构收到证书申请之后，使用申请中的Hash算法，对部分内容进行摘要，然后`使用CA机构自己的私钥对这段摘要信息进行签名`，

![CA机构进行签名](http://upload-images.jianshu.io/upload_images/2829175-06f817d855a63dd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3 然后CA机构把签名过的证书通过邮件形式发送到申请者手中。

4 申请者收到证书之后部署到自己的web服务器中。下面会在写一篇关于部署的文章

当然这是不通过CA代理机构进行申请的流程，现在网上有好多CA的代理机构，像腾讯云，阿里云都可以申请https证书，流程都差不多。
[阿里云申请证书流程](http://www.chinaz.com/web/2017/0105/639110.shtml)

#### 4. 客户端（浏览器）和服务器端交互流程

![客户端服务端交互](http://upload-images.jianshu.io/upload_images/2829175-9385a8c5e94ad1da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

1. client Hello,客户端（通常是浏览器）先向服务器发出加密通信的请求
> （1） 支持的协议版本，比如TLS 1.0版。
（2） 一个客户端生成的随机数 random1，稍后用于生成"对话密钥"。
（3） 支持的加密方法，比如RSA公钥加密。
（4） 支持的压缩方法。
2. 服务器收到请求,然后响应 (server Hello)
>（1） 确认使用的加密通信协议版本，比如TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。
（2） 一个服务器生成的随机数random2，稍后用于生成"对话密钥"。
（3） 确认使用的加密方法，比如RSA公钥加密。
（4） 服务器证书。

![证书内容](http://upload-images.jianshu.io/upload_images/2829175-fcc431ce19db1379.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![证书内容](http://upload-images.jianshu.io/upload_images/2829175-d399116519dbda05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 客户端收到证书之后会首先会进行验证
- 验证流程
> 1. 我们知道CA机构在签发证书的时候，都会使用自己的私钥对证书进行签名
证书里的签名算法字段 sha256RSA 表示，CA机构使用sha256对证书进行摘要，然后使用RSA算法对摘要进行私钥签名，而我们也知道RSA算法中，使用私钥签名之后，只有公钥才能进行验签。
>2. 如果我们使用的是购买的证书，那么很有可能，颁发这个证书的CA机构的公钥已经预置在操作系统中。这样浏览器就可以使用CA机构的公钥对服务器的证书进行验签。确定这个证书是不是由正规的CA机构颁发的。验签之后得到CA机构使用sha256得到的证书摘要，然后客户端再使用sha256对证书内容进行一次摘要，如果得到的值和验签之后得到的摘要值相同，则表示证书没有被修改过。
>3. 如果验证通过，就会显示上面的安全字样，如果服务器购买的证书是更高级的EV类型，就会显示出购买证书的时候提供的企业名称。如果没有验证通过，就会显示不安全的提示。
- 生成随机数
> 验证通过之后，客户端会生成一个随机数pre-master secret，然后使用证书中的公钥进行加密，然后传递给服务器端

**PreMaster secret**
> PreMaster Secret是在客户端使用RSA或者Diffie-Hellman等加密算法生成的。它将用来跟服务端和客户端在Hello阶段产生的随机数结合在一起生成 Master Secret。在客户端使用服务端的公钥对PreMaster Secret进行加密之后传送给服务端，服务端将使用私钥进行解密得到PreMaster secret。也就是说服务端和客户端都有一份相同的PreMaster secret和随机数。
>PreMaster secret前两个字节是TLS的版本号，这是一个比较重要的用来核对握手数据的版本号，因为在Client Hello阶段，客户端会发送一份加密套件列表和当前支持的SSL/TLS的版本号给服务端，而且是使用明文传送的，如果握手的数据包被破解之后，攻击者很有可能串改数据包，选择一个安全性较低的加密套件和版本给服务端，从而对数据进行破解。所以，服务端需要对密文中解密出来对的PreMaster版本号跟之前Client Hello阶段的版本号进行对比，如果版本号变低，则说明被串改，则立即停止发送任何消息。
[pre-master secret ](http://www.linuxidc.com/Linux/2016-05/131147.htm)

4. 服务器收到使用公钥加密的内容，在服务器端使用私钥解密之后获得随机数pre-master secret，然后根据radom1、radom2、pre-master secret通过一定的算法得出session Key和MAC算法秘钥，作为后面交互过程中使用对称秘钥。同时客户端也会使用radom1、radom2、pre-master secret，和同样的算法生成session Key和MAC算法的秘钥。
> 生成session Key的过程中会用到PRF(Pseudorandom Function伪随机方法)来生成一个key_block,然后再使用key_block,生成后面使用的秘钥。
key_block = PRF(SecurityParameters.master_secret,"key expansion",SecurityParameters.server_random +SecurityParameters.client_random);
PRF是在规范中约定的伪随机函数

>  在信息交互过程中用到的秘钥有6个分别是。客户端和服务器端分别使用相同的算法生成。

|秘钥名称|秘钥作用|
|:--:|:--:|
|client_write_MAC_key[SecurityParameters.mac_key_length]|客户端发送数据使用的摘要MAC算法|
|server_write_MAC_key[SecurityParameters.mac_key_length]|服务端发送数据使用摘要MAC算法|
|client_write_key[SecurityParameters.enc_key_length]|客户端数据加密，服务端解密|
|server_write_key[SecurityParameters.enc_key_length]|服务端加密，客户端解密|
|client_write_IV[SecurityParameters.fixed_iv_length]|初始化向量，运用于分组对称加密|
|server_write_IV[SecurityParameters.fixed_iv_length]|初始化向量，运用于分组对称加密|


5. 然后再后续的交互中就使用session Key和MAC算法的秘钥对传输的内容进行加密和解密。
>具体的步骤是先使用MAC秘钥对内容进行摘要，然后把摘要放在内容的后面使用sessionKey再进行加密。对于客户端发送的数据，服务器端收到之后，需要先使用client_write_key进行解密，然后使用client_write_MAC_key对数据完整性进行验证。服务器端发送的数据，客户端会使用server_write_key和server_write_MAC_key进行相同的操作。