1. copy ./firewall to /etc/network/if-pre-up.d/firewall
2. config iptables, allow port 1723, GRE and perform NAT
    ```
    
    sudo iptables -I INPUT -p tcp --dport 1723 -m state --state NEW -j ACCEPT
    
    sudo iptables -I INPUT -p gre -j ACCEPT
    
    sudo iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
    
    sudo iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -s 192.168.127.0/24 -j TCPMSS --clamp-mss-to-pmtu
    
    ```
3. save iptables rules
    ```
    
    sudo sh -c 'iptables-save > /etc/iptables.rules'
    
    ```
4. allow forward on host
    ```
    
    sudo sysctl net.ipv4.ip_forward=1
    
    sudo nano /etc/sysctl.conf  #net.ipv4.ip_forward=1
    
    sudo sysctl -p
    
    sudo nano /etc/default/ufw  #DEFAULT_FORWARD_POLICY=ACCEPT
    
    ```
5. modify chap-secrets to set username:password
6. install docker, docker-compose
    ```
    
    cd pptpd/
    
    docker-compose up -d
    
    ```
7. make sure all the things ready when reboot
    ```
    
    sudo reboot
    
    sudo service docker start
    
    ```
8. on your client device, connect PPTP server using username:password with mppe-128 encyption.