# Preparation:

SERVER A: 192.168.217.98

SERVER B: 192.168.217.99 (installed Nginx)

WIN 10: 192.168.217.1

**Turn off the firewall all of them**

On ubuntu, iptables and ufw are automatically installed so we just need to check their version using these following commands: `iptables –version` or `ufw –version`

By default, Ubuntu will use ufw so if we want to use iptables, we need to remove the ufw using the following command: 

    sudo apt-get remove ufw –y

**Task 1. Configure blocks all IP access to SERVER A via ssh port except IP 192.168.217.1**

Run 2 following commands in SERVER A

    iptables -I INPUT -p tcp --dport ssh -j REJECT
    
  >

    iptables -I INPUT -p tcp -s 192.168.217.1 --dport ssh -j ACCEPT

**Task 2. Configure blocks all IP access to SERVER B via http port except IP 192.168.217.1**

Run 2 following commands in SERVER B

    iptables -I INPUT -p tcp --dport http -j REJECT

  >

    iptables -I INPUT -p tcp -s 192.168.217.1 --dport http -j ACCEPT

**Task 3. When Win 10 with IP 192.168.217.1 accesses the http service on port 80 to SERVER A, convert the address to SERVER B (192.168.217.1 => SERVER A:80 => SERVER B:80)**

Run this command in SERVER B

    iptables -I INPUT -p tcp -s 192.168.217.98 --dport http -j ACCEPT

Run these following commands in SERVER A

    echo "1" > /proc/sys/net/ipv4/ip_forward

  >

    iptables -t nat -I PREROUTING -p tcp -d 192.168.217.98 --dport 80 -j DNAT --to-destination 192.168.217.99:80

  >

    iptables -t nat -I POSTROUTING -p tcp -d 192.168.217.99 --dport 80 -j SNAT --to-source 192.168.217.98



