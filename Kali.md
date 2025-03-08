配置永久Ip

```shell
vim /etc/network/interfaces

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
iface eth0 inet static
address 172.20.3.53
network 255.255.255.0
gateway 172.168.1.1

# 重启网络服务
systemctl restart networking.service 
```

开机自启ssh服务

```shell
update-rc.d ssh enable

systemctl enable ssh
```

DNS域名解析

1. 正向查询

   ```shell
   nslookup www.baidu.com 
   dig www.baidu.com 
   ```

2. 反向查询

   ```shell
   dig -x www.baidu.com 
   ```

查询DNS服务器bind版本信息

> 查询DNS服务器bind版本信息目的：可以通过版本信息来查找相关版本漏洞的利用方式

```shell
dig txt chaos VERSION.BIND @ns3.dnsv4.com
```

查询网站的域名注册信息和备案信息

```shell
]┌──(root💀kali)-[/home/lucas/桌面]
└─# whois xuegod.cn  
Domain Name: xuegod.cn
ROID: 20140908s10001s72166376-cn
Domain Status: ok
Registrant: 北京学神科技有限公司
Registrant Contact Email: jianmingbasic@163.com
Sponsoring Registrar: 阿里云计算有限公司（万网）
Name Server: dns7.hichina.com
Name Server: dns8.hichina.com
Registration Time: 2014-09-08 10:52:31
Expiration Time: 2021-09-08 10:52:31
DNSSEC: unsigned
```

### 信息扫描工具

#### Maltego

> 利用Maltego进行DNS信息查询

#### Shodan

> 黑客搜索引擎（扫描服务器漏洞信息）

利用Shodan进行信息收集

1. city:beijing webcam

2. net:101.200.128.35

3. 搜索开放80端口的服务器

   port: 80

### 主动信息收集

#### 主动信息收集原理

##### 特点

1. 直接与目标系统进行通信
2. 无法避免留下访问痕迹
3. 使用受控的肉鸡访问或者使用代理机器进行访问
4. 扫描发送不同的探测，根据返回结果判断目标主机的状态

![image-20210503204913244](https://i.loli.net/2021/05/03/qezNClhFOpo7ycu.png)

##### 基于OSI模型扫描的优缺点

1. 数据链路层扫描

   优点：扫描速度快，可靠

   缺点：不可路由

2. 网络层扫描

   > 使用IP，ICMP协议

   优点：可路由，速度较快

   缺点：速度比数据链路层慢，经常被防火墙过滤

3. 传输层扫描

   优点：可路由且结果可靠，不太可能被防火墙过滤，可以发现所有端口被过滤的主机

   缺点：基于状态过滤的防火墙可能被过滤扫描，全端口扫描速度慢

#### 基于ping命令的探测

查看当前主机访问目标主机经过了哪些网络设备？

```shell
# 中间经过了12台网络设备

┌──(root💀kali)-[/etc/network]
└─# traceroute xuegod.cn                                                                        1 ⚙
traceroute to xuegod.cn (101.200.128.35), 30 hops max, 60 byte packets
 1  192.168.0.1 (192.168.0.1)  1.396 ms  1.272 ms  2.631 ms
 2  100.102.0.1 (100.102.0.1)  17.526 ms  19.233 ms  19.185 ms
 3  221.224.229.177 (221.224.229.177)  24.068 ms  29.031 ms  28.992 ms
 4  221.224.235.185 (221.224.235.185)  20.666 ms  20.626 ms  21.720 ms
 5  202.97.56.246 (202.97.56.246)  26.586 ms  26.907 ms 202.97.99.78 (202.97.99.78)  26.995 ms
 6  36.110.248.218 (36.110.248.218)  34.520 ms 36.110.248.210 (36.110.248.210)  37.521 ms 36.110.248.2 (36.110.248.2)  40.638 ms
 7  106.38.196.238 (106.38.196.238)  35.588 ms 36.110.169.102 (36.110.169.102)  35.842 ms 36.110.245.157 (36.110.245.157)  37.267 ms
 8  10.102.34.190 (10.102.34.190)  37.234 ms 106.38.196.234 (106.38.196.234)  35.524 ms 36.110.169.226 (36.110.169.226)  40.032 ms
 9  117.49.35.141 (117.49.35.141)  34.436 ms 101.200.109.133 (101.200.109.133)  35.731 ms 117.49.35.141 (117.49.35.141)  34.021 ms
10  10.102.234.177 (10.102.234.177)  35.662 ms 10.102.233.173 (10.102.233.173)  34.683 ms 10.102.233.181 (10.102.233.181)  40.105 ms
11  10.36.68.6 (10.36.68.6)  34.256 ms  34.739 ms 10.54.180.221 (10.54.180.221)  35.239 ms
12  10.36.68.6 (10.36.68.6)  34.600 ms 101.200.128.35 (101.200.128.35)  35.052 ms 10.36.72.14 (10.36.72.14)  33.018 ms
```

##### netdiscover 

> 嗅探该局域网内是否有数据包传输（被动方式）

```shell
netdiscover -i eth0  -r 192.168.1.0/24 

 Currently scanning: Finished!   |   Screen View: Unique Hosts                                      
                                                                                                    
 378 Captured ARP Req/Rep packets, from 8 hosts.   Total size: 22680                                
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.0.107   42:e9:71:65:19:7e     99    5940  Unknown vendor                                   
 192.168.0.101   b0:d5:9d:8e:96:64     15     900  Shenzhen Zowee Technology Co., Ltd               
 192.168.0.104   0c:8c:24:02:84:70     13     780  SHENZHEN BILIAN ELECTRONIC CO.，LTD             
 192.168.0.103   08:ea:40:ee:72:2b     14     840  SHENZHEN BILIAN ELECTRONIC CO.，LTD             
 192.168.0.102   54:48:e6:6d:e0:90      2     120  Beijing Xiaomi Mobile Software Co., Ltd          
 192.168.0.109   5c:e5:0c:3c:ed:35      2     120  Beijing Xiaomi Mobile Software Co., Ltd          
 192.168.0.108   0a:25:5d:b1:0a:c1    232   13920  Unknown vendor                                   
 192.168.0.1     34:96:72:7a:1f:6c      1      60  TP-LINK TECHNOLOGIES CO.,LTD. 
```

##### Hping3

Hping3是一个命令行下使用TCP/IP数据包组装/分析工具，通常web服务器会来用做压力测试使用，也可以用来做DDOS攻击实验。一个Hping命令只能扫描一个目标

```shell
hping3 -c 1000 -d 120 -S -w 64 -p 80 --flood --rand-source kuhantech.com 

# -c 1000 每次发送数据包的数量
# -d 120 发送到目标机器的每个数据包的大小（单位：byte）
# -S 只能发送SYN数据包
# -w TCp窗口大小
# -p 80 目的主机的端口号（80是web端口，这里你可以扫描任何端口号）
# -- flood 洪水攻击模式，尽可能快速发送数据包，不需要考虑显示入站恢复
# --rand-source 使用随机性源头IP地址 这里伪造的IP地址只是在局域网内伪造。通过路由器后，还会还原成真实的IP地址
```

##### Fping

Fping是ping命令的加强版，可以对一个IP端进行扫描，而ping命令本身是不可以对网段进行扫描

```shell
fping -g 192.168.1.0/24 -c 1 > fping.txt
```

#### 基于Nmap的扫描方式

1. nmap基本扫描方式

   ```shell
   nmap -sn 192.168.0.1-254
   # -sn 只做ping扫描，不做端口扫描（速度相对更快）
   
   ┌──(root💀kali)-[~/桌面]
   └─# nmap -sn 192.168.0.1-254                                                                                
   Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 09:29 CST
   Nmap scan report for 192.168.0.1
   Host is up (0.0011s latency).
   MAC Address: 34:96:72:7A:1F:6C (Tp-link Technologies)
   Nmap scan report for 192.168.0.101
   Host is up (0.11s latency).
   MAC Address: B0:D5:9D:8E:96:64 (Shenzhen Zowee Technology)
   Nmap scan report for 192.168.0.102
   Host is up (0.13s latency).
   MAC Address: 54:48:E6:6D:E0:90 (Beijing Xiaomi Mobile Software)
   Nmap scan report for 192.168.0.103
   Host is up (0.11s latency).
   MAC Address: 08:EA:40:EE:72:2B (Shenzhen Bilian Electronicltd)
   Nmap scan report for 192.168.0.135
   Host is up.
   Nmap done: 254 IP addresses (16 hosts up) scanned in 4.96 seconds
   ```

2. nmap半连接扫描

   nmap扫描类型主要有TCP全连接扫描（会在被扫描机器上留下记录）、半连接扫描（不会留下记录）

   ![image-20210504093835123](https://i.loli.net/2021/05/04/A7MR9umz1SnNxeL.png)

   ```shell
   ┌──(root💀kali)-[~/桌面]
   └─# nmap -sS 101.200.128.35 -p 80,81,21,25,22,110,443
   Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-04 09:45 CST
   Nmap scan report for 101.200.128.35
   Host is up (0.079s latency).
   
   PORT    STATE  SERVICE
   21/tcp  open   ftp
   22/tcp  closed ssh
   25/tcp  closed smtp
   80/tcp  open   http
   81/tcp  open   hosts2-ns
   110/tcp closed pop3
   443/tcp closed https
   
   Nmap done: 1 IP address (1 host up) scanned in 0.27 seconds
   ```


#### 使用scapy定制数据包进行高级扫描

```shell
>>> ARP().display() 
###[ ARP ]### 
  hwtype= 0x1        # 硬件类型
  ptype= IPv4        # 协议类型
  hwlen= None        # 硬件地址长度(MAC地址) 
  plen= None         # 协议地址长度(IP地址)
  op= who-has        # who-has查询
  hwsrc= 00:0c:29:6b:24:59  # 源MAC地址
  psrc= 192.168.0.135       # 源IP地址
  hwdst= 00:00:00:00:00:00
  pdst= 0.0.0.0      # 向谁发送查询请求
```

```shell
# sr1函数包含了发送数据包、接收数据包的功能
>>> sr1(ARP(pdst="192.168.0.1"))
Begin emission:
Finished sending 1 packets.
*
Received 1 packets, got 1 answers, remaining 0 packets
<ARP  hwtype=0x1 ptype=IPv4 hwlen=6 plen=4 op=is-at hwsrc=34:96:72:7a:1f:6c psrc=192.168.0.1 hwdst=00:0c:29:6b:24:59 pdst=192.168.0.135 |<Padding  load='\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00' |>>  

# hwsrc=34:96:72:7a:1f:6c (目的主机MAC地址)
# psrc=192.168.0.1        (目的主机IP地址)
# pdst=192.168.0.135      (源主机IP地址)
```

```shell
# 使用scapy定制ping包
```



#### 僵尸扫描

