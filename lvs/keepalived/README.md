# 一、Keepalived介绍

    keepalived介绍： 是集群管理中保证集群高可用的一个服务软件，其功能类似于heartbeat，用来防止单点故障

    Keepalived作用： 为haproxy提供vip（192.168.2.10）在三个haproxy实例之间提供主备，降低当其中一个haproxy失效的时对服务的影响。
    
## 1、yum安装Keepalived
```bash
# 安装keepalived
yum install -y keepalived
```

## 2、配置Keepalived
```bash
cat <<EOF > /etc/keepalived/keepalived.conf
! Configuration File for keepalived

# 主要是配置故障发生时的通知对象以及机器标识。
global_defs {
   # 标识本节点的字条串，通常为 hostname，但不一定非得是 hostname。故障发生时，邮件通知会用到。
   router_id LVS_K8S
}

# 用来做健康检查的，当时检查失败时会将 vrrp_instance 的 priority 减少相应的值。
vrrp_script check_haproxy {
    script "killall -0 haproxy"   #根据进程名称检测进程是否存活
    interval 3
    weight -2
    fall 10
    rise 2
}

# rp_instance用来定义对外提供服务的 VIP 区域及其相关属性。
vrrp_instance VI_1 {
    state MASTER   #当前节点为MASTER，其他两个节点设置为BACKUP
    interface eth0 #改为自己的网卡
    virtual_router_id 51
    priority 200
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 35f18af7190d51c9f7f78f37300a0cbd
    }
    virtual_ipaddress {
        192.168.2.10   #虚拟ip，即VIP
    }
    track_script {
        check_haproxy
    }
}
EOF
```

参考文档:

https://mp.weixin.qq.com/s?__biz=MzAwNTM5Njk3Mw==&mid=2247487183&idx=1&sn=1dfddfd2d1f883cc568f311a4d77ced7&chksm=9b1c0e4dac6b875ba7f388f8f383d3c99dd10ea01f022d9bab8b929edb858d06433c5f2d30a9&mpshare=1&scene=23&srcid=&sharer_sharetime=1569483288434&sharer_shareid=73f6a617f7f81d90d08fd8ee497b58ac#rd  