1、检查系统内核是否支持MPPE

modprobe ppp-compress-18 && echo OK
显示OK说明系统支持MPPE

2、检查系统是否开启TUN/TAP支持

cat /dev/net/tun
cat: /dev/net/tun: 文件描述符处于错误状态, 则表明通过

3、 检查PPP是否支持MPPE
strings '/usr/sbin/pppd'|grep -i mppe|wc -l
43

如果以上命令输出为“0”则表示不支持；输出为“30”或更大的数字就表示支持.

4、安装ppp和iptables      #PPTP需要这两个软件包，一般centOS自带

yum install -y ppp iptables

5、安装PPTP

yum install -y epel-release
yum install -y pptpd

6、配置PPTP
（1）vi /etc/ppp/options.pptpd #编辑，保存

name pptpd                    #自行设定的VPN服务器的名字，可以任意
#refuse-pap                    #拒绝pap身份验证
#refuse-chap                   #拒绝chap身份验证
#refuse-mschap                 #拒绝mschap身份验证
require-mschap-v2             #为了最高的安全性，我们使用mschap-v2身份验证方法
require-mppe-128              #使用128位MPPE加密
ms-dns 8.8.8.8
ms-dns 8.8.4.4
proxyarp                      #启用ARP代理，如果分配给客户端的IP与内网卡同一个子网
#debug                         #关闭debug
lock
nobsdcomp
novj
novjccomp
#nologfd                       #不输入运行信息到stderr
logfile /var/log/pptpd.log    #存放pptpd服务运行的的日志

（2）vi /etc/ppp/chap-secrets  #编辑，保存

kuaile pptpd 666666 *
或者 
vpnuser add kuaile 666666

（3）vi /etc/pptpd.conf #编辑，保存

option /etc/ppp/options.pptpd
logwtmp
localip 10.0.6.1               #设置VPN服务器虚拟IP地址
remoteip 10.0.6.101-200        #为拨入VPN的用户动态分配10.0.6.101~10.0.0.200之间的IP

7. 开启系统路由模式

sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0

vi /etc/sysctl.conf            #编辑
net.ipv4.ip_forward = 1        #找到此行 去点前面#，把0改成1 开启路由模式，如果没有就自行添加
sysctl -p                      #使设置立刻生效

sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

7、配置防火墙NAT转发

centos 7 默认采用firewalld动态防火墙，我还是更习惯使用iptables

yum install iptables-services

systemctl stop firewalld.service
systemctl disable firewalld.service
yum erase firewalld

systemctl enable iptables.service
systemctl start iptables.service

对于开启了iptables过滤的主机，需要开放VPN服务的端口：1723 和gre协议

使用一下命令添加开放pptp使用的1723端口和gre协议

iptables -A INPUT -p tcp -m state --state NEW,RELATED,ESTABLISHED -m tcp --dport 1723 -j ACCEPT  
iptables -A INPUT -p gre -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT  

iptables -A FORWARD -i ppp+ -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT

如果iptables规则中有拒绝的选项，需要注意接受的要在拒绝的前面。

iptables --list

开启包转发
iptables -t nat -A POSTROUTING -s 10.0.6.0/24 -o eth0 -j MASQUERADE
或者(这俩条应该是等效的，一种不行的话， 用另一种试试)
iptables -t nat -A POSTROUTING -s 10.0.6.0/24 -o eth0 -j SNAT --to 你的主机IP

这里要注意服务器的网口不一定是eth0，用netstat -i 查看

意思是对即将发送出去的数据包进行修改，对来自设备eth0且源地址是10.0.6.0/24的数据包，把源地址修改为主机地址及vPN地址

iptables -t nat -L                     #完成后可以查看NAT表是否已经生效

没问题后可以保存一下

service iptables save

这是iptables规则文件
cat /etc/sysconfig/iptables 

8、设置PPTP开机启动

service pptpd start                     #启动pptpd

systemctl enable pptpd                  #设置开机启动

pptpd服务使用的端口是1723，这个端口是系统固定分配的，可以通过查看该端口检查pptpd服务的运行情况。

命令：netstat -ntpl

至此，VPN服务器搭建完成.
