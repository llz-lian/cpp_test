# DNS(Domain Name System)协议
DNS标准文档汇总[微软](https://learn.microsoft.com/zh-cn/windows/win32/dns/dns-standards-documents)。

应用层协议，
主要运行在UDP上，特定情况下可以TCP，端口53，
主要用于域名解析。

## DNS报文
主要存在两种报文，查询报文和回答报文，两种报文格式相同，通过标志位进行区分。
一个报文可以查询多个，一个查询可以有多个回答。

## 域名解析过程
1. 查看浏览器缓存
2. 查看操作系统缓存（windows hosts）
3. 本地域名服务器查询（通常递归查询）
4. 本地域名服务器迭代查询
5. 更新各自缓存

## gethostbyname
```c
 struct hostent *gethostbyname(const char *name);
```
name为域名，返回hostent结构体。
```c
#include<netdb.h>
struct hostent {
    char  *h_name;            /* official name of host */
    char **h_aliases;         /* alias list */
    int    h_addrtype;        /* host address type*/
    int    h_length;          /* length of address*/
    char **h_addr_list;       /* list of addresses*/
}
```