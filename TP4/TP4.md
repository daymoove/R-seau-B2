# TP4 : TCP, UDP et services réseau

## I. First steps

**🌞 Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**, **🌞 Demandez l'avis à votre OS**

Discord :

ip : 162.159.130.235

port : 443

port local : 1024

[discord](./discordTCP.pcapng)


```
PS C:\WINDOWS\system32> netstat -ab

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.19.78:1024       162.159.130.235:https  ESTABLISHED
 [Discord.exe]
 ```
 
steam :

ip : 185.25.182.77

port : 27022

port local : 31304

[steam](./steam.pcapng)

```
PS C:\WINDOWS\system32> netstat -ab

Connexions actives
[...]
  TCP    192.168.1.29:26532     185.25.182.77:27023    ESTABLISHED
 [Steam.exe]
```

google :

ip : 104.244.42.194

port : 443

port local : 29679

[google](./google.pcapng)


```
PS C:\WINDOWS\system32> netstat -ab

Connexions actives
[...]
  TCP    192.168.1.29:29679     104.244.42.194:https   ESTABLISHED
 [chrome.exe]
```


genshin impact :

ip : 47.254.179.15

port :22101

port local : 49248 

[genshin_impact](./genshinimpact.pcapng)

```
PS C:\WINDOWS\system32>  netstat -abp udp

Connexions actives
[...]
  UDP    0.0.0.0:49248          *:*
 [System]
```


amazon prime :


ip : 52.94.235.74

port :443

port local : 2098 

[amazonprime](./amazonprime.pcapng)

```
PS C:\WINDOWS\system32> netstat -ab

Connexions actives
  TCP    192.168.1.29:2098      52.94.235.74:https     ESTABLISHED
 [PrimeVideo.exe]
```

## II. Mise en place

### 1. SSH

**🌞 Examinez le trafic dans Wireshark**

Le service ssh utilise le protocole tcp car celui ci est plus sécurisé que le protocole udp, et de ne pas perdre de trame entre les deux machines lors d'échange entre elles.

[ssh](./ssh.pcapng)


**🌞 Demandez aux OS**

```
PS C:\WINDOWS\system32> netstat -ab

Connexions actives
  TCP    192.168.56.1:29267     192.168.56.102:22      ESTABLISHED
```

```
[daymoove@localhost ~]$ ss

tcp      ESTAB     0         0              192.168.56.102:ssh               192.168.56.1:29267
```

### 2. NFS

**🌞 Wireshark it !**

```
[daymoove@localhost /]$ sudo tcpdump -i enp0s8 -c 10 -w nfs.pcap not port 22
```

[nfs](./nfs.pcapng)

**nfs**

Port serveur : 760

**🌞 Demandez aux OS**


serveur :
```
[daymoove@localhost ~]$ ss

tcp     ESTAB    0        0                         192.168.56.102:nfs             192.168.56.103:krbupdate
```

client :

```
[daymoove@localhost ~]$ ss

tcp     ESTAB    0         0                         192.168.56.103:krbupdate       192.168.56.102:nfs
```

### 3. DNS

**🌞 Utilisez une commande pour effectuer une requête DNS depuis une des VMs**

[dns](./dns.pcapng)

ip serveur dns : 10.0.2.15
port serveur dns : 49788
