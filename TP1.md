# TP1 : Mise en jambes

Joseph SELVA B2 B

## I. Exploration locale en solo

### 1. Affichage d'informations sur la pile TCP/IP locale

**🌞 Affichez les infos des cartes réseau de votre PC**

ethernet

```
PS C:\Users\M.SELVA> ipconfig /all

Carte Ethernet Ethernet :

Adresse physique . . . . . . . . . . . : AC-9E-17-BF-94-9D
```

La carte ethernet n'est pas connecté au réseau et ne possède donc pas d'adresse ip. 

wifi :

```
PS C:\Users\M.SELVA> ipconfig /all

Carte réseau sans fil Wi-Fi :

 Adresse physique . . . . . . . . . . . : 94-08-53-54-46-6F
 
 Adresse IPv4. . . . . . . . . . . . . .: 10.33.19.78(préféré)
```

**🌞 Affichez votre gateway**

Gateway

```
PS C:\Users\M.SELVA> route print -4

          0.0.0.0          0.0.0.0     10.33.19.254      10.33.19.78     50
```

**🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

![](https://i.imgur.com/IPc5xAb.png)

Le gateway permet de communiqué avec d'autre réseau à l'extérieur du réseau local d'YNOV.

### 2. Modifications des informations

#### A. Modification d'adresse IP (part 1)

**🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :**

![](https://i.imgur.com/o4SLgGX.png)

**🌞 Il est possible que vous perdiez l'accès internet. Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.**

Il est possible de perdre internet car l'adresse ip choisi est déja utilisé.

## II. Exploration locale en duo

### 3. Modification d'adresse IP

**🌞Si vos PCs ont un port RJ45 alors y'a une carte réseau Ethernet associée :**

```
PS C:\Users\M.SELVA> ipconfig /all

Carte Ethernet Ethernet :

   Adresse IPv4. . . . . . . . . . . . . .: 192.168.30.1(préféré)
```

```
PS C:\Users\M.SELVA> ping 192.168.30.2

Envoi d’une requête 'Ping'  192.168.30.2 avec 32 octets de données :
Réponse de 192.168.30.2 : octets=32 temps<1ms TTL=128
Réponse de 192.168.30.2 : octets=32 temps<1ms TTL=128
Réponse de 192.168.30.2 : octets=32 temps<1ms TTL=128
Réponse de 192.168.30.2 : octets=32 temps<1ms TTL=128
```


```
PS C:\Users\M.SELVA> arp -a


Interface : 192.168.30.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  169.254.48.124        f0-2f-74-4d-0c-32     dynamique
  192.168.30.2          f0-2f-74-4d-0c-32     dynamique
  192.168.30.3          ff-ff-ff-ff-ff-ff     statique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```

### 4. Utilisation d'un des deux comme gateway

**🌞 pour tester la connectivité à internet on fait souvent des requêtes simples vers un serveur internet connu**

```
ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=23 ms TTL=54
```

**🌞 utiliser un traceroute ou tracert pour bien voir que les requêtes passent par la passerelle choisie (l'autre le PC)**

```
tracert 1.1.1.1

Détermination de l’itinéraire vers one.one.one.one [1.1.1.1]
avec un maximum de 30 sauts :

  1     1 ms     *        1 ms  LAPTOP-EIT6ULH9 [192.168.137.1]
  2     *        *        *     Délai d’attente de la demande dépassé.
  3     6 ms     9 ms     4 ms  10.33.19.254
  4     6 ms     5 ms     9 ms  137.149.196.77.rev.sfr.net [77.196.149.137]
```

### 5. Petit chat privé

**🌞 sur le PC serveur avec par exemple l'IP 192.168.1.1**

```
PS C:\Users\M.SELVA\Desktop\netcat-1.11> ncat -l -p 8888

Linux c'est trop trop bien
staline
```

**🌞 sur le PC client avec par exemple l'IP 192.168.1.2**
```
ncat 192.168.137.1 8888
libnsock ssl_init_helper(): OpenSSL legacy provider failed to load.

Linux c'est trop trop bien
staline
```

**🌞 pour aller un peu plus loin**

```
.\nc.exe  -l -p 8888 -s 192.168.30.2
j
jk
```

### 6. Firewall

**🌞 Autoriser les ping**

```
PS C:\Users\M.SELVA> ping 192.168.30.2

Envoi d’une requête 'Ping'  192.168.30.2 avec 32 octets de données :
Réponse de 192.168.30.2 : octets=32 temps=1 ms TTL=128
```

**🌞 Autoriser le traffic sur le port qu'utilise nc**

```
.\nc.exe  -l -p 8888 -s 192.168.30.2
```

```
netstat -a -b -n | select-string 8888

  TCP    192.168.30.2:8888      0.0.0.0:0              LISTENING
```


## III. Manipulations d'autres outils/protocoles côté client

### 1. DHCP

**🌞Exploration du DHCP, depuis votre PC**

ip du serveur DHCP

```
PS C:\Users\M.SELVA> ipconfig /all

   Serveur DHCP . . . . . . . . . . . . . : 10.33.19.254
```

date d'expiration :

```
PS C:\Users\M.SELVA> ipconfig /all

  Bail expirant. . . . . . . . . . . . . : jeudi 29 septembre 2022 08:13:28
```

### 2. DNS

**🌞 trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

```
PS C:\Users\M.SELVA> ipconfig /all

   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1
```
**🌞 utiliser, en ligne de commande l'outil nslookup (Windows, MacOS) ou dig (GNU/Linux, MacOS) pour faire des requêtes DNS à la main**
                                     
```
PS C:\Users\M.SELVA> nslookup google.com

Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:808::200e
          216.58.215.46
```

```
PS C:\Users\M.SELVA> nslookup ynov.com
Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    ynov.com
Addresses:  2606:4700:20::681a:ae9
          2606:4700:20::681a:be9
          2606:4700:20::ac43:4ae2
          172.67.74.226
          104.26.10.233
          104.26.11.233
```

Lorsque l'on fait un lookup sur le serveur google.com on ping via le dns 8.8.8.8 afin qu'il nous renvoient l'adresse ip du serveur. Et lorsque l'on le fait sur le serveur Ynov.com on passe aussi par le dns 8.8.8.8 car il s'agit du serveur dns fournit par le serveur dhcp. Ynov.com nous renvoient également plusieurs ip pour éviter la surcharge du réseau.

**déterminer l'adresse IP du serveur à qui vous venez d'effectuer ces requêtes**

l'adresse ip du serveur est 8.8.8.8.

```
PS C:\Users\M.SELVA> nslookup 78.74.21.21
Serveur :   dns.google
Address:  8.8.8.8

Nom :    host-78-74-21-21.homerun.telia.com
Address:  78.74.21.21
```

```
PS C:\Users\M.SELVA> nslookup 92.146.54.88
Serveur :   dns.google
Address:  8.8.8.8

*** dns.google ne parvient pas à trouver 92.146.54.88 : Non-existent domain
```

Lorsque l'on fait un lookup de 78.74.21.21 via le serveur dns 8.8.8.8 on nous renvoie la même adresse ip et le nom de domaine du serveur. Mais dans le cas de 92.146.54.88, le serveur ne pas ping car cette ip n'a pas de nom de domaine.

## IV. Wireshark

**🌞 utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :**

**ping**

![](https://i.imgur.com/7eIoVDI.png)

**netcat**

![](https://i.imgur.com/5RgC43h.png)

**requête DNS**

![](https://i.imgur.com/EXjiGOi.png)
