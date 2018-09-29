---
title: Keepalived
categories:
- Keepalived
tags:
- Cluster 
- Keepalived


---
Keepalived是基于vrrp协议的一款高可用软件。Keepailived有一台主服务器和多台备份服务器，在主服务器和备份服务器上面部署相同的服务配置，使用一个虚拟IP地址对外提供服务，当主服务器出现故障时，虚拟IP地址会自动漂移到备份服务器。

VRRP（Virtual Router Redundancy Protocol，虚拟路由器冗余协议），VRRP是为了解决静态路由的高可用。VRRP的基本架构  
虚拟路由器由多个路由器组成，每个路由器都有各自的IP和共同的VRID(0-255)，其中一个VRRP路由器通过竞选成为MASTER，占有VIP，对外提供路由服务，其他成为BACKUP，MASTER以IP组播（组播地址：224.0.0.18）形式发送VRRP协议包，与BACKUP保持心跳连接，若MASTER不可用（或BACKUP接收不到VRRP协议包），则BACKUP通过竞选产生新的MASTER并继续对外提供路由服务，从而实现高可用。

vrrp协议的相关术语：

        虚拟路由器：Virtual Router 
        虚拟路由器标识：VRID(0-255)
        物理路由器：
            master  ：主设备
            backup  ：备用设备
            priority：优先级
        VIP：Virtual IP 
        VMAC：Virutal MAC (00-00-5e-00-01-VRID)
        GraciousARP
    

安全认证：

    简单字符认证、HMAC机制，只对信息做认证
    MD5（leepalived不支持）
    

工作模式：

    主/备：单虚拟路径器；
    主/主：主/备（虚拟路径器），备/主（虚拟路径器）
    

工作类型:

    抢占式：当出现比现有主服务器优先级高的服务器时，会发送通告抢占角色成为主服务器
    非抢占式：
    

keepalived
----------

### 核心组件：

            vrrp stack：vrrp协议的实现
            ipvs wrapper：为集群内的所有节点生成IPVS规则
            checkers：对IPVS集群的各RS做健康状态检测
            控制组件：配置文件分析器，用来实现配置文件的分析和加载
            IO复用器
            内存管理组件，用来管理keepalived高可用是的内存管理
    

注意：

1.  各节点时间必须同步
2.  确保各节点的用于集群服务的接口支持MULTICAST通信（组播）；

### 安装

从CentOS 6.4开始keepalived随系统base仓库提供，可以使用`yun -y install keepalived`安装。

###### 配置文件：

            主配置文件：/etc/keepalived/keepalived.conf
            主程序文件：/usr/sbin/keepalived
            提供校验码：/usr/bin/genhash
            Unit File：keepalived.service
            Unit File的环境配置文件：/etc/sysconfig/keepalived
    

### 配置

### 主配置文件详解

    ! Configuration File for keepalived
    
    global_defs {
       notification_email {   #发送报警邮件收件地址
         acassen@firewall.loc
         failover@firewall.loc
         sysadmin@firewall.loc
       }
       notification_email_from Alexandre.Cassen@firewall.loc   #指明报警邮件的发送地址
       smtp_server 192.168.200.1    #邮件服务器地址
       smtp_connect_timeout 30      #smtp的超时时间
       router_id LVS_DEVEL          #物理服务器的主机名
       vrrp_mcast_group4            #定义一个组播地址
       static_ipaddress
            {
            192.168.1.1/24 dev eth0 scope global
            }
            static_routes
            {
            192.168.2.0/24 via 192.168.1.100 dev eth0
            }
    
    }
        vrrp_sync_group VG_1 {              #定义一个故障组，组内有一个虚拟路由出现故障另一个也会一起跟着转移，适用于LVS的NAT模型。
            group {
                VI1     # name of vrrp_instance (below)         
                VI2     # One for each moveable IP
    
                 }
        }
    
    
    vrrp_instance VI_1 {        #定义一个虚拟路由
        state MASTER|BACKUP     #当前节点在此虚拟路由器上的初始状态；只能有一个是MASTER，余下的都应该为BACKUP；
        interface eth0          #绑定为当前虚拟路由器使用的物理接口；
        virtual_router_id 51    #当前虚拟路由器的惟一标识，范围是0-255；
        priority 100            #当前主机在此虚拟路径器中的优先级；范围1-254；
        advert_int 1            #通告发送间隔，包含主机优先级、心跳等。
        authentication {        #认证配置   
            auth_type PASS      #认证类型，PASS表示简单字符串认证
            auth_pass 1111      #密码,PASS密码最长为8位
    
       virtual_ipaddress {
        192.168.200.16          #虚拟路由IP地址，以辅助地址方式设置
        192.168.200.18/24 dev eth2 label eth2:1 #以别名的方式设置
        }
    
    track_interface {
            eth0
            eth1
    
    }                           #配置要监控的网络接口，一旦接口出现故障，则转为FAULT状态；
    nopreempt                   #定义工作模式为非抢占模式；
    preempt_delay 300           #抢占式模式下，节点上线后触发新选举操作的延迟时长；
       virtual_routes {         #配置路由信息，可选项
                   #  src <IPADDR> [to] <IPADDR>/<MASK> via|gw <IPADDR> [or <IPADDR>] dev <STRING> scope
           <SCOPE> tab
                   src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
                   192.168.112.0/24 via 192.168.100.254       192.168.113.0/24  via  192.168.200.254  or 192.168.100.254 dev eth1      blackhole 192.168.114.0/24
               }
    
    
        notify_master <STRING>|<QUOTED-STRING>       #当前节点成为主节点时触发的脚本。
        notify_backup <STRING>|<QUOTED-STRING>       #当前节点转为备节点时触发的脚本。
        notify_fault <STRING>|<QUOTED-STRING>        #当前节点转为“失败”状态时触发的脚本。
        notify <STRING>|<QUOTED-STRING>              #通用格式的通知触发机制，一个脚本可完成以上三种状态的转换时的通知。
        smtp_alert                                   #如果加入这个选项，将调用前面设置的邮件设置，并自动根据状态发送信息 
    }
    virtual_server 192.168.200.100 443 {    #LVS配置段 ，设置LVS的VIP地址和端口
        delay_loop                          #服务轮询的时间间隔；检测RS服务器的状态。
        lb_algo rr                          #调度算法，可选rr|wrr|lc|wlc|lblc|sh|dh。
        lb_kind NAT                         #集群类型。
        nat_mask 255.255.255.0              #子网掩码，可选项。
        persistence_timeout 50              #是否启用持久连接，连接保存时长
        protocol TCP                        #协议，只支持TCP
        sorry_server <IPADDR> <PORT>        #备用服务器地址，可选项。
    
        real_server 192.168.201.100 443 {   #配置RS服务器的地址和端口
            weight 1                        #权重
         SSL_GET {                          #检测RS服务器的状态，发送请求报文
                url {
                  path /                    #请求的URL
                  digest ff20ad2481f97b1754ef3e12ecd3a9cc  #对请求的页面进行hash运算，然后和这个hash码进行比对，如果hash码一样就表示状态正常
                  status_code <INT>         #判断上述检测机制为健康状态的响应码,和digest二选一即可。
    
                }                           #这个hash码可以使用genhash命令请求这个页面生成
                connect_timeout 3           #连接超时时间
                nb_get_retry 3              #超时重试次数
                delay_before_retry 3        #每次超时过后多久再进行连接
                connect_ip <IP ADDRESS>     #向当前RS的哪个IP地址发起健康状态检测请求
                connect_port <PORT>         #向当前RS的哪个PORT发起健康状态检测请求
                bindto <IP ADDRESS>         #发出健康状态检测请求时使用的源地址；
                bind_port <PORT>            #发出健康状态检测请求时使用的源端口；
            }
        }
    }
    

###### 健康状态检测机制

1.  HTTP_GET
2.  SSL_GET
3.  TCP_CHECK
4.  SMTP_CHECK
5.  MISS_CHECK #调用自定义脚本进行检测

    TCP_CHECK {
            connect_ip <IP ADDRESS>         #向当前RS的哪个IP地址发起健康状态检测请求;
            connect_port <PORT>             #向当前RS的哪个PORT发起健康状态检测请求;
            bindto <IP ADDRESS>             #发出健康状态检测请求时使用的源地址；
            bind_port <PORT>                #发出健康状态检测请求时使用的源端口；
            connect_timeout <INTEGER>       #连接请求的超时时长；
    }
    

### 实现LVS高可用集群

实验主机  
虚拟IP:192.168.166.100

2台CentOS 7.3

    CentOS 7.3   主服务器，  IP：192.168.166.130
    CentOS 7.3-1 备份服务器，IP：192.168.166.132
    

2台CentOS 6.9

    CentOS 6.9  IP：192.168.166.129
    CentOS6.9-1 IP：192.168.166.131
    

### 注：在配置服务前需要注意几台主机的防火墙策略，和SELinux配置。

主调度器配置

    [root@CentOS7.3 ~]#yum -y install keepalived ipvsadm        #安装keepalived和LVS管理软件ipvsadm
    [root@CentOS7.3 ~]#vim /etc/keepalived/keepalived.conf  #配置keepalived
    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
       root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1                    #邮件服务器的地址
       smtp_connect_timeout 30
       router_id CentOS7.3                      #主调度器的主机名
       vrrp_mcast_group4 224.26.1.1             #发送心跳信息的组播地址
        
    }
    
    vrrp_instance VI_1 {
        state MASTER                            #主调度器的初始角色
        interface eth0                          #虚拟IP工作的网卡接口
        virtual_router_id 66                    #虚拟路由的ID
        priority 100                            #主调度器的选举优先级
        advert_int 1
        authentication {
            auth_type PASS                      #集群主机的认证方式
            auth_pass 123456                    #密钥,最长8位
        }
        virtual_ipaddress {
            192.168.166.100                     #虚拟IP
        }
    }
    
    virtual_server 192.168.166.100 80 {         #LVS配置段，VIP
        delay_loop 6
        lb_algo rr                              #调度算法轮询
        lb_kind DR                              #工作模式DR
        nat_mask 255.255.255.0
    #    persistence_timeout 50                  #持久连接，在测试时需要注释，否则会在设置的时间内把请求都调度到一台RS服务器上面
        protocol TCP
        sorry_server 127.0.0.1 80               #Sorry server的服务器地址及端口
    #Sorry server就是在后端的服务器全部宕机的情况下紧急提供服务。
        real_server 192.168.166.129 80 {        #RS服务器地址和端口
            weight 1                            #RS的权重
            HTTP_GET {                          #健康状态检测方法
                url {
                  path /
                  status_code 200               #状态判定规则
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    
        real_server 192.168.166.131 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    }
    
    [root@CentOS7.3 keepalived]#systemctl start keepalived          #启动keepalived
    [root@CentOS7.3 keepalived]#ip a l eth0                         #查看虚拟路由绑定的网卡    
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:b9:7d:cb brd ff:ff:ff:ff:ff:ff
        inet 192.168.166.130/24 brd 192.168.166.255 scope global eth0
           valid_lft forever preferred_lft forever
        inet 192.168.166.100/32 scope global eth0                   #虚拟IP已经绑定在了eth网卡上
           valid_lft forever preferred_lft forever
        inet6 fe80::50fe:a3f3:83a0:d38a/64 scope link 
           valid_lft forever preferred_lft forever
    

备份调度器的配置

    [root@centos7.3-1 ~]#yum -y install keepalived ipvsadm 
    
    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
       root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id CentOS7.3-1                #备份调度器的主机名
       vrrp_mcast_group4 224.26.1.1         #这个组播地址需与集群内的其他主机相同
    
    }
    
    vrrp_instance VI_1 {
        state BACKUP                        #初始角色，备份服务器需设置为BACKUP
        interface eth0
        virtual_router_id 66                #虚拟路由的ID一定要和集群内的其他主机相同
        priority 90                         #选举优先级，要比主调度器地一些
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 123456                #密钥需要和集群内的主服务器相同
        }
        virtual_ipaddress {
            192.168.166.100
        }
    }
                    #余下配置和主服务器相同
    virtual_server 192.168.166.100 80 {
        delay_loop 6
        lb_algo rr
        lb_kind DR
        nat_mask 255.255.255.0
    #    persistence_timeout 50              
        protocol TCP
        sorry_server 127.0.0.1 80
    
        real_server 192.168.166.129 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                  status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    
        real_server 192.168.166.131 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    }
    [root@centos7.3-1 ~]#systemctl start keepalived         #启动备份keepalived
    [root@centos7.3-1 ~]#ip a l eth0                        #查看虚拟路由绑定的网卡接口
    [root@centos7.3-1 ~]#ip a l eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:7e:ec:ef brd ff:ff:ff:ff:ff:ff
        inet 192.168.166.132/24 brd 192.168.166.255 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::9aab:52b3:cc1e:fbef/64 scope link 
           valid_lft forever preferred_lft forever
    

测试虚拟IP地址漂移

关闭主服务器的keepalived，并查看eth0接口

  

![](//upload-images.jianshu.io/upload_images/6094829-e4d7bff45ad727cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/787)

image.png

查看备份服务器的eth0接口，地址已经漂移到了备份服务器上面

  

![](//upload-images.jianshu.io/upload_images/6094829-c1f62fd0aa971d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/841)

image.png

可以看到上图提示有新邮件。使用mail命令查看邮件列表，都是后端服务器状态检测的邮件，说明配置的报警邮件生效了。应为后端服务器还没有配置所以检测的状态全是down。

  

![](//upload-images.jianshu.io/upload_images/6094829-fbe80da78025eb5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000)

image.png

启动主服务器，地址又漂移回了主服务器

  

![](//upload-images.jianshu.io/upload_images/6094829-7676dc6d0bd89c99.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/827)

image.png

配置RS服务器

RS1配置

    [root@CentOS6.9 ~]#yum -y install httpd                 #安装httpd服务
    [root@CentOS6.9 ~]#vim lvs.sh                           #创建一个配置脚本
    #!/bin/bash
    vip=192.168.166.100                                     #VIP地址
    mask=255.255.255.255                    
    dev=eth0:1
    case $1 in
    start)
            echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
            echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
            echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
            echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
            ifconfig $dev $vip netmask $mask broadcast $vip up
            route add -host $vip dev $dev
    ;;
    stop)
            ifconfig $dev down
            echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
            echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
            echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
            echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ;;
    *)
            echo "Usage: $(basename $0) start|stop"
            exit 1
    ;;
    esac
    
    [root@CentOS6.9 ~]#bash lvs.sh start
    [root@CentOS6.9 ~]#ip a l eth0:0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
        link/ether 00:0c:29:23:38:c9 brd ff:ff:ff:ff:ff:ff
        inet 192.168.166.129/24 brd 192.168.166.255 scope global eth0
        inet 192.168.166.100/32 brd 192.168.166.100 scope global eth0:1
        inet6 fe80::20c:29ff:fe23:38c9/64 scope link 
           valid_lft forever preferred_lft forever
    [root@CentOS6.9 ~]#echo WebServer1 > /var/www/html/index.html
    [root@CentOS6.9 ~]#cat /var/www/html/index.html 
    WebServer1
    [root@CentOS6.9 ~]#service httpd start
    
    

使用ipvsadm 命令查看lvs配置信息，RS1服务器已经调度器添加进集群。

![](//upload-images.jianshu.io/upload_images/6094829-e0f2bd2f0bb81415.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/662)

image.png

RS2配置

    [root@CentOS6.9-1 ~]#yum -y install httpd
    [root@CentOS6.9-1 ~]#vim lvs.sh                        #和上面RS1的lvx.sh内容相同
    [root@CentOS6.9-1 ~]#echo WebServer2 > /var/www/html/index.html
    [root@CentOS6.9-1 ~]#cat /var/www/html/index.html 
    WebServer2
    [root@CentOS6.9-1 ~]#service httpd start
    
    

第二台RS服务器上线

  

![](//upload-images.jianshu.io/upload_images/6094829-975303b8c8acc0f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/702)

image.png

客户端测试

因为使用的是轮询算法，所以会在Web1和Web2之间来回调度。

  

![](//upload-images.jianshu.io/upload_images/6094829-e1f8348995e3a2b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/756)

image.png

### 注意：如果在测试是开启了lvs的长连接，会导致在设置的时间内把客户端一直调度到同一台RS服务器上面。

关闭主调度器

image.png

客户端访问

  

image.png

### 我们还可以使用这些主机配置来两套LVS高可用，做一个双主模型

第二套LVS信息

VIP：192.168.166.200

第一台调度器为备份服务器

第二台调度器为主服务器

第一台调度器配置

    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
       root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id CentOS7.3
       vrrp_mcast_group4 224.26.1.1
    }
    
    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 66
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 123456
        }
        virtual_ipaddress {
            192.168.166.100
        }
    }
    
    virtual_server 192.168.166.100 80 {
        delay_loop 6
        lb_algo rr
        lb_kind DR
        nat_mask 255.255.255.0
    #   persistence_timeout 50
        protocol TCP
        sorry_server 127.0.0.1 80
        real_server 192.168.166.129 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                  status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    
        real_server 192.168.166.131 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    }
    #第二套虚拟路由
    vrrp_instance VI_2 {
        state BACKUP
        interface eth0
        virtual_router_id 88            #ID不要和第一套虚拟路由相同
        priority 90
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 12345678
        }
       virtual_ipaddress {
            192.168.166.200         
        }
    }
    
    virtual_server 192.168.166.200 80 {
        delay_loop 6
        lb_algo rr
        lb_kind DR
        nat_mask 255.255.255.0
    #   persistence_timeout 50
        protocol TCP
        sorry_server 127.0.0.1 80
    
        real_server 192.168.166.129 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                  status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    
        real_server 192.168.166.131 80 {
            weight 1
            HTTP_GET {
                url {
                  path /
                    status_code 200
                }
                connect_timeout 1
                nb_get_retry 3
                delay_before_retry 1
            }
        }
    }
    
    

### 第二台的配置这里就不列出了,把第一台服务器的配置文件修改一下。下图是配置好的结果。

第一台服务器

image.png

第二台服务器

  

![](//upload-images.jianshu.io/upload_images/6094829-c9714a74f433d683.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/936)

image.png

RS配置

    #!/bin/bash
    vip=192.168.166.100
    mask=255.255.255.255
    dev=eth0:1                      
    vip2=192.168.166.200        #添加一个VIP2
    mask2=255.255.255.255
    dev2=eth0:2                 #再添加一个eth0：2别名
    case $1 in
    start)
            echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
            echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
            echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
            echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
            ifconfig $dev $vip netmask $mask broadcast $vip up
            ifconfig $dev2 $vip2 netmask $mask2 broadcast $vip2 up      #设置地址
            route add -host $vip dev $dev
            route add -host $vip2 dev $dev2
    
    ;;
    stop)
            ifconfig $dev down
            ifconfig $dev2 down
            echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
            echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
            echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
            echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ;;
    *)
            echo "Usage: $(basename $0) start|stop"
            exit 1
    ;;
    esac
    
    

### 注：上面这份脚本RS1和RS2通用

RS

  

![](//upload-images.jianshu.io/upload_images/6094829-92de2d83b55393f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/770)

image.png

RS2

  

![](//upload-images.jianshu.io/upload_images/6094829-e629acd026382e14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/933)

image.png

调度器LVS规则

  

![](//upload-images.jianshu.io/upload_images/6094829-8d41d5870a1ea579.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/702)

image.png

测试

![](//upload-images.jianshu.io/upload_images/6094829-e0096656a580461b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/856)

image.png

image.png

### 高可用双主Nginx

配置Nginx主机

    [root@centos7.3 ~]#yum -y install nginx                 #安装nginx，nginx在epel源。
    [root@centos7.3 ~]#vim /etc/nginx/nginx.conf            #修改nginx主配置文件
    user nginx;
    worker_processes auto;
    error_log /var/log/nginx/error.log;
    pid /run/nginx.pid;
    
    # Load dynamic modules. See /usr/share/nginx/README.dynamic.
    include /usr/share/nginx/modules/*.conf;
    
    events {
        worker_connections 1024;
    }
    
    http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        access_log  /var/log/nginx/access.log  main;
    
        sendfile            on;
        tcp_nopush          on;
        tcp_nodelay         on;
        keepalive_timeout   65;
        types_hash_max_size 2048;
    
        include             /etc/nginx/mime.types;
        default_type        application/octet-stream;
        
    upstream web {                          #在http上下文中添加一个服务器组，web是组名。
        server 192.168.166.129:80;              #后端服务器的地址和端口
        server 192.168.166.132:80;
    }
    
        # Load modular configuration files from the /etc/nginx/conf.d directory.
        # See http://nginx.org/en/docs/ngx_core_module.html#include
        # for more information.
        include /etc/nginx/conf.d/*.conf;
    
        server {
            listen       [::]:80 default_server;
            server_name  _;
            root         /usr/share/nginx/html;
    
            # Load configuration files for the default server block.
            include /etc/nginx/default.d/*.conf;
    
            location / {
            }
    
            error_page 404 /404.html;
                location = /40x.html {
            }
        include /etc/nginx/conf.d/*.conf;
    
        server {
            listen       [::]:80 default_server;
            server_name  _;
            root         /usr/share/nginx/html;
    
            # Load configuration files for the default server block.
            include /etc/nginx/default.d/*.conf;
    
            location / {
            }
    
            error_page 404 /404.html;
                location = /40x.html {
            }
    
            error_page 500 502 503 504 /50x.html;
                location = /50x.html {
            }
        }
    

定义两个Nginx虚拟主机

    [root@centos7.3 nginx]#vim /etc/nginx/conf.d/host.conf 
    server {
            server_name www.test.com;
            listen 80;
            index index.html;
            root /app/web;
            location / {
            proxy_pass http://web;
    
            }
    }
    
    

### zhu注：以上内容两台主机相同

配置keepalived  
第一台

    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
         root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id CentOS7.3
       vrrp_mcast_group4 224.24.1.1
    }
    
    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 66
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 111111
        }
        virtual_ipaddress {
            192.168.166.100/24 dev eth0
        }
    }
    vrrp_instance VI_2 {
        state BACKUP
        interface eth0
        virtual_router_id 88
        priority 90
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 11111111
        }
        virtual_ipaddress {
            192.168.166.200/24 dev eth0
        }
    }
    
    

第二台Nginx

    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
         root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id CentOS7.3
       vrrp_mcast_group4 224.24.1.1
    }
    
    vrrp_instance VI_1 {
        state BACKUP
        interface eth0
        virtual_router_id 66
        priority 90
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 111111
        }
        virtual_ipaddress {
            192.168.166.100/24 dev eth0
        }
    }
    vrrp_instance VI_2 {
        state MASTER
        interface eth0
        virtual_router_id 88
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 11111111
        }
        virtual_ipaddress {
            192.168.166.200/24 dev eth0
        }
    }
    
    

    
    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
         root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id CentOS7.3-1
       vrrp_mcast_group4 224.24.1.1
    }
    
    vrrp_instance VI_1 {
        state BACKUP
        interface eth0
        virtual_router_id 66
        priority 90
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 111111
        }
        virtual_ipaddress {
            192.168.166.100/24 dev eth0
        }
    
    

测试

![image](https://images2018.cnblogs.com/blog/1279115/201809/1279115-20180912174105907-2030065151.png)

keepalived可以调用外部的辅助脚本进行资源监控，并根据监控的结果状态能实现优先动态调整；

先定义一个脚本

    vrrp_script <SCRIPT_NAME> {                  #定义脚本
        script "killall -0 sshd"                 #可以在引号内调用命令或者脚本路径，如果脚本执行成功则不变，如果失败则执行下面的命令
        interval INT                         #检测间隔时间
        weight -INT                              #减掉权重
        fall    2                                #检测几次判定为失败
        rise    2                                #检测几次判定为成功
    }
    

killall -0 只是测试，并不执行操作，用来测试进程是否运行正常  
调用此脚本

    track_script {
        SCRIPT_NAME_1               #调用脚本
        SCRIPT_NAME_2  weight  -2    #如果脚本健康状态检测失败优先级减2
    }
    

配置文件

    ! Configuration File for keepalived
    
    global_defs {
       notification_email {
         root@localhost
       }
       notification_email_from keepalived@localhost
       smtp_server 127.0.0.1
       smtp_connect_timeout 30
       router_id CentOS7.3
       vrrp_mcast_group4 224.24.1.1
    }
    vrrp_script nginx {
            script "killall -0 nginx && exit 0 || exit 1"
            interval 1
            weight -15
            fall 2
            rise 1
    }
    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 66
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 111111
        }
        virtual_ipaddress {
            192.168.166.100/24 dev eth0
        }
       track_script {
            nginx
       }
    }
    
    vrrp_instance VI_2 {
        state BACKUP
        interface eth0
        virtual_router_id 88
        priority 90
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 11111111
        }
        virtual_ipaddress {
            192.168.166.200/24 dev eth0
        }
    }
    
    
    

测试

![image](https://images2018.cnblogs.com/blog/1279115/201809/1279115-20180912174109429-1007529962.png)


![image](https://images2018.cnblogs.com/blog/1279115/201809/1279115-20180912174111604-1147812384.png)


![image](https://images2018.cnblogs.com/blog/1279115/201809/1279115-20180912174113597-69130709.png)


![image](https://images2018.cnblogs.com/blog/1279115/201809/1279115-20180912174115367-1221507812.png)



 keepalived通知脚本
    
    下面的脚本可以接受选项，其中：
    -s, --service SERVICE,...：指定服务脚本名称，当状态切换时可自动启动、重启或关闭此服务；
    -a, --address VIP: 指定相关虚拟路由器的VIP地址；
    -m, --mode {mm|mb}：指定虚拟路由的模型，mm表示主主，mb表示主备；它们表示相对于同一种服务而方，其VIP的工作类型；
    -n, --notify {master|backup|fault}：指定通知的类型，即vrrp角色切换的目标角色；
    -h, --help：获取脚本的使用帮助；
    
    #!/bin/bash
    # Author: MageEdu <linuxedu@foxmail.com>
    # description: An example of notify script
    # Usage: notify.sh -m|--mode {mm|mb} -s|--service SERVICE1,... -a|--address VIP  -n|--notify {master|backup|falut} -h|--help 
     
    #contact='linuxedu@foxmail.com'
    helpflag=0
    serviceflag=0
    modeflag=0
    addressflag=0
    notifyflag=0
     
    contact='root@localhost'
     
    Usage() {
      echo "Usage: notify.sh [-m|--mode {mm|mb}] [-s|--service SERVICE1,...] <-a|--address VIP>  <-n|--notify {master|backup|falut}>" 
      echo "Usage: notify.sh -h|--help"
    }
     
    ParseOptions() {
      local I=1;
      if [ $# -gt 0 ]; then
        while [ $I -le $# ]; do
          case $1 in
      -s|--service)
    [ $# -lt 2 ] && return 3
             serviceflag=1
             services=(`echo $2|awk -F"," '{for(i=1;i<=NF;i++) print $i}'`)
    shift 2 ;;
      -h|--help)
             helpflag=1
    return 0
            shift
    ;;
      -a|--address)
    [ $# -lt 2 ] && return 3
        addressflag=1
    vip=$2
    shift 2
    ;;
      -m|--mode)
    [ $# -lt 2 ] && return 3
    mode=$2
    shift 2
    ;;
      -n|--notify)
    [ $# -lt 2 ] && return 3
    notifyflag=1
    notify=$2
    shift 2
    ;;
      *)
    echo "Wrong options..."
    Usage
    return 7
    ;;
           esac
        done
        return 0
      fi
    }
     
    #workspace=$(dirname $0)
     
    RestartService() {
      if [ ${#@} -gt 0 ]; then
        for I in $@; do
          if [ -x /etc/rc.d/init.d/$I ]; then
            /etc/rc.d/init.d/$I restart
          else
            echo "$I is not a valid service..."
          fi
        done
      fi
    }
     
    StopService() {
      if [ ${#@} -gt 0 ]; then
        for I in $@; do
          if [ -x /etc/rc.d/init.d/$I ]; then
            /etc/rc.d/init.d/$I stop
          else
            echo "$I is not a valid service..."
          fi
        done
      fi
    }
     
     
    Notify() {
        mailsubject="`hostname` to be $1: $vip floating"
        mailbody="`date '+%F %H:%M:%S'`, vrrp transition, `hostname` changed to be $1."
        echo $mailbody | mail -s "$mailsubject" $contact
    }
     
     
    # Main Function
    ParseOptions $@
    [ $? -ne 0 ] && Usage && exit 5
     
    [ $helpflag -eq 1 ] && Usage && exit 0
     
    if [ $addressflag -ne 1 -o $notifyflag -ne 1 ]; then
      Usage
      exit 2
    fi
     
    mode=${mode:-mb}
     
    case $notify in
    'master')
      if [ $serviceflag -eq 1 ]; then
          RestartService ${services[*]}
      fi
      Notify master
      ;;
    'backup')
      if [ $serviceflag -eq 1 ]; then
        if [ "$mode" == 'mb' ]; then
          StopService ${services[*]}
        else
          RestartService ${services[*]}
        fi
      fi
      Notify backup
      ;;
    'fault')
      Notify fault
      ;;
    *)
      Usage
      exit 4
      ;;
    esac
    
    在keepalived.conf配置文件中，其调用方法如下所示：
    notify_master "/etc/keepalived/notify.sh -n master -a 172.16.100.1"  
    notify_backup "/etc/keepalived/notify.sh -n backup -a 172.16.100.1"  
    notify_fault "/etc/keepalived/notify.sh -n fault -a 172.16.100.1"  
        