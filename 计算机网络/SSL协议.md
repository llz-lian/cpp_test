# TLS和SSL
SSL（secure sockets layer）是由Netscape公司开发，用于安全通信。
TLS在SSLv3.0的基础上有IETF定义的协议，可在任何可靠传输层上提供机密性和身份验证。

DTLS（datagram TLS）与TLS目标相似，但是可以在不可靠的传输层协议下进行传输。

# TLS
TLS1.0的RFC文档:[RFC2246](https://datatracker.ietf.org/doc/html/rfc2246)

TLS也是分层的协议，由记录协议（record protocol），握手协议（handshake protocol）和警报协议（alert protocol）组成。record protocol提供加密和数据验证，alert protocol向其他协议提供信号，通知失败和错误。

handshake protocol负责安全参数的协商，初始密钥的交换。

---------------
handshake ,alert, application protocol

TLS record protocol

传输层

---------------

## 记录协议
用于加密和验证数据，握手过程完成后，需要接收和发送数据时，可以随时调用record layer函数。在DTLS中，设置一个重传定时器。
接收到的数据经过解密、验证、解压和重组交给上一层，传输的类型和长度不受TLS保护。

接收到无法理解的数据，TLS会丢弃这条数据。

## 警报协议
通知对等方协议失败的原因，包括warning 和 fatal等级。fatel直接关闭当前链接。

## 握手协议
负责密码套件协商，初始密钥交换和身份验证。完全由应用层控制。
密码套件在不同协议中略有不同，但支持绝大部分算法。
涉及一下步骤：
1. 交换问候消息，就算法达成一致，交换随机值。
2. 交换必要加密参数。
3. 交换证书和加密信息。
4. 通过随机值生成主密钥
5. 向记录层提供加密参数
```
Client                                               Server

      ClientHello                  -------->
                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data
```

# 在应用层使用TLS（未完成）
