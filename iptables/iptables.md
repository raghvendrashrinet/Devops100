
## Install iptable 
`sudo dnf install iptables-nft-services -y`
#### enable and start
```
sudo systemctl enable iptables
sudo systemctl status iptables

sudo systemctl start iptables
sudo systemctl start ip6tables
```

### default rules

#### List tables
`sudo iptables -vnL INPUT --line-numbers`


```
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  226 17176 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT 144 packets, 16112 bytes)
 pkts bytes target     prot opt in     out     source               destination
```
#### Description
##### 1. Global Policies
   * Chain INPUT (policy ACCEPT): By default, the system will accept incoming packets unless they hit a blocking rule lower down in the list.
   * Chain OUTPUT (policy ACCEPT): Your server is allowed to send out any traffic it wants without restriction
##### 2. Line-by-Line Breakdown of the INPUT Chain
   * Line 1: **ACCEPT all** -- * * 0.0.0.0/0 0.0.0.0/0 state RELATED,ESTABLISHED
       - Meaning: Allows traffic for connections that are already open. For example, if you download a file or run a web update, the returning traffic is allowed back in. This line handles almost all active traffic on your server (226 packets processed so far).
   * Line 2: **ACCEPT icmp** -- * * 0.0.0.0/0 0.0.0.0/0
       - Meaning: Allows network diagnostic pings (ping) to reach your server.
  * Line 3: A**CCEPT all** -- lo * 0.0.0.0/0 0.0.0.0/0
       - Meaning: Allows absolute freedom for your server's apps to talk to each other locally over the loopback interface (lo / 127.0.0.1
  * Line 4: **ACCEPT tcp** -- * * 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:22
       - Meaning: Opens port 22 specifically for new Secure Shell (SSH) connections so you can remotely log into the terminal.
  * Line 5: **REJECT all** -- * * 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
       - Meaning: The Catch-All Block. Any incoming packet that did not match lines 1 through 4 hits this rule and gets actively rejected. This is why your port 8087 or 3004 traffic was failing.
   
##### 3. The FORWARD Chain
  * **REJECT all** -- * * 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
      - Meaning: Your server refuses to act as a router or gateway for other computers. It drops packets passing through it

### Example allow on LB to access web server on port 5001
**Add below rules on web server**

* 1. Allow hostlb(LB  host) to access port 5001
```
sudo iptables -I INPUT -p tcp -s 10.244.195.62 --dport 5001 -j ACCEPT
```

* 2. Block everyone else on port 5001
```
sudo iptables -I INPUT 2 -p tcp --dport 5001 -j REJECT --reject-with icmp-host-prohibited
```
* 3. Save the rules permanently
```
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```
