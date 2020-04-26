

### LVS 配置[非KeepAlived版本]

环境

使用3台服务器1个做lvs负载均衡服务器，2个做应用服务器


<!--more-->
#### 第一台LVS负载均衡服务器配置

配置网卡下的子网卡编号为 2     配置VIP为192.168.0.80

`ifconfig  eth33:2  192.168.0.80/24` 或者`ifconfig  eth33:2  192.168.0.80  netmast 255.255.255.0`           配置LVS服务器的VIP

如果想去掉子网卡的配置，可以用下面的命令[扩展]

`ifconfig eth33:3 down` 关闭eth33网卡下的3子网卡

第二台和第三台应用服务器配置

echo  1 >  /proc/sys/net/ipv4/conf/ens33/arp_ignore

echo  1 >  /proc/sys/net/ipv4/conf/all/arp_ignore

echo  2 >  /proc/sys/net/ipv4/conf/ens33/arp_announce

echo  2 >  /proc/sys/net/ipv4/conf/all/arp_announce

`ifconfig  lo:2  192.168.0.80  network:255.255.255.255`

然后开启80的服务器

最后在第一台上开始操作

yum -y install ipvsadm

`ipvsadm  -A  -t 192.168.0.80:80     -s  rr`轮询

ipvsadm  -a  -t 192.168.0.80:80     -r  192.168.0.115  -g -w l

ipvsadm  -a  -t 192.168.0.80:80     -r  192.168.0.116  -g -w l

OK了

ipvsadm -ln  查看状态 

ipvsadm -lnc    查看ipvs的转发记录表

netstat  -natp  第一台上没有连接



删除keepalived

rm -f /usr/local/sbin/keepalived
rm -f /usr/local/etc/rc.d/init.d/keepalived
rm -f /usr/local/etc/sysconfig/keepalived
rm -rf /usr/local/etc/keepalived
rm -f /usr/local/bin/genhash
rm -rf /usr/local/keepalived
rm -rf /etc/keepalived
rm -f /etc/rc.d/init.d/keepalived
rm -f /usr/sbin/keepalived
rm -f /etc/sysconfig/keepalived
rm -f /etc/systemd/system/multi-user.target.wants/keepalived.service



重新开始

LVS不要和应用服务器放在一台上

先关掉3台的防火墙

```shell
systemctl stop firewalld.service
```

启动两台应用服务器。开发80端口并测试访问

配置下面四句话

```shell
echo  1 >  /proc/sys/net/ipv4/conf/ens33/arp_ignore
echo  1 >  /proc/sys/net/ipv4/conf/all/arp_ignore
echo  2 >  /proc/sys/net/ipv4/conf/ens33/arp_announce
echo  2 >  /proc/sys/net/ipv4/conf/all/arp_announce
```

最后一台安装keepalived  +  ipvsadm(lvs客户端)

```shell
yum -y install  ipvsadm  keepalived
```

配置/etc/keepalived/keepalived.conf文件

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.80/24 dev ens33 label ens33:8
    }
}

virtual_server 192.168.0.80 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    net_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP

    real_server 192.168.0.114 80 {
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
    
    real_server 192.168.0.115 80 {
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