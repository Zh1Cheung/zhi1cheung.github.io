---
title: HA Cluster实现方案
categories:
- Cluster
tags:
- Cluster 
- HA


---
1、vrrp协议的实现

        keepalived
2、ais（available Interface standard）：可用接口标准，完备HA集群

        RHCS（cmam）
        heartbeat
        corosync
        
        Keepalived：
        vrrp协议：Virtual Redundant Protocol
        术语:
        虚拟路由：virtual router
        虚拟路由器标识：VRID（0-255）
        物理路由：
        master：主设备
        back：备用设备
        priority：优先级
        VIP：virtual Ip
        VMAC:Virtual MAC (00-00-5e-00-01-VRID)
        GraciousARP(免费arp)
        通告：心跳，优先级等；周期性；
        抢占式，非抢占式；
        安全工作：
           认证：
        无认证
        简单字符认证
        MD5
        工作模式
        主/备：单虚拟路由器；
        主/主：主/备（虚拟路由器1），备/主（虚拟路径器2）
        特点：
        vrrp协议的软件实现，原生设计的目的为了高可用ipvs服务：
        vrrp协议完成地址流动；
        为vip地址所在的节点生成ipvs规则（在配置文件中预先定义）；
        为ipvs集群的各RS做健康状态检测；
        基于脚本调用接口通过执行脚本完成脚本中定义的功能，进而影响集群事务；
        组件：
        核心组件：
        vrrp stack
        ipvs wrapper
        checkers
        控制组件：配置文件分析器
        IO复用器
        内存管理组件
        HA Cluster的配置前提：
        （1）各节点时间必须同步
        （2）确保iptables及selinux不会成为阻碍；
        （3）各节点之间可通过主机名互相通信(对KA并非必须)；
        建议使用/etc/hosts文件实现；
        （4）各节点之间的root用户可以基于密钥认证的ssh服务完成互相通信（并非必须）
    
    
####     keepalived安装配置：
        CentOS 6.4+随base仓库提供；
1、同步时间

        配置chronyd服务器172.18。200.100
        yum安装chrony，并启动服务
        [root@localhost ~]# service chronyd start
        Starting chronyd:                                          [  OK  ]
        
        使用ntpdate命令，同步172.18.10.10以及172.18.10.11的时间
        [root@localhost ~]# ntpdate 172.18.200.100
2、清空iptables和selinux

        iptables -F
        setenforce 0
3、配置hosts文件（非必须）
4、安装keepalived
        [root@localhost ~]# yum install keepalived
        [root@localhost ~]# cd /etc/keepalived/
        [root@localhost keepalived]# ls
        keepalived.conf
        [root@localhost keepalived]# cp keepalived.conf keepalived.conf.bak
        [root@localhost keepalived]# ls
        keepalived.conf  keepalived.conf.bak
        [root@localhost keepalived]# vim keepalived.conf
        主配置文件：/etc/keepalived/keepalived.conf
        配置文件组成部分及相关选项解释
        TOP HIERACHY
        GLOBAL CONFIGURATION
        Global definitions
        Static routes/addresses
        VRRPD CONFIGURATION
        VRRP synchronization group(s)：vrrp同步组；
        VRRP instance(s)：每个vrrp instance即一个vrrp路由器；
        LVS CONFIGURATION
        Virtual server group(s)
        Virtual server(s)：ipvs集群的vs和rs；
        global_defs {    ###全局定义
           notification_email {
             acassen@firewall.loc
             failover@firewall.loc  ####定义出现问题后发送邮箱的地址
             sysadmin@firewall.loc
           }
           notification_email_from Alexandre.Cassen@firewall.loc   ##从哪里发过来
           smtp_server 192.168.200.1  ###邮件服务器地址
           smtp_connect_timeout 30#####超时时间
           router_id LVS_DEVEL###路由器IP
           vrrp_mcast_group4  224.0.100.5###ipv4多播地址
        
        }
        
        vrrp_instance VI_1 {  ##vrrp配置段
            state MASTER###表示是主还是从这里显示主，另一个则为从
            interface eth0###表明工作从哪个网卡发出 “多波心跳信息”
            virtual_router_id 51###虚拟路由ID
            priority 100###主的优先级
            advert_int 1       ##通告时间间隔
            authentication###认证
                auth_type PASS####认证类型：简单密钥认证
                auth_pass 1111#####认证密码：最多不能超过8位
            }
            virtual_ipaddress {##虚拟IP地址配在哪个网卡上
                192.168.200.16/24 dev eth0   ##定义配置在哪个网卡的别名上
                192.168.200.17
                192.168.200.18
            }
        }
        track_interface {   ##配置要监控的网络接口，一旦接口出现故障，则转为FAULT状态；即接口跟踪
        eth0
        eth1
        ...
        }
        nopreempt：定义工作模式为非抢占模式；
        preempt_delay 300：抢占式模式下，节点上线后触发新选举操作的延迟时长；
5、修改配置文件

        [root@localhost keepalived]# vim keepalived.conf
        global_defs {
           notification_email {
                root@localhost
           }
           notification_email_from keepalived@localhost
           smtp_server 127.0.0.1
           smtp_connect_timeout 30
           router_id node1
           vrrp_mcast_group4  224.0.100.50
        }
        
        vrrp_instance myroute {
            state MASTER
            interface eth1
            virtual_router_id 50
            priority 100
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123456
            }
            virtual_ipaddress {
                172.18.50.50/16 dev eth1
            }
        }
6、将配置文件发送到另一台机器10上
        
        [root@localhost keepalived]# scp keepalived.conf 172.18.10.10:/etc/keepalived/
        修改配置文件
        [root@localhost keepalived]# vim keepalived.conf
        global_defs {
           notification_email {
                root@localhost
           }
           notification_email_from keepalived@localhost
           smtp_server 127.0.0.1
           smtp_connect_timeout 30
           router_id node2
           vrrp_mcast_group4  224.0.100.50
        }
        
        vrrp_instance myroute {
            state BACKUP
            interface eth1
            virtual_router_id 50
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123456
            }
            virtual_ipaddress {
                172.18.50.50/16 dev eth1
            }
        }
        
7、启动服务

        启动备用服务器11
        [root@localhost ~]# service keepalived start
        查看地址
        [root@localhost ~]# ip a
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:07:27:ff brd ff:ff:ff:ff:ff:ff
            inet 172.18.10.10/16 brd 172.18.255.255 scope global eth1
            inet 172.18.50.50/16 scope global secondary eth1
            inet6 fe80::20c:29ff:fe07:27ff/64 scope link 
               valid_lft forever preferred_lft forever
        发现地址已经添加，这是若开启主服务器，由于没有设置抢断延迟，则会立刻抢断
        
8、启动主服务器

        [root@localhost keepalived]# service keepalived start
        Starting keepalived:                                       [  OK  ]
        [root@localhost keepalived]# ip a
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:99:76:84 brd ff:ff:ff:ff:ff:ff
            inet 172.18.10.11/16 brd 172.18.255.255 scope global eth1
            inet 172.18.50.50/16 scope global secondary eth1
            inet6 fe80::20c:29ff:fe99:7684/64 scope link 
               valid_lft forever preferred_lft forever
        发现地址已经添加
        而从服务器11上
        [root@localhost ~]# ip a
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:07:27:ff brd ff:ff:ff:ff:ff:ff
            inet 172.18.10.10/16 brd 172.18.255.255 scope global eth1
            inet6 fe80::20c:29ff:fe07:27ff/64 scope link 
               valid_lft forever preferred_lft forever
        IP地址已经删除
        
9、使用tcpdump抓包工具查看主从服务器的相应心跳测试
        [root@localhost keepalived]# tcpdump -i eth1 host 224.0.100.50   ###在主服务器端抓包
        tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
        listening on eth1, link-type EN10MB (Ethernet), capture size 65535 bytes
        16:39:33.357307 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:34.358905 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:35.360605 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:36.362301 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:37.363904 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:38.365658 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:39.367266 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:40.368921 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:41.370599 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        [root@localhost ~]#  tcpdump -i  eth1  -nn host 224.0.100.50   ###在从服务器端抓包
        tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
        listening on eth1, link-type EN10MB (Ethernet), capture size 65535 bytes
        16:39:40.367044 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:41.368741 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:42.370289 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:43.371983 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:44.373750 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:45.375413 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:46.377092 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        16:39:47.378760 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        分析说明。实现简单的vrrp
        即从服务器每隔一秒向主服务器发送1个通报报文。探测主服务器是否存活，实现具体实施软件keepalived
        ###############################################################################################################################
        
        
        
###     双主模型
    
1、我们在172.18.10.11上配置了主服务器配置，双主服务可在配置文件下面继续添如下内容，配置如下

        [root@localhost keepalived]# vim keepalived.conf
        vrrp_instance myroute2 {
            state BACKUP
            interface eth1
            virtual_router_id 51
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123457
            }
            virtual_ipaddress {
                172.18.51.51/16 dev eth1
            }
        }
2、将内容服务至粘贴至172.18.10.10服务器的keepalived.conf配置文件中,然后需要在state和priority上进行相应修改

        vrrp_instance myroute2 {
            state MASTER
            interface eth1
            virtual_router_id 51
            priority 100
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123457
            }
            virtual_ipaddress {
                172.18.51.51/16 dev eth1
            }
        }
        保存并退出，实现双主模型的设置
        
3、从启服务并测试

        service keepalived restart
        Stopping keepalived:                                       [  OK  ]
        Starting keepalived:                                       [  OK  ]
        使用tcpdump抓包，结果如下
        172.18.10.11端
        [root@localhost keepalived]#  tcpdump -i  eth1  -nn host 224.0.100.50
        tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
        listening on eth1, link-type EN10MB (Ethernet), capture size 65535 bytes
        00:50:20.150330 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:20.521639 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:50:21.151175 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:21.522539 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:50:22.152517 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:22.523232 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:50:23.154334 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:23.524046 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        172.18.10.10端
        [root@localhost keepalived]# tcpdump -i eth1 host 224.0.100.50
        tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
        listening on eth1, link-type EN10MB (Ethernet), capture size 65535 bytes
        00:54:01.436075 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:54:01.437266 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:54:02.437295 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:54:02.438831 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:54:03.438695 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:54:03.439205 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        
        分析每次都会收到两次信息，一次发送，一次接收
        使用iptable设置规则，拒绝172.18.10.11向224.0.100.50发送通知报文
        [root@localhost keepalived]# iptables -A OUTPUT -s 172.18.10.11 -d 224.0.100.50 -j REJECT
        在172.18.10.10端使用tcpdump抓包
        [root@localhost keepalived]#  tcpdump -i  eth1  -nn host 224.0.100.50
        tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
        listening on eth1, link-type EN10MB (Ethernet), capture size 65535 bytes
        00:50:20.150330 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:20.521639 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:50:21.151175 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:21.522539 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        00:50:22.152517 IP 172.18.10.10 > 224.0.100.50: VRRPv2, Advertisement, vrid 51, prio 100, authtype simple, intvl 1s, length 20
        00:50:22.523232 IP 172.18.10.11 > 224.0.100.50: VRRPv2, Advertisement, vrid 50, prio 100, authtype simple, intvl 1s, length 20
        分析发现发送两条通告，因为172.18.10.11不通告，便认为172.18.10.11挂掉了，因此抢断，让自己变为主机。即别人不通告则认为对方挂掉了
        可以使用ip a l 查看相应的ip地址获取：
        [root@localhost keepalived]# ip a l
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:07:27:ff brd ff:ff:ff:ff:ff:ff
            inet 172.18.10.10/16 brd 172.18.255.255 scope global eth1
            inet 172.18.51.51/16 scope global secondary eth1
            inet 172.18.50.50/16 scope global secondary eth1
            inet6 fe80::20c:29ff:fe07:27ff/64 scope link 
               valid_lft forever preferred_lft forever
        再次在172.18.10.11服务器上，清空iptables规则
        [root@localhost keepalived]# iptables -F
        再回到172.18.10.10服务器上使用ip a l 查询
        [root@localhost keepalived]# ip a l
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:07:27:ff brd ff:ff:ff:ff:ff:ff
            inet 172.18.10.10/16 brd 172.18.255.255 scope global eth1
            inet 172.18.51.51/16 scope global secondary eth1
            inet6 fe80::20c:29ff:fe07:27ff/64 scope link 
               valid_lft forever preferred_lft forever
        发现地址已经立马被夺回，是因为工作在抢占模式下。没有设置preempt_delay 300抢占延迟时间，
        结论：实现双主模型实验
    
        ##################################################################################################################
        
####     如何实现自定义通知脚本
    
一、在172.18.10.11服务器上添加脚本，实现自动发邮件

        1.编写邮件脚本
        vim notify.sh
        #!/bin/bash
        #
        contact='root@localhost'
        
        notify() {
                mailsubject="vrrp： $(hostname) to be $1"
                mailbody="$(hostname) to be $1,vrrp transition, $(date)."
                echo "$mailbody" | mail -s "$mailsubject" $contact
        
        }
        
        case $1 in
        master)
                notify master ;;
        backup)
                notify backup ;;
        fault)
                notify fault ;;
        *)
                echo "Usage: $(basename $0 ) master|backup|fault"
                exit 1
                ;;
        esac
              
        2、测试脚本
        语法检测
        [root@localhost keepalived]# bash -n notify.sh
        运行脚本测试
        [root@localhost keepalived]# bash -x notify.sh master
        + contact=root@localhost
        + case $1 in
        + notify master
        ++ hostname
        + mailsubject='localhost.localdomain to be master'
        ++ hostname
        ++ date
        + mailbody='localhost.localdomain to be master,vrrp transition, Mon May 15 01:36:33 CST 2017.'
        + echo 'localhost.localdomain to be master,vrrp transition, Mon May 15 01:36:33 CST 2017.'
        + mail -s mailsubject root@localhost
        [root@localhost keepalived]# vim notify.sh
        You have mail in /var/spool/mail/root
        
        3、查看收到的邮件
        
        [root@localhost keepalived]# mail
        Heirloom Mail version 12.4 7/29/08.  Type ? for help.
        "/var/spool/mail/root": 1 message 1 new
        >N  1 root                  Mon May 15 01:36  18/696   "mailsubject"
        &     
        Message  1:
        From root@localhost.localdomain  Mon May 15 01:36:34 2017
        Return-Path: <root@localhost.localdomain>
        X-Original-To: root@localhost
        Delivered-To: root@localhost.localdomain
        Date: Mon, 15 May 2017 01:36:33 +0800
        To: root@localhost.localdomain
        Subject: mailsubject
        User-Agent: Heirloom mailx 12.4 7/29/08
        Content-Type: text/plain; charset=us-ascii
        From: root@localhost.localdomain (root)
        Status: R
        
        localhost.localdomain to be master,vrrp transition, Mon May 15 01:36:33 CST 2017.
        
        & 
        4、将脚本发送至172.18.10.10端
        
        [root@localhost keepalived]# scp notify.sh 172.18.10.10:/etc/keepalived/
        root@172.18.10.10's password: 
        notify.sh                                                                       100%  367     0.4KB/s   00:00
        
        5、调用脚本
        
        [root@localhost keepalived]# vim keepalived.conf
        在172.18.10.11上的vrrp_instance myrouter1下面添加如下内容,注意是放在vrrp_instance myrouter1上下文中调用
        
                notify_master "/etc/keepalived/notify.sh master"
                notify_backup "/etc/keepalived/notify.sh backup"
                notify_fault "/etc/keepalived/notify.sh fault"
        在172.18.10.10上的vrrp_instance myrouter2下面添加如下内容
        
                notify_master "/etc/keepalived/notify.sh master"
                notify_backup "/etc/keepalived/notify.sh backup"
                notify_fault "/etc/keepalived/notify.sh fault"
        
        6，为了实现测试效果，将之前定义的双主模型删除，并停止服务（在10.10和10.11上做同样的操作）
        ：.,$d  表示从当前行都最后一行全部删除
        
        [root@localhost keepalived]# service keepalived stop
        Stopping keepalived:                                       [  OK  ]
        
        7、给之前编写的脚本加上执行权限
        [root@localhost keepalived]# chmod +x  notify.sh 
        [root@localhost keepalived]# ll
        total 8
        -rw-r--r-- 1 root root 658 May 15 02:01 keepalived.conf
        -rwxr-xr-x 1 root root 367 May 15 01:41 notify.sh
        
        8、启动服务
        在172.18.10.11端
        [root@localhost keepalived]# service keepalived start
        Starting keepalived:                                       [  OK  ]
        [root@localhost keepalived]# ip a l
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:99:76:84 brd ff:ff:ff:ff:ff:ff
            inet 172.18.10.11/16 brd 172.18.255.255 scope global eth1
            inet 172.18.50.50/16 scope global secondary eth1
            inet6 fe80::20c:29ff:fe99:7684/64 scope link 
               valid_lft forever preferred_lft forever
        [root@localhost keepalived]# mail
        Heirloom Mail version 12.4 7/29/08.  Type ? for help.
        "/var/spool/mail/root": 3 messages 2 unread
            1 root                  Mon May 15 01:36  19/707   "mailsubject"
        >U  2 root                  Mon May 15 11:03  19/735   "vrrp: localhost.localdomain to be master"
        & 
        9、启动172.18.10.10端的keepalived，并且再次到172.18.10.11端查看邮件
        [root@localhost ~]# mail
        Heirloom Mail version 12.4 7/29/08.  Type ? for help.
        "/var/spool/mail/root": 7 messages 5 new 7 unread
         U  1 root                  Mon May 15 11:09  19/735   "vrrp: localhost.localdomain to be backup"
         U  2 root                  Mon May 15 11:11  19/735   "vrrp: localhost.localdomain to be backup"
        >N  3 root                  Mon May 15 11:11  18/725   "vrrp: localhost.localdomain to be master"
         N  4 root                  Mon May 15 11:11  18/725   "vrrp: localhost.localdomain to be backup"
         N  5 root                  Mon May 15 11:26  18/725   "vrrp: localhost.localdomain to be backup"
         N  6 root                  Mon May 15 11:26  18/725   "vrrp: localhost.localdomain to be master"
         N  7 root                  Mon May 15 11:26  18/725   "vrrp: localhost.localdomain to be backup"
        & 
        结论：通知脚本功能实现
        ######################################################################################################
    
####     如何实现 keepalived 高可用LVS （重点）
        
        实验准备：4台虚拟主机
        其中172.18.10.10和172.18.10.11做为VS端分别为VS2和VS1
        172.18.200.100和172.18.249.57做为RS分别为RS1和RS2
        首先分别再RS1和RS2端安装httpd
        
        1、进行如下配置
        
        [root@localhost ~]# cat /var/www/html/index.html
        <h1>RS1:172.18.200.100</h1>
        [root@localhost ~]# cat /var/www/html/index.html 
        <h1>RS2:172.18.249.57</h1>
        
        2、编写VIP配置脚本
        vim setparam.sh
        #!/bin/bash
        #
        vip='172.18.50.50'
        netmask='255.255.255.255'
        iface='lo:0'
        
        case $1 in
        start)
                echo 1 > /pro/sys/net/ipv4/conf/all/arp_ignore
                echo 1 > /pro/sys/net/ipv4/conf/lo/arp_ignore
                echo 2 > /pro/sys/net/ipv4/conf/all/arp_ignore
                echo 2 > /pro/sys/net/ipv4/conf/lo/arp_ignore
                ifconfig $iface $vip netmask $netmask broadcast $vip up
                route add -host $vip dev $iface
                ;;
        stop)
                ifconfig $iface down
                echo 0 > /pro/sys/net/ipv4/conf/all/arp_ignore
                echo 0 > /pro/sys/net/ipv4/conf/lo/arp_ignore
                echo 0 > /pro/sys/net/ipv4/conf/all/arp_ignore
                echo 0 > /pro/sys/net/ipv4/conf/lo/arp_ignore
                ;;
        esac
        
        3、测试脚本
        [root@localhost ~]# bash -n setparam.sh 
        [root@localhost ~]# bash -x setparam.sh start
        + vip=172.18.50.50
        + netmask=255.255.255.255
        + iface=lo:0
        + case $1 in
        + echo 1
        setparam.sh: line 9: /pro/sys/net/ipv4/conf/all/arp_ignore: No such file or directory
        + echo 1
        setparam.sh: line 10: /pro/sys/net/ipv4/conf/lo/arp_ignore: No such file or directory
        + echo 2
        setparam.sh: line 11: /pro/sys/net/ipv4/conf/all/arp_announce: No such file or directory
        + echo 2
        setparam.sh: line 12: /pro/sys/net/ipv4/conf/lo/arp_announce: No such file or directory
        + ifconfig lo:0 172.18.50.50 netmask 255.255.255.255 broadcast 172.18.50.50 up
        + route add -host 172.18.50.50 dev lo:0
        
        4、使用scp将脚本分发至RS2
        [root@localhost ~]# scp setparam.sh 172.18.249.57:/root
        root@172.18.249.57's password: 
        setparam.sh                                                                                  100%  610     0.6KB/s   00:00 
        
        5、在RS2端执行脚本，并查看是否生成VIP
        [root@localhost ~]# bash -x setparam.sh start
        + vip=172.18.50.50
        + netmask=255.255.255.255
        + iface=lo:0
        + case $1 in
        + echo 1
        setparam.sh: line 9: /pro/sys/net/ipv4/conf/all/arp_ignore: No such file or directory
        + echo 1
        setparam.sh: line 10: /pro/sys/net/ipv4/conf/lo/arp_ignore: No such file or directory
        + echo 2
        setparam.sh: line 11: /pro/sys/net/ipv4/conf/all/arp_announce: No such file or directory
        + echo 2
        setparam.sh: line 12: /pro/sys/net/ipv4/conf/lo/arp_announce: No such file or directory
        + ifconfig lo:0 172.18.50.50 netmask 255.255.255.255 broadcast 172.18.50.50 up
        + route add -host 172.18.50.50 dev lo:0
        [root@localhost ~]# ip a
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
            inet 127.0.0.1/8 scope host lo
            inet 172.18.50.50/32 brd 172.18.50.50 scope global lo:0
            inet6 ::1/128 scope host 
               valid_lft forever preferred_lft forever
        2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
            link/ether 00:0c:29:b2:ca:ea brd ff:ff:ff:ff:ff:ff
            inet 172.18.249.57/16 brd 172.18.255.255 scope global eth0
            inet6 fe80::20c:29ff:feb2:caea/64 scope link 
               valid_lft forever preferred_lft forever
        
        6、启动RS1和RS2的httpd服务，并查看端口，两端都要查看，这里只演示一端的
        [root@localhost ~]# service httpd start
        [root@localhost ~]# ss -tnl
        State       Recv-Q Send-Q                               Local Address:Port                                 Peer Address:Port 
        LISTEN      0      128                                             :::80                                             :::*     
        LISTEN      0      128                                             :::22                                             :::*     
        LISTEN      0      128                                              *:22                                              *:*     
        LISTEN      0      100                                            ::1:25                                             :::*     
        LISTEN      0      100                                      127.0.0.1:25       
        
        7、在两个前段节点生成ipvs规则
        在VS2端
        停止keepalived服务
        配置sorry server页面
        vim /var/www/html/index.html
        Director2 sorry server2
        
        启动httpd服务
        [root@localhost ~]# service httpd start
        
        在VS1端
        首先停止keepalived服务
        [root@localhost ~]#  service keepalived stop  
        vim /var/www/html/index.html
        Director1
        
        启动httpd服务
        [root@localhost ~]# service httpd start
        
        在VS1端编辑keepalived配置文件，添加如下内容：
        virtual_server 172.18.50.50 80 {
            delay_loop 6
            lb_algo wrr
            lb_kind DR
            persistence_timeout 0
            protocol TCP
        
            real_server 172.18.10.11 80 {
                weight 1
                HTTP_GET {
            persistence_timeout 0
            protocol TCP
        sorry_server 127.0.0.1 80
        
            real_server 172.18.10.11 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
            real_server 172.18.10.10 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
        }
        
        在VS2端，同样编辑keepalived.conf文件，添加如下内容
        virtual_server 172.18.50.50 80 {
            delay_loop 6
            lb_algo wrr
            lb_kind DR
            persistence_timeout 0
            protocol TCP
        sorry_server 127.0.0.1 80
            real_server 172.18.10.11 80 {
                weight 1
                HTTP_GET {
            persistence_timeout 0
            protocol TCP
        
            real_server 172.18.10.11 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
            real_server 172.18.10.10 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
        }
        在VS2上启动keepalived服务
        [root@localhost ~]# service keepalived start
        Starting keepalived:                                       [  OK  ]
        [root@localhost ~]# ipvsadm -ln
        IP Virtual Server version 1.2.1 (size=4096)
        Prot LocalAddress:Port Scheduler Flags
          -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
        TCP  172.18.50.50:80 wrr
          -> 172.18.200.100:80            Route   1      0          0         
          -> 172.18.249.57:80             Route   1      0          0   
        
        在客户端使用curl进行访问测试（配置完有一定延迟，稍等片刻在访问）
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS2:172.18.249.57</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS1:172.18.200.100</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS2:172.18.249.57</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS1:172.18.200.100</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS2:172.18.249.57</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS1:172.18.200.100</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS2:172.18.249.57</h1>
        [root@localhost ~]# curl http://172.18.50.50
        <h1>RS1:172.18.200.100</h1>
        
        在172.18.200.100端停止httpd服务
        [root@localhost ~]# service httpd stop
        Stopping httpd:                                            [  OK  ]
        
        在VS2端使用ipvsadm观察
        [root@localhost keepalived]# ipvsadm -ln
        IP Virtual Server version 1.2.1 (size=4096)
        Prot LocalAddress:Port Scheduler Flags
          -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
        TCP  172.18.50.50:80 wrr
          -> 172.18.249.57:80             Route   1      0          2  
        
        在172.18.200.100端停止httpd服务
        [root@localhost ~]# service httpd start
        
        在VS2端使用ipvsadm观察
        [root@localhost keepalived]# ipvsadm -ln
        IP Virtual Server version 1.2.1 (size=4096)
        Prot LocalAddress:Port Scheduler Flags
          -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
        TCP  172.18.50.50:80 wrr
          -> 172.18.200.100:80            Route   1      0          0         
          -> 172.18.249.57:80             Route   1      0          0    
        
        启动VS1上的keepalived服务，并且关闭VS2，客户端使用curl测试发现，仍然能够访问
        [root@localhost keepalived]# curl http://172.18.50.50
        <h1>RS2:172.18.249.57</h1>
        [root@localhost keepalived]# curl http://172.18.50.50
        <h1>RS1:172.18.200.100</h1>
        [root@localhost keepalived]# curl http://172.18.50.50
        <h1>RS2:172.18.249.57</h1>
        [root@localhost keepalived]# curl http://172.18.50.50
        <h1>RS1:172.18.200.100</h1>
        
        更改配置文件，将之前删除的双主内容添加进去
        VS1端
        vrrp_instance myroute2 {
            state BACKUP
            interface eth1
            virtual_router_id 51
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123457
            }
            virtual_ipaddress {
        172.18.51.51/16 dev eth1
            }
        }
        VS2端
        vrrp_instance myroute2 {
            state MASTER
            interface eth1
            virtual_router_id 51
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123457
            }
            virtual_ipaddress {
        172.18.51.51/16 dev eth1
            }
        }
        
        
        重启keepalived服务,相当重要，，，，，不重启不会有效果，这就是个坑
        
        
        
        总结VS端
        VS2端keepalived配置
        ! Configuration File for keepalived
        
        global_defs {
           notification_email {
        root@localhost
           }
           notification_email_from keepalived@localhost
           smtp_server 127.0.0.1
           smtp_connect_timeout 30
           router_id node2
           vrrp_mcast_group4  224.0.100.50
        }
        
        vrrp_instance myroute1 {
            state BACKUP
            interface eth1
            virtual_router_id 50
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123456
            }
            virtual_ipaddress {
        172.18.50.50/16 dev eth1
            }
        
        notify_master "/etc/keepalived/notify.sh master"
        notify_backup "/etc/keepalived/notify.sh backup"
        notify_fault "/etc/keepalived/notify.sh fault"
        }
        
        vrrp_instance myroute2 {
            state MASTER
            interface eth1
            virtual_router_id 51
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123457
            }
            virtual_ipaddress {
        172.18.51.51/16 dev eth1
            }
        }
        
        virtual_server 172.18.50.50 80 {
            delay_loop 6
            lb_algo wrr
            lb_kind DR
            persistence_timeout 0
            protocol TCP
            sorry_server 127.0.0.1 80
        
            real_server 172.18.200.100 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
            real_server 172.18.249.57 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
        }
        
        virtual_server 172.18.51.51 80 {
            delay_loop 6
            lb_algo wrr
            lb_kind DR
            persistence_timeout 0
            protocol TCP
            sorry_server 127.0.0.1 80
        
            real_server 172.18.200.100 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
            real_server 172.18.249.57 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
                        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
        }
        
        VS1端keepalived配置
        ! Configuration File for keepalived
        
        global_defs {
           notification_email {
        root@localhost
           }
           notification_email_from keepalived@localhost
           smtp_server 127.0.0.1
           smtp_connect_timeout 30
           router_id node1
           vrrp_mcast_group4  224.0.100.50
        }
        
        vrrp_instance myroute1 {
            state MASTER
            interface eth1
            virtual_router_id 50
            priority 100
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123456
            }
            virtual_ipaddress {
        172.18.50.50/16 dev eth1
            }
        
        notify_master "/etc/keepalived/notify.sh master"
                notify_backup "/etc/keepalived/notify.sh backup"
                notify_fault "/etc/keepalived/notify.sh fault"
        }
        
        vrrp_instance myroute2 {
            state BACKUP
            interface eth1
            virtual_router_id 51
            priority 98
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass 123457
            }
            virtual_ipaddress {
        172.18.51.51/16 dev eth1
            }
        }
        
        
        virtual_server 172.18.50.50 80 {
            delay_loop 6
            lb_algo wrr
            lb_kind DR
            persistence_timeout 0
            protocol TCP
        sorry_server 127.0.0.1 80
        
            real_server 172.18.200.100 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
            real_server 172.18.249.57 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
        }
        
        virtual_server 172.18.51.51 80 {
            delay_loop 6
            lb_algo wrr
            lb_kind DR
            persistence_timeout 0
            protocol TCP
        sorry_server 127.0.0.1 80
        
            real_server 172.18.200.100 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
            real_server 172.18.249.57 80 {
                weight 1
                HTTP_GET {
                    url {
                      path /
        status_code 200
                    }
                    connect_timeout 3
                    nb_get_retry 3
                    delay_before_retry 3
                }
            }
        }
        
        VIP配置脚本 （由用户是双主模型因此VIP有两个）
        #!/bin/bash
        #
        vip='172.18.50.50'
        vip2='172.18.51.51'
        netmask='255.255.255.255'
        iface='lo:0'
        iface2='lo:1'
        
        case $1 in
        start)
        echo 1 > /pro/sys/net/ipv4/conf/all/arp_ignore
        echo 1 > /pro/sys/net/ipv4/conf/lo/arp_ignore
        echo 2 > /pro/sys/net/ipv4/conf/all/arp_announce
        echo 2 > /pro/sys/net/ipv4/conf/lo/arp_announce
        ifconfig $iface $vip netmask $netmask broadcast $vip up
        ifconfig $iface2 $vip2 netmask $netmask broadcast $vip2 up
        route add -host $vip dev $iface
        ;;
        stop)
        ifconfig $iface down
        ifconfig $iface2 down
        echo 0 > /pro/sys/net/ipv4/conf/all/arp_ignore
        echo 0 > /pro/sys/net/ipv4/conf/lo/arp_ignore
        echo 0 > /pro/sys/net/ipv4/conf/all/arp_announce
        echo 0 > /pro/sys/net/ipv4/conf/lo/arp_announce
        ;;
        esac
        实验结论：实现keepalived 高可用lvs负载均衡

