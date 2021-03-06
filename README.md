# Preparation:

> SERVER A: 192.168.217.98
>
> SERVER B: 192.168.217.99 (installed Nginx)
>
> WIN 10: 192.168.217.1
>
> **Turn off the firewall all of them**

On ubuntu, iptables and ufw are automatically installed so we just need to check their version using these following commands: `iptables --version` or `ufw --version`

By default, Ubuntu will use ufw so if we want to use iptables, we need to remove the ufw using this command: 

    sudo apt-get remove ufw -y

**Task 1. Configure blocks all traffic to SERVER A via ssh port except IP 192.168.217.1**

Run 2 following commands in SERVER A

    iptables -I INPUT -p tcp --dport ssh -j REJECT

  > Reject all traffic using the tcp protocol via port 22 to SERVER A
    
  >

    iptables -I INPUT -p tcp -s 192.168.217.1 --dport ssh -j ACCEPT
  
  > Accept IP 192.168.217.1 using the tcp protocol via port 22 to SERVER A

**Task 2. Configure blocks all traffic to SERVER B via http port except IP 192.168.217.1**

Run 2 following commands in SERVER B

    iptables -I INPUT -p tcp --dport http -j REJECT

  > Reject all traffic using the tcp protocol via port 80 to SERVER B

  >

    iptables -I INPUT -p tcp -s 192.168.217.1 --dport http -j ACCEPT

  > Accept IP 192.168.217.1 using the tcp protocol via port 80 to SERVER B 

**Task 3. When Win 10 with IP 192.168.217.1 accesses the http service on port 80 to SERVER A, convert the address to SERVER B (192.168.217.1 => SERVER A:80 => SERVER B:80)**

Run this command in SERVER B

    iptables -I INPUT -p tcp -s 192.168.217.98 --dport http -j ACCEPT

  > Accept IP 192.168.217.98 using the tcp protocol via port 80 to SERVER B 

Run these following commands in SERVER A

    echo "1" > /proc/sys/net/ipv4/ip_forward

  > Make SERVER A has the ability to routing packets

  >

    iptables -t nat -I PREROUTING -p tcp -d 192.168.217.98 --dport 80 -j DNAT --to-destination 192.168.217.99:80

  > 192.168.217.98:80 converts to 192.168.217.99:80

  >

    iptables -t nat -I POSTROUTING -p tcp -d 192.168.217.99 --dport 80 -j SNAT --to-source 192.168.217.98
  
  > After converting, return 192.168.217.98 on the Web browser



