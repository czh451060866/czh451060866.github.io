---
title: 数字签名和证书
author: 阿呆
date: 2020-11-10
categories: 网络安全
tags: 数字签名和证书
---


1.信息安全三要素
信息安全中有三个需要解决的问题：

保密性(Confidentiality)：信息在传输时不被泄露
完整性（Integrity）：信息在传输时不被篡改
有效性（Availability）：信息的使用者是合法的
这三要素统称为CIA Triad。

公钥密码解决保密性问题
数字签名解决完整性问题和有效性问题

2.数字签名
现实生活中，签名有什么作用？在一封信中，文末的签名是为了表示这封信是签名者写的。计算机中，数字签名也是相同的含义：证明消息是某个特定的人，而不是随随便便一个人发送的（有效性）；除此之外，数字签名还能证明消息没有被篡改（完整性）。

简单来说，数字签名（digital signature）是公钥密码的逆应用：用私钥加密消息，用公钥解密消息。

用私钥加密的消息称为签名，只有拥有私钥的用户可以生成签名。
用公钥解密签名这一步称为验证签名，所有用户都可以验证签名(因为公钥是公开的)

一旦签名验证成功，根据公私钥数学上的对应关系，就可以知道该消息是唯一拥有私钥的用户发送的，而不是随便一个用户发送的。

由于私钥是唯一的，因此数字签名可以保证发送者事后不能抵赖对报文的签名。由此，消息的接收者可以通过数字签名，使第三方确信签名人的身份及发出消息的事实。当双方就消息发出与否及其内容出现争论时，数字签名就可成为一个有力的证据。

生成签名
一般来说，不直接对消息进行签名，而是对消息的哈希值进行签名，步骤如下。

对消息进行哈希计算，得到哈希值
利用私钥对哈希值进行加密，生成签名
将签名附加在消息后面，一起发送过去
验证签名
收到消息后，提取消息中的签名
用公钥对签名进行解密，得到哈希值1。
对消息中的正文进行哈希计算，得到哈希值2。
比较哈希值1和哈希值2，如果相同，则验证成功。
3.证书
证书实际上就是对公钥进行数字签名，它是对公钥合法性提供证明的技术。

考虑这样一种场景：我们对签名进行验证时，需要用到公钥。如果公钥也是伪造的，那怎么办？如果公钥是假的，验证数字签名那就无从谈起，根本不可能从数字签名确定对方的合法性。
这时候证书就派上用场了。

证书一般包含：公钥（记住证书中是带有公钥的），公钥的数字签名，公钥拥有者的信息
若证书验证成功，这表示该公钥是合法，可信的。

接下来又有问题了：验证证书中的数字签名需要另一个公钥，那么这个公钥的合法性又该如何保证？该问题可以无限循环下去，岂不是到不了头了？这已经是个社会学问题了。我们为什么把钱存进银行？因为我们相信银行，它是一个可信的机构（虽然也有破产的风险）。跟银行一样，我们需要一个可信的机构来颁发证书和提供公钥，只要是它提供的公钥，我们就相信是合法的。

这种机构称为认证机构(Certification Authority， CA)。CA就是能够认定”公钥确实属于此人”，并能生成公钥的数字签名的组织或机构。CA有国际性组织和政府设立的组织，也有通过提供认证服务来盈利的组织。

如何生成证书？
服务器将公钥A给CA（公钥是服务器的）
CA用自己的私钥B给公钥A加密，生成数字签名A
CA把公钥A，数字签名A，附加一些服务器信息整合在一起，生成证书，发回给服务器。
注：私钥B是用于加密公钥A的，私钥B和公钥A并不是配对的。

如何验证证书？
客户端得到证书
客户端得到证书的公钥B（通过CA或其它途径）
客户端用公钥B对证书中的数字签名解密，得到哈希值
客户端对公钥进行哈希值计算
两个哈希值对比，如果相同，则证书合法。
注：公钥B和上述的私钥B是配对的，分别用于对证书的验证（解密）和生成（加密）。

证书作废
当用户私钥丢失、被盗时，认证机构需要对证书进行作废(revoke)。要作废证书，认证机构需要制作一张证书作废清单(Certificate Revocation List)，简称CRL

假设我们有Bob的证书，该证书有合法的认证机构签名，而且在有效期内，但仅凭这些还不能说明该证书一定有效，还需要查询认证机构最新的CRL，并确认该证书是否有效。

使用场景
下面用两个使用场景来帮助大家理解证书的作用。

客户端在发送或接收消息之前，要验证服务器的合法性(这个服务器是真实的服务器，还是伪造者，我们不知道)

场景1

服务器生成公钥和私钥密码对
服务器把公钥给CA。CA生成证书，发送给客户端
客户端验证证书，取得公钥：此刻证明公钥是合法的
客户端用公钥加密消息，发送给服务器
服务器用私钥解密消息（消息加密发送，具有保密性）
场景2

服务器生成公钥和私钥密码对
服务器生成消息，用私钥对消息进行数字签名
服务器把公钥给CA。CA生成证书
服务器将消息，数字签名，证书一起发送给客户端
客户端验证证书，取得公钥：此刻证明公钥是合法的
客户端用公钥验证数字签名，检查消息的完整性和服务器的合法性