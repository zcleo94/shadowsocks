
# 1.shadowsocks
#安装pip
首先安装epel扩展源：

　　yum -y install epel-release

　　更新完成之后，就可安装pip：

　　yum -y install python-pip

　　安装完成之后清除cache：

　　yum clean all

这是在root用户时使用的命令，当前用户如果不具有root权限，加上sudo。

更新pip:

    pip install --upgrade pip

#安装shadowsocks：

    pip install shadowsocks
    
#配置shadowsocks：

    vi /etc/shadowsocks.json

#按以下配置:

    {
    "server":"主机ip",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "130":"密码",
         "131":"密码",
         "132":"密码",
         "3389":"密码",
         "53":"密码",
         "443":"密码"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}

# 2. 开启相应防火墙端口：

    firewall-cmd --zone=public --add-port=端口/tcp --permanent
    
# 重启端口：

    firewall-cmd --reload
 
# 3. 配置shadow socks自动启动服务

    vi  /etc/systemd/system/shadowsocks.service
    
# 写入以下内容保存后退出:

        [Unit]
        Description=Shadowsocks

        [Service]
        TimeoutStartSec=0
        ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

        [Install]
        WantedBy=multi-user.target
        
# 执行以下命令启动 shadowsocks 服务：

    systemctl enable shadowsocks
    
    systemctl start shadowsocks
    
# 查看服务状态
    
    systemctl status shadowsocks -l
    
    
# 4. 配置Google BBR加速效果

#更新系统版本：

    yum update
    
#查看系统版本:

    cat /etc/redhat-release 
    
# 安装elrepo并升级内核：

    rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
    yum --enablerepo=elrepo-kernel install kernel-ml -y

#耐心等等完成后 更新grup文件并重启系统：

    egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
    结果如下:
    [root@vultr ~]# egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
    CentOS Linux 7 Rescue 600024e37a16475b9366a1d70a5be3b9 (5.1.5-1.el7.elrepo.x86_64)
    CentOS Linux (5.1.5-1.el7.elrepo.x86_64) 7 (Core)
    CentOS Linux (3.10.0-957.12.2.el7.x86_64) 7 (Core)
    CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
    CentOS Linux (0-rescue-a9fa8fe7b3214c368cdebd8c7e36dd85) 7 (Core)
#设置为最新的后重启:

    grub2-set-default 0
    reboot
    
 #查看内核版本:
 
    uname -r
    得到如下：
    [root@vultr ~]# uname -r
    5.1.5-1.el7.elrepo.x86_64
    
# 开启BBR：

    vi /etc/sysctl.conf    # 在文件末尾添加如下内容

    net.core.default_qdisc = fq
    net.ipv4.tcp_congestion_control = bbr

# 加载系统参数:

    sysctl -p
    得到：
    net.ipv6.conf.all.accept_ra = 2
    net.ipv6.conf.eth0.accept_ra = 2
    net.core.default_qdisc = fq
    net.ipv4.tcp_congestion_control = bbr
    
 # 检查BBR开启:
 
    确定bbr已经成功开启：
    sysctl net.ipv4.tcp_available_congestion_control
    得到：
    net.ipv4.tcp_available_congestion_control = reno cubic bbr
    继续:
    lsmod | grep bbr
    得到类似下面的就表示开启了:
    tcp_bbr                20480  1
    
 # 结束！

