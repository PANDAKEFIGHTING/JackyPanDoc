# LINUX 系统初始化脚本

##  Cent OS (Rocky) System INIT

```
#!/bin/sh 

# 输出初始化信息
cat << EOF 
CentOS system INIT. Editor by PANDAKE.
EOF 

# 禁用IPv6
cat << EOF 
| === Welcome to Disable IPV6 === | 
+--------------------------------------------------------------+ 
EOF 

# 将IPv6的模块别名设置为关闭
echo "alias net-pf-10 off" >> /etc/modprobe.conf 
echo "alias ipv6 off" >> /etc/modprobe.conf 

# 关闭ip6tables服务，确保IPv6被禁用
/sbin/chkconfig --level 35 ip6tables off 
echo "ipv6 is disabled!" 

# 禁用SELinux
# 修改SELinux配置文件，将模式设置为disabled
sed -i '/SELINUX/s/enforcing/disabled/' /etc/selinux/config 
echo "selinux is disabled, effective after restart!" 

# 设置默认编辑器为vim
# 在.bashrc中添加别名，将vi替换为vim
sed -i "8 s/^/alias vi='vim'/" /root/.bashrc 
# 创建.vimrc文件并开启语法高亮
echo 'syntax on' > /root/.vimrc 

# 设置系统语言为中文
# 修改系统语言配置文件，将LANG设置为zh_CN.UTF-8
sed -i -e 's/^LANG=.*/LANG="zh_CN.UTF-8"/' /etc/sysconfig/i18n 

# 配置文件描述符最大限制为52100
# 在limits.conf中设置软限制和硬限制
echo "* soft nofile 52100 
* hard nofile 52100" >> /etc/security/limits.conf 

# 关闭不必要的服务
#-------------------------------------------------------------------------------- 
cat << EOF 
Welcome to trunoff services
EOF 
#--------------------------------------------------------------------------------- 

# 遍历运行级别3中所有的服务
for i in `ls /etc/rc3.d/S*` 
do 
    # 提取服务名称
    CURSRV=`echo $i|cut -c 15-` 

    echo $CURSRV 
    case $CURSRV in 
        # 列出需要保留的基本服务
        cpuspeed | crond | irqbalance | microcode_ctl | mysqld | network | nginx | php-fpm | sendmail | sshd | syslog ) 
            # 这些服务根据具体的应用情况设置，其中network、sshd、syslog是三项必须要启动的系统服务！ 
            echo "Base services, Skip!" 

        *) 
            # 对于其他服务，关闭它们
            echo "change $CURSRV to off" 
            chkconfig --level 235 $CURSRV off  # 将服务设置为关闭
            service $CURSRV stop  # 停止服务
    esac 
done 

# 清空sysctl.conf文件
rm -rf /etc/sysctl.conf 

# 配置内核参数
echo "net.ipv4.ip_forward = 0 
net.ipv4.conf.default.rp_filter = 1 
net.ipv4.conf.default.accept_source_route = 0 
kernel.sysrq = 0 
kernel.core_uses_pid = 1 
net.ipv4.tcp_syncookies = 1 
kernel.msgmnb = 65536 
kernel.msgmax = 65536 
kernel.shmmax = 68719476736 
kernel.shmall = 134217728 
net.ipv4.ip_local_port_range = 1024 65536 
net.core.rmem_max = 16777216 
net.core.wmem_max = 16777216 
net.ipv4.tcp_rmem = 4096 87380 16777216 
net.ipv4.tcp_wmem = 4096 65536 16777216 
net.ipv4.tcp_fin_timeout = 3 
net.ipv4.tcp_tw_recycle = 1 
net.core.netdev_max_backlog = 30000 
net.ipv4.tcp_no_metrics_save = 1 
net.core.somaxconn = 262144 
net.ipv4.tcp_syncookies = 0 
net.ipv4.tcp_max_orphans = 262144 
net.ipv4.tcp_max_syn_backlog = 262144 
net.ipv4.tcp_synack_retries = 2 
net.ipv4.tcp_syn_retries = 2 
vm.swappiness = 6" >> /etc/sysctl.conf 

# 输出内核优化配置完成的信息
echo "optimized kernel configure was done!" 
```

