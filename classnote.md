# DevOps 2105

What's Devops?
- CI/CD
- Docker
- Kubernetes
- Cloud (AWS/Azure/GCP/...)

How much is dev and ops in devops?
- dev 80%
- ops 20%

What should I start learning?
- IAC
- CI/CD
- Networking
- Linux - Core Linux Networking
- Cloud Networking
- Software Engineering


#### TODO
Write a REST-API for all the tech stacks in any language.
Why? --> 
To understand how it's talking to backend????

What should a system admin learn?
- Jenkins
- Gitlab
- Dockerize
- Kubernetes
- Python
- JS

How to learn software engineering?
How to learn frontend and backend development?


## IAC - Infrastructure as Code
To automate the tasks -> Trainer is saying it's not possible to manage resources by clicking the buttons.
Then maybe same goes for commands in my opinion.

Rather it should be managed with codes

Network Namespace
## OSI Model
Please Do Not Throw Sausage Pizza Away

Application
Presentation
Session
Transport  - Port
Network    - IP
Data Link   - MAC
Physical 

### ARP - Address Resolution protocol
- ARP Request
- ARP Table

### Route table


# Interface 
eth0
wlan0
lo0 = Loopback interface - software interface. 127.0.0.1

MTU = Maximum transmission unit / maximum transfer unit.
mtu 1500 = MTU is 1500 bytes

`ifconfig` to check the interface.
`tcpdump` to check the network packets at the interface level.


 
DNS Name or A Record -- [mapping to] --> IP
DNS Query - What's the IP of facebook.com?
DNS Resolve - Get the IP of facebook.com 

```
ash@owl:~$ dig facebook.com

; <<>> DiG 9.16.1-Ubuntu <<>> facebook.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3892
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;facebook.com.			IN	A

;; ANSWER SECTION:
facebook.com.		174	IN	A	179.60.194.35

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sat Apr 24 20:42:04 +08 2021
;; MSG SIZE  rcvd: 57
```

```
ash@owl:~$ telnet 179.60.194.35 443
Trying 179.60.194.35...
Connected to 179.60.194.35.
Escape character is '^]'.
Test
2Connection closed by foreign host.
```


dns name resolve query from Linux

dig command
```
ash@owl:~$ dig localhost

; <<>> DiG 9.16.1-Ubuntu <<>> localhost
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48331
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;localhost.			IN	A

;; ANSWER SECTION:
localhost.		0	IN	A	127.0.0.1

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sat Apr 24 20:44:41 +08 2021
;; MSG SIZE  rcvd: 54

```

nslook command
```
ash@owl:~$ nslookup localhost
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	localhost
Address: 127.0.0.1
Name:	localhost
Address: ::1

ash@owl:~$ 

```

/etc/hosts
```
ash@owl:~$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	owl

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```


tcpdump of accessing facebook. Though the resolved IP of facebook is 179.60.194.35, when I am accessing the facebook it connects to different dns and different ip.

Only the tcpbump -A command gives the following edge hostname of facebook.

`edge-star-shv-02-kut2.facebook.com.https`

The dns query command gives the following output.

```
ash@owl:~$ dig edge-star-shv-02-kut2.facebook.com

; <<>> DiG 9.16.1-Ubuntu <<>> edge-star-shv-02-kut2.facebook.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23175
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;edge-star-shv-02-kut2.facebook.com. IN	A

;; ANSWER SECTION:
edge-star-shv-02-kut2.facebook.com. 2107 IN A	179.60.194.20

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sat Apr 24 23:14:21 +08 2021
;; MSG SIZE  rcvd: 79
```

Then I found that when my browser tries to connect and get data from facebook, it's different. 179.60.194.20

```
ash@owl:~$ sudo tcpdump -A -i wlo1 dst 179.60.194.20 and port 443
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlo1, link-type EN10MB (Ethernet), capture size 262144 bytes
23:10:01.573612 IP owl.56280 > edge-star-shv-02-kut2.facebook.com.https: Flags [P.], seq 304164220:304164270, ack 3280850813, win 719, options [nop,nop,TS val 1099823912 ecr 2571337942], length 50
E..f..@.@.U....
.<.......!-|...}...........
A..(.C......-.........1.A..`.ak..J........+....!L..#...a..
23:10:01.604835 IP owl.56280 > edge-star-shv-02-kut2.facebook.com.https: Flags [P.], seq 50:80, ack 1, win 719, options [nop,nop,TS val 1099823944 ecr 2571343377], length 30
E..R..@.@.U....
.<.......!-....}....p......
A..H.C........(........`....#.9..8.y..
23:10:01.605016 IP owl.56280 > edge-star-shv-02-kut2.facebook.com.https: Flags [F.], seq 80, ack 1, win 719, options [nop,nop,TS val 1099823944 ecr 2571343377], length 0
E..4..@.@.U....
.<.......!-....}.... 2.....

``` 
Number of ports in the OS is 2 ** 16.



# Commands to create Network Namspace and setting IPv6 467  iptables --list
  468  sudo iptables --list
  469  sudo ip route show
  470  sudo ip adr show
  471  sudo ip addr show
  472  sudo ip netns exec red sh
  473  sudo ip netns exec red bash
  474  sudo ip netns exec red sh
  475  ip 
  476  ip link
  477  ip link -statistics
  478  ip -statistics link
  479  ip -statistics addr show
  480  sudo ip netns exec red ip a
  481  sudo ip netns exec red ip addr sow
  482  sudo ip netns exec red ip addr show
  483  sudo ip link add veth-a type veth peer name veth-b
  484  sudo ip link show
  485  sudo ip link set veth-b netns red
  486  sudo ip addr show
  487  sudo ip link show
  488  sudo ip netns exec red ip addr show
  489  sudo ip link
  490  sudo ip netns list
  491  sudo ifconfig 
  492  sudo ip link show
  493  sudo ip addr add 192.168.1.1/24 dev veth-a
  494  sudo ip link show
  495  sudo ip addr show
  496  sudo ip netns exec red ip addr add 192.168.1.2/24 dev veth-b
  497  sudo ip netns exec red ip addr show
  498  sudo ip link -h
  499  sudo ip link help
  500  sudo ip link set veth-a up
  501  sudo ip addr show
  502  sudo ip netns exec red ip link set veth-b up
  503  sudo ip netns exec red ip addr show
  504  sudo ip addr show
  505  arp -a
  506  sudo arp -a
  507  sudo arp
  508  ip addr show
  509  history
