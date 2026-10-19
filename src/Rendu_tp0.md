# TP0

## 1. Configuration des routeurs 
### _Question 1_
#### 1.
Il y a 5 interfaces (on ne compte pas la première)

```
ip l / ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: host0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 4e:a3:9d:37:5a:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
3: eth0@if10: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 4a:5f:57:6e:aa:33 brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: eth1@if11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 62:c4:6e:01:07:e8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
5: eth2@if12: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether aa:62:5f:be:a8:5a brd ff:ff:ff:ff:ff:ff link-netnsid 0
6: eth3@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 2e:48:bf:d9:27:17 brd ff:ff:ff:ff:ff:ff link-netnsid 0

```

#### 2.
Mon ip est ==172.21.180.190/24==

```
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: host0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 4e:a3:9d:37:5a:a6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.21.180.190/24 metric 1024 brd 172.21.180.255 scope global dynamic host0
       valid_lft 2259sec preferred_lft 2259sec
3: eth0@if10: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 4a:5f:57:6e:aa:33 brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: eth1@if11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 62:c4:6e:01:07:e8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
5: eth2@if12: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether aa:62:5f:be:a8:5a brd ff:ff:ff:ff:ff:ff link-netnsid 0
6: eth3@if13: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 2e:48:bf:d9:27:17 brd ff:ff:ff:ff:ff:ff link-netnsid 0

```

#### 3.
Mon routage :

```
ip r
default via 172.21.180.1 dev host0 proto dhcp src 172.21.180.190 metric 1024 
172.21.0.60 via 172.21.180.1 dev host0 proto dhcp src 172.21.180.190 metric 1024 
172.21.0.61 via 172.21.180.1 dev host0 proto dhcp src 172.21.180.190 metric 1024 
172.21.0.203 via 172.21.180.1 dev host0 proto dhcp src 172.21.180.190 metric 1024 
172.21.0.204 via 172.21.180.1 dev host0 proto dhcp src 172.21.180.190 metric 1024 
172.21.180.0/24 dev host0 proto kernel scope link src 172.21.180.190 metric 1024 
172.21.180.1 dev host0 proto dhcp scope link src 172.21.180.190 metric 1024 
```

#### 4.
Le seul lien qui est disponible est mon host0 (regarder mon ip a)

#### 5.
Toutes mes adresses MAC:

* 4e:a3:9d:37:5a:a6
* 4a:5f:57:6e:aa:33 
* 62:c4:6e:01:07:e8
* aa:62:5f:be:a8:5a
* 2e:48:bf:d9:27:17

#### 6.
Sur mon S1 :

```
ip addr add 192.168.24.101/24 dev eth3
```

Sur mon S2 : 

```
ip addr add 192.168.24.102/24 dev eth3
```

#### 7.
Sur mon S1 :

```
ip link set eth3 up
ping 192.168.24.102
```

Sur mon S2 : 

```
ip link set eth3 up
ping 192.168.24.101
```

#### 8.

Sur mon S1 (cela marche aussiavec le S2) :

```
ping 192.168.24.102
```
Sur Silverblue :

```
vm-tcpdump S1 eth3
```

## 2. Routage Statique 
### _Question 1:_ Situation 1
#### 1. 
S1 : 

```
ip link set eth0 up // on met en up une interface qui nous servira de communication
ip addr add 172.24.1.1/24 dev eth0 \\ Warning : ne pas oublier le mask
```

S2 :
 
```
ip link set eth1 up
ip addr add 172.24.2.1/24 dev eth1
```

R1 :

```
ip link set eth0 up 
ip link set eth1 up
ip addr add 172.24.1.254/24 dev eth0
ip addr del 172.24.2.254/24 dev eth1
```
#### 2.

On ne peut communiquer qu'avec les interfaces qui possédent le même ethernet

#### 3.
S1 :

```
ip r add 172.24.2.0/24 via 172.24.1.254 dev eth0 
```

###  4.
S1 :

```
cat /proc/sys/net/ipv4/ip_forward
sysctl net.ipv4.ip_forward
echo 1 > /proc/sys/net/ipv4/ip_forward // elle oblige le forwading
sysctl net.ipv4.ip_forward=1
```

#### 5.

S2 : 

```
ip r a 172.24.1.0/24 via 172.24.2.254 dev eth1
```


### _Question 2:_ Situation 2

#### 1.

S1 :

```
ip link set eth0 up
ip addr add 172.24.1.1/24 dev eth0
ip r a 10.1.1.0/24 via 172.24.1.254 dev eth0
ip r a 10.1.2.0/24 via 172.24.1.254 dev eth0
ip r a 172.24.3.0/24 via 172.24.1.254 dev eth0
ip r a 172.24.2.0/24 via 172.24.1.254 dev eth0
```

S2 : 

```
ip link set eth2 up 
ip addr add 172.24.2.1/24 dev eth2
```

S3 : 

```
ip link set eth0 up
ip addr add 172.24.3.1/24 dev eth0
```

R1 :

```
ip link set eth0 up 
ip link set eth1 up
ip addr add 172.24.1.254/24 dev eth0
ip addr add 10.24.1.1/24 dev eth1
```

R2 :

```
ip link set eth1 up 
ip addr add 10.24.1.2/24 dev eth1
ip link set eth2 up
ip addr add 172.24.2.254/24 dev eth2
ip link set eth3 up
ip addr add 10.24.2.2/24 dev eth3
```

R3 :

```
ip link set eth3 up
ip addr add 10.24.2.3/24 dev eth3
ip link set eth0 up
ip addr add 172.24.3.254/24 dev eth0
```

*note: je n'ai pas mis tout le routage (flemme), mais c'est globalement la même chose que le S1*

## 3. Routage Dynamique 
 
### _Question 3:_ Situation 3









