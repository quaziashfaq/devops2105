# Docker Network Internals Basics

Install docker in ubuntu.
```
$ sudo apt update
$ sudo apt -y upgrade
$ sudo apt -y install docker.io
```

We spun up a nginx docker container.

```
$ sudo docker run --publish 8080:80 nginx
```

Here the port 8080 is the external port in the host.
The 80 is the internal port of the nginx container.

We want to know what's the new container's IP assigned by the Docker bridge.

First, we need to know about the running container.

```
$ sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
34dc63020981   nginx     "/docker-entrypoint.…"   29 minutes ago   Up 29 minutes   0.0.0.0:8080->80/tcp   brave_tesla
```
Here it says that ontainer ID is ```34dc63020981``` but it does not tell what's the IP set to the machine.


2nd, we run the below command to know about existing bridge info.

```
$ sudo docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
741477f2f860   bridge    bridge    local
a356785b2230   host      host      local
d4ec2e99bfb3   none      null      local
```
These are the 3 default bridges are created when we install docker. Now we want to know details about bridge network.

```
$ sudo docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "741477f2f86081f7edc89f7d880f61d3fd58b73a6805af31a2ebbc4cde676e71",
        "Created": "2021-06-15T20:33:32.002583261+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "34dc63020981c44ef844815a850f1723bb25b6134d7716f53bab402aec70db17": {
                "Name": "brave_tesla",
                "EndpointID": "600dc5bfdf8726b85ff75523a095dc904f0449c330cd29479d68a7609a1c5906",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

Here we got to know that the network subnet ip and it's gateway IP of the bridge is 
```
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
 
```

If run the command ```ip``` command, we also find that the above gateway ip is actually one of the virtual NIC's IP of the host. And it's called **docker0**.
Because, the host machine works as a gateway for the containers / guests

```
$ sudo ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:4c:d6:96 brd ff:ff:ff:ff:ff:ff
    inet 192.168.39.128/24 brd 192.168.39.255 scope global dynamic ens33
       valid_lft 931sec preferred_lft 931sec
    inet6 fe80::20c:29ff:fe4c:d696/64 scope link 
       valid_lft forever preferred_lft forever
3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:4c:d6:a0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.226.128/24 brd 192.168.226.255 scope global dynamic ens38
       valid_lft 1173sec preferred_lft 1173sec
    inet6 fe80::20c:29ff:fe4c:d6a0/64 scope link 
       valid_lft forever preferred_lft forever
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:d4:06:3e:d1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d4ff:fe06:3ed1/64 scope link 
       valid_lft forever preferred_lft forever
9: veth87dbf45@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 1a:ad:8f:c1:52:d4 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::18ad:8fff:fec1:52d4/64 scope link 
       valid_lft forever preferred_lft forever
```

Here, I take out the docker0 info for better understanding

```
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:d4:06:3e:d1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d4ff:fe06:3ed1/64 scope link 
       valid_lft forever preferred_lft forever
```


Now we find the IP of the container.
```
        "Containers": {
            "34dc63020981c44ef844815a850f1723bb25b6134d7716f53bab402aec70db17": {
                "Name": "brave_tesla",
                "EndpointID": "600dc5bfdf8726b85ff75523a095dc904f0449c330cd29479d68a7609a1c5906",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
```

Match the container ID and the name of the container.
Then we can see that the IP of the container is ```172.16.0.2/16```.

Now we are going to ping the container and we will check the packets that are going through this network.

```
$ ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.165 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.082 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.070 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.095 ms
64 bytes from 172.17.0.2: icmp_seq=5 ttl=64 time=0.097 ms
^C
--- 172.17.0.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4094ms
rtt min/avg/max/mdev = 0.070/0.101/0.165/0.033 ms
ash@ubuntu20:~$ 

```

In another window we ran ```tcpdump``` to capture the packets.

```
$ sudo tcpdump -nnvvS net 172.17.0.0/16
tcpdump: listening on docker0, link-type EN10MB (Ethernet), capture size 262144 bytes
01:24:23.958342 ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 172.17.0.2 tell 172.17.0.1, length 28
01:24:23.958369 ARP, Ethernet (len 6), IPv4 (len 4), Reply 172.17.0.2 is-at 02:42:ac:11:00:02, length 28
01:24:23.958378 IP (tos 0x0, ttl 64, id 40329, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.0.1 > 172.17.0.2: ICMP echo request, id 4, seq 1, length 64
01:24:23.958414 IP (tos 0x0, ttl 64, id 24607, offset 0, flags [none], proto ICMP (1), length 84)
    172.17.0.2 > 172.17.0.1: ICMP echo reply, id 4, seq 1, length 64
01:24:24.970512 IP (tos 0x0, ttl 64, id 40568, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.0.1 > 172.17.0.2: ICMP echo request, id 4, seq 2, length 64
01:24:24.970569 IP (tos 0x0, ttl 64, id 24718, offset 0, flags [none], proto ICMP (1), length 84)
    172.17.0.2 > 172.17.0.1: ICMP echo reply, id 4, seq 2, length 64
01:24:25.994424 IP (tos 0x0, ttl 64, id 40723, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.0.1 > 172.17.0.2: ICMP echo request, id 4, seq 3, length 64
01:24:25.994468 IP (tos 0x0, ttl 64, id 24916, offset 0, flags [none], proto ICMP (1), length 84)
    172.17.0.2 > 172.17.0.1: ICMP echo reply, id 4, seq 3, length 64
01:24:27.018511 IP (tos 0x0, ttl 64, id 40906, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.0.1 > 172.17.0.2: ICMP echo request, id 4, seq 4, length 64
01:24:27.018571 IP (tos 0x0, ttl 64, id 25160, offset 0, flags [none], proto ICMP (1), length 84)
    172.17.0.2 > 172.17.0.1: ICMP echo reply, id 4, seq 4, length 64
01:24:28.042491 IP (tos 0x0, ttl 64, id 41017, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.0.1 > 172.17.0.2: ICMP echo request, id 4, seq 5, length 64
01:24:28.042546 IP (tos 0x0, ttl 64, id 25264, offset 0, flags [none], proto ICMP (1), length 84)
    172.17.0.2 > 172.17.0.1: ICMP echo reply, id 4, seq 5, length 64
01:24:29.098413 ARP, Ethernet (len 6), IPv4 (len 4), Request who-has 172.17.0.1 tell 172.17.0.2, length 28
01:24:29.098528 ARP, Ethernet (len 6), IPv4 (len 4), Reply 172.17.0.1 is-at 02:42:d4:06:3e:d1, length 28
^C
14 packets captured
14 packets received by filter
0 packets dropped by kernel

```
Here we can see that ping is going through the L2 bridge from 172.17.0.1 to 172.17.0.2.


Now how it goes through from outside world to container? The host port 8080 is mapped to the container and its port 80. The iptables of the host manages this mapping. Iptable has a rule that says when any packet is coming to host port 8080, it will route the packet to container IP and port. So effectively it is removing the previous destination port **8080** information and adding the container IP and port 80 as the destination IP and port.

Let's check the iptables' NAT rule.

```
ash@ubuntu20:~$ sudo iptables -t nat -L -v -n
[sudo] password for ash:
Chain PREROUTING (policy ACCEPT 2 packets, 120 bytes)
 pkts bytes target     prot opt in     out     source               destination
    6   348 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 2 packets, 120 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 408 packets, 44749 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 408 packets, 44749 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.17.0.2:80
```

Here, the last line is telling the packet to send it to ip 172.17.0.2:80. So the packet gets sent to host routing table.

```
$ sudo ip route show
default via 192.168.39.2 dev ens33 proto dhcp src 192.168.39.128 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.39.0/24 dev ens33 proto kernel scope link src 192.168.39.128
192.168.39.2 dev ens33 proto dhcp scope link src 192.168.39.128 metric 100
192.168.226.0/24 dev ens38 proto kernel scope link src 192.168.226.128
ash@ubuntu20:~$

```
The 2nd line: ```172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1``` is telling that the packet having the descination IP 172.17.0.2 or destination network 172.17.0.0/16 will go through the interface docker0 (172.17.0.1).

```
$ sudo ip addr show
... ...
5: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:d4:06:3e:d1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d4ff:fe06:3ed1/64 scope link
       valid_lft forever preferred_lft forever
```

And the docker bridge is telling us which network this 'docker0' belongs to and the connected containers.

```
$ sudo docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "741477f2f86081f7edc89f7d880f61d3fd58b73a6805af31a2ebbc4cde676e71",
        "Created": "2021-06-15T20:33:32.002583261+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },

```


Let's check by connecting to 127.0.0.1 and port 8080 from the host machine.

```
$ telnet localhost 8080
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
a
HTTP/1.1 400 Bad Request
Server: nginx/1.21.0
Date: Wed, 16 Jun 2021 01:03:01 GMT
Content-Type: text/html
Content-Length: 157
Connection: close

<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx/1.21.0</center>
</body>
</html>
Connection closed by foreign host.
ash@ubuntu20:~$ 

```

Running tcpdump to capture and analyze the packets.
```
ash@ubuntu20:~$ sudo tcpdump -nS -i lo src port 8080 or dst port 8080 
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 262144 bytes
09:03:01.267297 IP6 ::1.54320 > ::1.8080: Flags [S], seq 517773132, win 65476, options [mss 65476,sackOK,TS val 3122474240 ecr 0,nop,wscale 7], length 0
09:03:01.267321 IP6 ::1.8080 > ::1.54320: Flags [R.], seq 0, ack 517773133, win 0, length 0
09:03:01.268132 IP 127.0.0.1.49560 > 127.0.0.1.8080: Flags [S], seq 2331250003, win 65495, options [mss 65495,sackOK,TS val 2173264456 ecr 0,nop,wscale 7], length 0
09:03:01.268159 IP 127.0.0.1.8080 > 127.0.0.1.49560: Flags [S.], seq 1105789991, ack 2331250004, win 65483, options [mss 65495,sackOK,TS val 2173264457 ecr 2173264456,nop,wscale 7], length 0
09:03:01.268181 IP 127.0.0.1.49560 > 127.0.0.1.8080: Flags [.], ack 1105789992, win 512, options [nop,nop,TS val 2173264457 ecr 2173264457], length 0
```
The above are SYN-SYNACK-ACK packets. It's the handshaking between sender and receiver applications to open a connection. Check the flags 'S'.

The below packets are the acutal HTTP packets. Check the TCP flags 'P'
```
09:03:01.980638 IP 127.0.0.1.49560 > 127.0.0.1.8080: Flags [P.], seq 2331250004:2331250007, ack 1105789992, win 512, options [nop,nop,TS val 2173265169 ecr 2173264457], length 3: HTTP
09:03:01.980692 IP 127.0.0.1.8080 > 127.0.0.1.49560: Flags [.], ack 2331250007, win 512, options [nop,nop,TS val 2173265169 ecr 2173265169], length 0
09:03:01.981537 IP 127.0.0.1.8080 > 127.0.0.1.49560: Flags [P.], seq 1105789992:1105790301, ack 2331250007, win 512, options [nop,nop,TS val 2173265170 ecr 2173265169], length 309: HTTP: HTTP/1.1 400 Bad Request
09:03:01.981563 IP 127.0.0.1.49560 > 127.0.0.1.8080: Flags [.], ack 1105790301, win 510, options [nop,nop,TS val 2173265170 ecr 2173265170], length 0
```

The below are the packets to close the tcp connection.
```
09:03:01.982135 IP 127.0.0.1.8080 > 127.0.0.1.49560: Flags [F.], seq 1105790301, ack 2331250007, win 512, options [nop,nop,TS val 2173265170 ecr 2173265170], length 0
09:03:01.982250 IP 127.0.0.1.49560 > 127.0.0.1.8080: Flags [F.], seq 2331250007, ack 1105790302, win 512, options [nop,nop,TS val 2173265171 ecr 2173265170], length 0
09:03:01.982269 IP 127.0.0.1.8080 > 127.0.0.1.49560: Flags [.], ack 2331250008, win 512, options [nop,nop,TS val 2173265171 ecr 2173265171], length 0
```



