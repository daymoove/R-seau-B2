# TP3 : On va router des trucs

## I. ARP

### 1. Echange ARP

**🌞Générer des requêtes ARP**

ping :
```
[daymoove@localhost ~]$ ping 10.3.1.11

PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.728 ms
```

arp Marcel :
```
[daymoove@localhost ~]$ ip neigh show

10.3.1.254 dev enp0s3  FAILED
10.3.1.11 dev enp0s3 lladdr 08:00:27:5a:9c:1e REACHABLE
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:3b DELAY
```

arp John :

```
[daymoove@localhost ~]$ ip neigh show
10.3.1.12 dev enp0s3 lladdr 08:00:27:7f:f8:3e REACHABLE
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:3b REACHABLE
10.3.1.254 dev enp0s3  FAILED
```

MAC Marcel table arp John :

```
[daymoove@localhost ~]$ ip neigh show

10.3.1.11 dev enp0s3 lladdr 08:00:27:5a:9c:1e REACHABLE
```

MAC John table arp Marcel :

```
[daymoove@localhost ~]$ ip neigh show

10.3.1.12 dev enp0s3 lladdr 08:00:27:7f:f8:3e STALE
```

Mac de Marcel depuis Marcel :
```
[daymoove@localhost ~]$ ip a

link/ether 08:00:27:7f:f8:3e brd ff:ff:ff:ff:ff:ff
```

### 2. Analyse de trames

**🌞Analyse de trames**

```
[daymoove@localhost ~]$ sudo tcpdump -i enp0s3 -c 10 -w ping-pong.pcap not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

vider table arp :

```
[daymoove@localhost ~]$ sudo ip neigh flush all
```

[Arp]((./tp3_arp.pcapng))

## II. Routage

### 1. Mise en place du routage

**🌞Activer le routage sur le noeud router**

```
[daymoove@localhost ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```

**🌞Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping**

```
[daymoove@John ~]$ sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
[...]

10.3.1.11/24 via 10.3.1.254 dev enp0s3

```

john :
```
[daymoove@John ~]$ ping 10.3.2.12

PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.968 ms
```

Marcel :
```
[daymoove@Marcel ~]$ ping 10.3.1.11

PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=0.864 ms
```

### 2. Analyse de trames

**🌞Analyse des échanges ARP**

Videz les tables ARP des trois noeuds :

```
[daymoove@John ~]$ sudo ip neigh flush all
```

Effectuez un ping de john vers marcel :

```
[daymoove@John ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.15 ms
```

Regardez les tables ARP des trois noeuds :

John :
```
[daymoove@John ~]$ ip neigh show
10.3.1.254 dev enp0s3 lladdr 08:00:27:6a:a4:23 REACHABLE
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:3b REACHABLE
```

Marcel :
```
[daymoove@Marcel ~]$ ip neigh show
10.3.2.254 dev enp0s3 lladdr 08:00:27:1c:1f:9b REACHABLE
10.3.2.1 dev enp0s3 lladdr 0a:00:27:00:00:40 REACHABLE
```

Routeur :
```
[daymoove@Routeur ~]$ ip neigh show
10.3.1.11 dev enp0s3 lladdr 08:00:27:5a:9c:1e STALE
10.3.2.12 dev enp0s8 lladdr 08:00:27:7f:f8:3e STALE
10.3.1.1 dev enp0s3 lladdr 0a:00:27:00:00:3b DELAY
```

Essayez de déduire un peu les échanges ARP qui ont eu lieu :

La machine John contact d'abord le routeur afin de pouvoir se connecter au reseau de Marcel afin de pouvoir recuperer son adresse MAC pour pouvoir contacter la machine Marcel via un ping.

Ecrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames utiles pour l'échange :

|ordre      |type trame |IP source   |MAC source       |IP destination|MAC destination|
|   1       |ARP| 10.3.2.254 |08:00:27:1c:1f:9b| 10.3.2.12    |08:00:27:7f:f8:3e| 
|   2       |ARP| 10.3.2.12  |08:00:27:7f:f8:3e|10.3.2.254    |08:00:27:1c:1f:9b|
|   3       |ICMP      | 10.3.2.254 |08:00:27:1c:1f:9b|10.3.2.254    |08:00:27:7f:f8:3e|
|   4       |ICMP      | 10.3.2.12  |08:00:27:7f:f8:3e|10.3.2.254    |08:00:27:1c:1f:9b|

[Routage_Marcel]((./TP3_routage_marcel.pcap))

### 3. Accès internet

**🌞Donnez un accès internet à vos machines**

Ajoutez une carte NAT en 3ème inteface sur le router pour qu'il ait un accès internet :

```
[daymoove@Routeur ~]$ ip a

4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:be:78:98 brd ff:ff:ff:ff:ff:ff
    inet 10.0.4.15/24 brd 10.0.4.255 scope global dynamic noprefixroute enp0s9
       valid_lft 86024sec preferred_lft 86024sec
    inet6 fe80::d2ba:31cd:a165:3ac7/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

Ajoutez une route par défaut à john et marcel :

John :

```
[daymoove@John ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=19.1 ms
```

Marcel :

```
[daymoove@Marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=17.2 ms
```

Donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utilise :

Vérifiez que vous avez une résolution de noms qui fonctionne avec dig

```
[daymoove@John ~]$ dig google.com

; <<>> DiG 9.16.23-RH <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51878
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             260     IN      A       216.58.214.174

;; Query time: 23 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Oct 03 17:28:21 CEST 2022
;; MSG SIZE  rcvd: 55

```

Puis avec un ping vers un nom de domaine :

```
[daymoove@John ~]$ ping google.com
PING google.com (216.58.214.174) 56(84) bytes of data.
64 bytes from par10s42-in-f14.1e100.net (216.58.214.174): icmp_seq=1 ttl=113 time=20.4 ms
```

**🌞Analyse de trames**

|ordre| type trame| IP source | MAC source      | IP destination| MAC destination|
| 1   | ICMP      |10.3.1.11  |08:00:27:5a:9c:1e|8.8.8.8        |08:00:27:6a:a4:23|
| 2   | ICMP     |8.8.8.8    |08:00:27:6a:a4:23|10.3.1.11      |08:00:27:5a:9c:1e|

[Routage_internet]((./TP3_routage_internet.pcap))

## III. DHCP

### 1. Mise en place du serveur DHCP

**🌞Sur la machine john, vous installerez et configurerez un serveur DHCP**

Installation du serveur sur john :
```
[daymoove@John ~]$ sudo dnf install dhcp-server -y
[...]
Installed:
  dhcp-common-12:4.4.2-15.b1.el9.noarch                      dhcp-server-12:4.4.2-15.b1.el9.x86_64

Complete!
```

```
[daymoove@John ~]$ sudo vim /etc/dhcp/dhcpd.conf
```

```
[daymoove@John ~]$ sudo cat /etc/dhcp/dhcpd.conf

default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.1.0 netmask 255.255.255.0 {
  range 10.3.1.12 10.3.1.250;
  option subnet-mask 255.255.255.0;

}
```

```
[daymoove@John ~]$ sudo firewall-cmd --add-port=67/udp --permanent
success
```

```
[daymoove@John ~]$ sudo systemctl daemon-reload
[daymoove@John ~]$ sudo systemctl enable --now dhcpd
```

```
[daymoove@John ~]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2022-10-03 18:23:46 CEST; 14s ago
[...]
```

Faites lui récupérer une IP en DHCP à l'aide de votre serveur :
```
[daymoove@localhost ~]$ sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

```
[daymoove@localhost ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
NAME=enp0s3
DEVICE=enp0s3

BOOTPROTO=dhcp
ONBOOT=yes
```

```
[daymoove@localhost ~]$ ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5e:85:27 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 789sec preferred_lft 789sec
    inet6 fe80::a00:27ff:fe5e:8527/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

**🌞Améliorer la configuration du DHCP**

ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :

Une route par défaut :

```
[daymoove@John ~]$ sudo cat /etc/dhcp/dhcpd.conf
[...]
  option routers 10.3.1.254;
}
```

Un serveur DNS à utiliser :

```
[daymoove@John ~]$ sudo cat /etc/dhcp/dhcpd.conf
[...]
option domain-name-servers 8.8.8.8;
```

Récupérez de nouveau une IP en DHCP sur Bob pour tester :

```
[daymoove@BOB ~]$ sudo ip addr del 10.3.1.12/24 dev enp0s3
```

vérifier avec une commande qu'il a récupéré son IP :
```
[daymoove@BOB ~]$ ip a

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5e:85:27 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 879sec preferred_lft 879sec
    inet6 fe80::a00:27ff:fe5e:8527/64 scope link
       valid_lft forever preferred_lft forever
```

vérifier qu'il peut ping sa passerelle

```
[daymoove@BOB ~]$ ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.354 ms
```

vérifier la présence de la route avec une commande

```
[daymoove@BOB ~]$ ip route show
default via 10.3.1.254 dev enp0s3 proto dhcp src 10.3.1.12 metric 100
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.12 metric 100
```

vérifier que la route fonctionne avec un ping vers une IP

```
[daymoove@BOB ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.867 ms
```

vérifier avec la commande dig que ça fonctionne :

```
[daymoove@BOB ~]$ dig google.com

; <<>> DiG 9.16.23-RH <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 28161
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             202     IN      A       142.250.179.78

;; Query time: 20 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Oct 03 19:02:21 CEST 2022
;; MSG SIZE  rcvd: 55
```

vérifier un ping vers un nom de domaine :

```
[daymoove@BOB ~]$ ping google.com
PING google.com (142.250.179.78) 56(84) bytes of data.
64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=1 ttl=114 time=17.2 ms
```

### 2. Analyse de trames

**🌞Analyse de trames**

[dhcp]((./tp3_dhcp.pcap))