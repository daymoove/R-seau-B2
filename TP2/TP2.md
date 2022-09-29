# TP2 : Ethernet, IP, et ARP

## I. Setup IP

**🌞 Mettez en place une configuration réseau fonctionnelle entre les deux machines**

```PS C:\WINDOWS\system32> netsh interface ipv4 set address name="Ethernet" static 192.168.1.1 255.255.255.192 192.168.1.2
```

```
PS C:\WINDOWS\system32> ipconfig /all

   Adresse IPv4. . . . . . . . . . . . . .: 192.168.1.1(tentative)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.192
   Passerelle par défaut. . . . . . . . . : 192.168.1.2
```

adresse ip : 192.168.1.1 /26
adresse ip2 : 192.168.1.2 /26

Masque : 255.255.255.192

adresse de réseau : 192.168.1.0

adresse de brodcast : 192.168.1.63


**🌞 Prouvez que la connexion est fonctionnelle entre les deux machines**

```
PS C:\WINDOWS\system32> ping 192.168.1.2

Envoi d’une requête 'Ping'  192.168.1.2 avec 32 octets de données :
Réponse de 192.168.1.2 : octets=32 temps=2 ms TTL=128
```

**🌞 Wireshark it**


#### Déterminez, grâce à Wireshark, quel type de paquet ICMP est envoyé par ping :

Le ping que l'on envoie est du type Echo ping (request) et le pong que l'on reçois est du type Echo ping (reply).


**🦈 PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP**

**Lien ici**

## II. ARP my bro

**🌞 Check the ARP table**

Utilisez une commande pour afficher votre table ARP:

```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.1.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  192.168.1.2           7c-10-c9-ac-92-58     dynamique
  192.168.1.63          ff-ff-ff-ff-ff-ff     statique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```

Déterminez la MAC de votre binome depuis votre table ARP :

```
PS C:\WINDOWS\system32> arp -a

  192.168.1.2           7c-10-c9-ac-92-58     dynamique
```

Déterminez la MAC de la gateway de votre réseau:

```
PS C:\WINDOWS\system32> arp -a

    10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

**🌞 Manipuler la table ARP**

Utilisez une commande pour vider votre table ARP :

```
PS C:\WINDOWS\system32> netsh interface IP delete arpcache
```

Prouvez que ça fonctionne en l'affichant et en constatant les changements :

```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.1.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
```

Ré-effectuez des pings, et constatez la ré-apparition des données dans la table ARP :

```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.1.1 --- 0xe
  Adresse Internet      Adresse physique      Type
  192.168.1.2           7c-10-c9-ac-92-58     dynamique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

**🌞 Wireshark it**

source :

Sender IP address: 192.168.1.2
Sender MAC address: ASUSTekC_ac:92:58 (7c:10:c9:ac:92:58)

destinataire :

Target IP address: 192.168.1.1
Target MAC address: ASUSTekC_bf:94:9d (ac:9e:17:bf:94:9d)

La premiere source est l'ordinateur envoyant la requete arp. La deuxieme est la reponse de l'autre ordinateur.

**🦈 PCAP qui contient les trames ARP**

**Lien arp**

## III. DHCP you too my brooo

**🌞 Wireshark it**

1er trame:

adresse source : 0.0.0.0
adresse de destination : 255.255.255.255
ip utilisé : 10.33.19.78

2er trame:

adresse source : 10.33.19.254
adresse de destination : 10.33.19.78
ip passerelle: 10.33.19.254
dns : 8.8.8.8 / 8.8.4.4 / 1.1.1.1

3er trame:

adresse source : 0.0.0.0
adresse de destination : 255.255.255.255

4er trame:

adresse source : 10.33.19.254
adresse de destination : 10.33.19.78

**🦈 PCAP qui contient l'échange DORA**

**lien dhcp**

## IV. Avant-goût TCP et UDP

**🌞 Wireshark it**

Le pc se connecte à l'ip 162.159.130.234 et au port 443

**🦈 PCAP qui contient un extrait de l'échange qui vous a permis d'identifier les infos**

yt(./yt.pcapng)