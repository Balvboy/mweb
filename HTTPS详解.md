之前写过一个关于HTTPS的文章，最近又重新看了看，发现还有很多地方可以补充完善，正好最近也看了关于RSA加密算法的知识，正好一起完善一下。


--------

[toc]

#什么是HTTPS
HTTPS简单的说就是安全版的HTTP。
因为HTTP协议的数据都是明文进行传输的，所以对于一些敏感信息的传输就很不安全，为了安全传输敏感数据，网景公司设计了SSL（Secure Socket Layer），在HTTP的基础上添加了一个安全传输层，对所有的数据都加密后再进行传输，客户端和服务器端收到加密数据后按照之前约定好的秘钥解密。

#HTTPS是如何保证安全的
HTTPS的安全性是建立在密码学的基础之上的，尤其是非对称加密算法RSA在HTTPS中起到了极为关键的作用。


## HTTPS的交互过程
通过上面的描述，我们已经能大概知道HTTPS是使用加密算法在浏览器和服务器之前传递秘钥，然后再使用秘钥完成信息的加解密。所以这个秘钥是如何生成的，还有秘钥是如何在浏览器和服务器之间传递的就成了HTTPS的关键，下面我们来详细的了解一下这个过程。 

![客户端与服务端交互](https://upload-images.jianshu.io/upload_images/2829175-b4a66afd059747b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

### 1.交互流程 
1. Client Hello 客户端（通常是浏览器）先向服务器发出加密通信的请求,请求大概中包括下面内容
    *  支持的协议版本，比如TLS 1.0版。
    *  一个客户端生成的随机数 Random Number-RNc，稍后用于生成"对话密钥"。
    *  支持的加解密方法，比如对称加密支持AES，秘钥交换算法支持RSA、DH，签名算法支持sha256等。(在更高版本的TLS协议中，交换的是密码学套件，所谓的套件就是一整套的加解密，秘钥交换方案)。
    *  支持的压缩方法。
2. 服务器收到请求,然后响应
    *  确认使用的加密通信协议版本，比如TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。
    *  一个服务器生成的随机数Random Number-RNs，稍后用于生成"对话密钥"。
    *  确认使用的加密方法、秘钥交换算法、签名算法等等。
    *  把服务器证书发送给客户端。
    *  服务器要求验证客户端(浏览器)的证书(可选，大部分的服务器都不会要求)。
3. 客户端收到服务器证书之后，会检查证书的有效性，如果服务器证书并不是经过CA机构认证的，浏览器就会在这个时候给用户提出警告。
4. 客户端收到服务器验证客户端证书的请求，会将自己证书发送给服务器。客户端如果没有证书，则需要发送一个不包含证的证书消息。如果服务器需要客户端身份验证才能继续握手，则可能会使用致命的握手失败警报进行响应。(双向认证一般只存在于银行等一些安全性要求比较高的场景中，像早些时候我们使用的网银，里面存储的就是证书，用来在交易的时候和服务器端进行双向认证，保证安全)
5. 服务器收到客户端证书，校验客户端证书是否有效
6. 客户端把把上面流程中发送的所有消息(除了Client Hello)，使用客户端的私钥进行签名，然后发送给服务器。
7. 服务器也保存着之前客户端发送的消息，然后用客户端发送过来的公钥，进行验签。
8. 客户端生成一个Pre-Master-Secret随机数，然后使用服务器证书中的公钥加密，发送给服务器端。
9. 服务器端收到Pre-Master-Secret的加密数据，因为是使用它的公钥加密的，所以可以使用私钥解密得到Pre-Master-Secret。
10. 这时候客户端和服务端都同时知道了，RNc、RNs、Pre-Master-Secret这三个随机数，然后客户端和服务器端使用相同的PRF算法计算得到一个Master-Secret。然后可以从Master-Secret中再生成作为最终客户端和服务器端消息对称加密的秘钥，和对消息进行认证的MAC秘钥。

### 2.Pre-Master-Secret
Pre-Master-Secret前两个字节是TLS的版本号，这是一个比较重要的用来核对握手数据的版本号，因为在Client Hello阶段，客户端会发送一份加密套件列表和当前支持的SSL/TLS的版本号给服务端，而且是使用明文传送的，如果握手的数据包被破解之后，攻击者很有可能串改数据包，选择一个安全性较低的加密套件和版本给服务端，从而对数据进行破解。
所以，服务端需要对密文中解密出来对的Pre-Master-Secret中的版本号跟之前Client Hello阶段的版本号进行对比，如果版本号变低，则说明被篡改，则立即停止发送任何消息。

### 3.Master-Secret
客户端和服务端在生成Master-Secret的同时，还会生成下面6个秘钥，分别用于MAC算法和加解密算法。

|秘钥名称|秘钥作用|
| --- | --- |
|client write MAC key|客户端对发送数据进行MAC计算使用的秘钥，服务端使用同样的秘钥确认数据的完整性|
|server write MAC key|服务端对返回数据进行MAC计算使用的秘钥，客户端使用同一个秘钥验证完整性|
|client write key|对称加密key，客户端数据加密，服务端解密|
|server write key|服务端加密，客户端解密|
|client write IV|初始化向量，运用于分组对称加密|
|server write IV|初始化向量，运用于分组对称加密|

### 4.PRF算法

### 5.双向认证
其实我们日常访问的绝大多数网站，都是单向认证的(也就是说并没有上面流程中的3、4、5、6步骤)，这里为例展示HTTPS的完整交互流程，所以分析的是双向认证。
服务器可以选择是否要真正客户端的证书。这里以使用NGINX配置HTTPS为例，如果我们在NGINX的HTTPS相关配置中添加了下面这个配置，就表示需要验证客户端的证书。

```
ssl_verify_client on;
```

### 6.密码学套件

参考： [密码学套件](https://zhuanlan.zhihu.com/p/37239435)
## HTTPS中的算法
除了了解HTTPS的交互流程，HTTPS中使用的算法及算法的作用，也是我们必须要了解的一部分。
下面会按照算法的作用进行分类，并简单的介绍其中比较常见算法的作用，单并不会对算法的原理做过多的说明(主要是我也弄不明白)，会把相关的讲解原理的文章链接提供出来，大家有兴趣的可以自行去了解。

HTTPS中算法，根据算法的用途可以分为几大类
* 加密算法 - 加密传递的信息，包括对称加密和非对称加密
* 秘钥传递算法 - 在客户端和服务器端传递加密使用的key，当然通过上面的流程我们知道并不是直接传递加密的key
* 信息摘要/签名算法 - 对传递的信息摘要，确保信息在传递过程中不会被篡改

### 加密算法
加密算法基本可以分为两种 对称加密和非对称加密
#### 对称加密 
顾名思义就是加密和解密都是用一个同样的秘钥，它的优点就行加解密的速度很快，缺点就是尤其需要注意秘钥的传输，不能泄露。
包含的算法有 AES DES RC4等，常用的是AES

#### 非对称加密-RSA
非对称加密有一对秘钥公钥和私钥。使用公钥加密，然后使用私钥解密。公钥可以公开的发送给任何人。使用公钥加密的内容，只有私钥可以解开。安全性比对称加密大大提高。缺点是和对称加密相比速度较慢，加解密耗费的计算资源较多。

这里稍微说一下RSA算法
RSA算法可以说是非对称加密的代表算法了，算了还是不说了，先把其他的写完吧。

[RSA算法原理1](https://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
[RSA算法原理2](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)

### 秘钥传递算法
在HTTPS中比较常用的有RSA和DH算法。
对你没看错还是RSA，因为RSA非对称加密的特性，非常适合用来传递秘钥，而RSA在HTTPS中也正是承担的这一个关键作用。

DH算法流程
1. Alice 像 Bob 发送两个质数P和G。p是非常大的质数，g是生成元。
2. Alice生成一个随机数A。A是1~p-2之间的整数。这个数只能Alice知道。
3. Bob生成一个随机数B。B是一个1~p-2之间的整数。这个是只能Bob知道。
4. Alice将G^A mod P 发送给B
5. Bob将G^B mod P 发送给A
6. Alice用Bob发过来的数计算A次方，并求mod P （G^A*B mod P）
7. Bob用Alice发过来的数计算B次方并求mod P（G^A*B mod P）

DH算法和RSA把数据加密然后进行传输不同，它的算法过程更像是客户端和服务器端协商出来的一个秘钥。
看到DH的算法和RSA的算法，感觉都是运用了质数和模运算相关的数学原理和公式，他们之间应该也是有一定的关联和关系。(后悔没有好好学数学呀)

参考：[RSA和DH算法](https://blog.csdn.net/u013066244/article/details/79364011?utm_source=blogxgwz4)

### 摘要/签名算法
HTTPS中常见的有MD5、SHA256、MAC、HMAC等，下面主要说明一下这些算法的区别和联系。

#### 1. MD5
是一种消息摘要算法(Message-Digest Algorithm),一种被广泛使用的密码散列函数(Hash Function)，针对任意长度的输入，可以产生出一个定长128位（16字节）的散列值。

#### 2. SHA
SHA表示安全哈希算法(Secure Hash Algorithm)，经过了很长时间的发展SHA算法已经发展成了一个拥有众多算法的SHA家族。常见的有SHA0、SHA1、SHA2(包含SHA224、SHA256等)、SHA3。0，1，2，3表示SHA算法大的版本，每个大版本中又根据输出字节长度的不同分为和不同的算法。比如SHA256 使用的是SHA2，输出的是256字节。更详细的大家可以看下面wiki百科中的内容，很详细。

SHA称作安全哈希算法的原因是，它相比MD5算法，需要更多的计算次数，最终的输出长度也要长，(SHA0和SHA1是160字节。SHA256是256字节)。如果想要破解需要付出比MD5高的多的计算次数。

经过长时间的发展，MD5和SHA0、SHA1已经被认为不安全，已经不再建议使用。
SHA2是目前被最常使用的算法，目前还没有针对SHA2的有效攻击。
SHA3是2015年才发布的，还没有大规模的取代SHA2。

参考 [SHA算法家族](https://zh.wikipedia.org/wiki/SHA%E5%AE%B6%E6%97%8F)

#### 3. MAC 和 HMAC
相对于上面的MD5和SHA，这两种算法对于我算是比较陌生的。

MAC是消息认证码(Message Authentication Code)的简称。它和上面两种算法的区别是MAC的计算需要一个Key(上面HTTPS流程中就生了计算MAC的KEY)。只有知道了KEY才能正确的计算出对应的MAC。

HMAC的全称是密钥散列消息认证码(Keyed-hash message authentication code)。是指用秘钥并使用Hash散列算法生成认证码的一类算法的总称。

那么MAC算法和HMAC算法是什么关系呢？

我觉得可以这么理解。
MAC只是定义了一个概念---使用一个key，给一段消息生成一个授权码；但是生成这个授权码的算法它并没有定义。所以如果你使用SHA256这种Hash散列算法来生成授权码，那么这种算法就可以被称为HMAC-SHA256。
所以HMAC是MAC的一类实现方式，就像快排是排序算法中的一种实现方式一样。

参考：
[MAC-Wiki](https://zh.wikipedia.org/wiki/%E8%A8%8A%E6%81%AF%E9%91%91%E5%88%A5%E7%A2%BC)
[Difference between MAC and HMAC?](https://crypto.stackexchange.com/questions/6523/what-is-the-difference-between-mac-and-hmac)


#### 4. Salted Hash 和 HMAC
加盐Hash和HMAC在某种程度上很相似，但是在使用场景上还是有很大的区别。目前还有没找到解释的比较好的文章，后面再进行补充


# HTTPS证书
现在HTTPS基本已经成为了一个网站的标配。想要给一个网站添加对HTTPS的支持，就需要针对这个网站的域名申请证书。
## 如何获得HTTPS证书
简单来说获的HTTPS证书有两种方式
* 在有CA认证的机构申请
* 自己生成

### 1.通过CA机构申请

### 2.自己生成证书
参考 [自己生成HTTPS证书](https://www.barretlee.com/blog/2015/10/05/how-to-build-a-https-server/)

大家可以参考上面的文章自己创建一个证书试试。
自己创建的证书同样可以完成上面的步骤。只不过有一点就是，因为自己生成的证书没有得到CA机构的私钥签名，所以当浏览器通过HTTPS握手获得服务器证书的时候，没有办法确定这个证书是否就是来自请求的服务器。
这个时候浏览器就会给出该网站不安全的提示。

![不安全提示](http://upload-images.jianshu.io/upload_images/2829175-8749dcdef5960ff5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果客户端不能确认证书是安全，但是却贸然使用，就会有受到中间人攻击的风险。
#### 中间人攻击



![证书内容](http://upload-images.jianshu.io/upload_images/2829175-fcc431ce19db1379.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![证书内容](http://upload-images.jianshu.io/upload_images/2829175-d399116519dbda05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 客户端收到证书之后会首先会进行验证






 大家都知道要使用https，需要在网站的服务器上配置https证书（一般是nginx，或者tomcat），证书可以使用自己生成，也可以向专门的https证书提供商进行购买。这两种的区别是自己生成的证书是不被浏览器信任的,所以当访问的时候回提示不安全的网站，需要点击信任之后才能继续访问

![自己生成的](http://upload-images.jianshu.io/upload_images/2829175-8749dcdef5960ff5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而购买的https证书会提示安全
![DV,OV](http://upload-images.jianshu.io/upload_images/2829175-5c621e00597c6c00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![EV](http://upload-images.jianshu.io/upload_images/2829175-ef3241aa68eeb225.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是因为浏览器中预置了一些https证书提供商的证书，在浏览器获取到服务器的https证书进行验证的时候就知道这个https证书是可信的；而自己生成的证书，浏览器获取到之后无法进行验证是否可信，所以就给出不安全的提示。
下面对具体的一些知识点进行介绍
---



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