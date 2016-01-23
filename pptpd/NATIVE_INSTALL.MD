VPN pptpd SERVER

-enable port forwarding on router : port 1723

# install pptp
sudo apt-get install pptpd telnet iptables

# edit pptp
sudo nano /etc/pptpd.conf

localip <local_ip>
remoteip <remote_ip start - end>




# edit pptp options
sudo nano /etc/ppp/pptpd-options

noauth
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
nomppe-stateful ## MPPE should be stateless
proxyarp
nodefaultroute
lock
nobsdcomp
noaccomp
noipx ## you don't need IPX
mtu 1490 ## may help your linux client from disconnecting
mru 1490 ## may help your linux client from disconnecting

lcp-echo-failure 3
lcp-echo-interval 60

# edit users
sudo nano /etc/ppp/chap-secrets 

# ipp masquerading
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
or
sudo iptables -t nat -A POSTROUTING -s <local_nw> -o eth0 -j MASQUERADE
# allow port 1723 and protocol 47

sudo iptables --append INPUT --protocol 47 --jump ACCEPT
sudo iptables --append INPUT --protocol tcp --match tcp --destination-port 1723 --jump ACCEPT

# ip forwarding
sudo sysctl net.ipv4.ip_forward=1
or 
sudo nano /etc/sysctl.conf
net.ipv4.ip_forward=1

in /etc/default/ufw change the DEFAULT_FORWARD_POLICY to “ACCEPT”

# restart pptp
sudo /etc/init.d/pptpd restart

# troubleshoot - see logs
sudo tail -f /var/log/syslog