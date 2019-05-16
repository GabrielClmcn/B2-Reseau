# TP 3

## Sommaire

I. Manipulation de switches et de VLAN
    1. Mise en place du lab
    2. Configuration des VLANs

II. Manipulation simple de routeurs
    1. Mise en place du lab
    2. Configuration du routage statique
     
III. Mise en place d'OSPF
    1. Mise en place du lab
    2. Configuration de OSPF

IV. Lab Final
Annexe 1 : NAT dans GNS3

---

## I. Manipulation de switches et de VLAN

### 1. Mise en place du lab

**> Vérification :**

Nom de domaines sur toutes les machines :

c1 :

        $ cat /etc/hostname
        client1.lab1.tp3

c2 :

        $ cat /etc/hostname
        client2.lab1.tp3

c3 :

        $ cat /etc/hostname
        client3.lab1.tp3


Toutes les machines doivent pouvoir se `ping` :

En partant du principes que si d'un côté un ping se fait l'autre répond pong

ping c1 vers c2 :

        $ ping 10.1.1.2
        PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=1.20 ms
        64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=3.95 ms
        64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=3.92 ms
        64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=4.52 ms

        --- 10.1.1.2 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3002ms
        rtt min/avg/max/mdev = 1.208/3.405/4.528/1.293 ms

ping c1 vers c3 :

        $ ping 10.1.1.3
        PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
        64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=4.55 ms
        64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=2.36 ms
        64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=2.42 ms
        64 bytes from 10.1.1.3: icmp_seq=4 ttl=64 time=3.47 ms

        --- 10.1.1.3 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3005ms
        rtt min/avg/max/mdev = 2.362/3.204/4.558/0.899 ms

ping c2 vers c3 :

        $ ping 10.1.1.3
        PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
        64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=4.85 ms
        64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=4.76 ms
        64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=11.5 ms
        64 bytes from 10.1.1.3: icmp_seq=4 ttl=64 time=5.29 ms

        --- 10.1.1.3 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3008ms
        rtt min/avg/max/mdev = 4.762/6.605/11.511/2.840 ms

### 2. Configuration des VLANs

**> Vérifications :**

c1 ping c3 :
        
        $ ping 10.1.1.3
        PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
        64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=1.58 ms
        64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=4.78 ms
        64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=3.86 ms
        64 bytes from 10.1.1.3: icmp_seq=4 ttl=64 time=4.07 ms

        --- 10.1.1.3 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3004ms
        rtt min/avg/max/mdev = 1.584/3.578/4.789/1.201 ms

traceroute c1 vers c3 :

    $ sudo traceroute -I 10.1.1.3
    traceroute to 10.1.1.3 (10.1.1.3), 30 hops max, 60 byte packets
    1  10.1.1.3 (10.1.1.3)  8.417 ms  8.318 ms  8.530 ms


ping c1 vers c2 :

        $ ping 10.1.1.2
        PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        From 10.1.1.1 icmp_seq=1 Destination Host Unreachable
        From 10.1.1.1 icmp_seq=2 Destination Host Unreachable
        From 10.1.1.1 icmp_seq=3 Destination Host Unreachable
        From 10.1.1.1 icmp_seq=4 Destination Host Unreachable

        --- 10.1.1.2 ping statistics ---
        4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3005ms
        pipe 4

ping c3 vers c2 :

        $ ping 10.1.1.2
        PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        From 10.1.1.3 icmp_seq=1 Destination Host Unreachable
        From 10.1.1.3 icmp_seq=2 Destination Host Unreachable
        From 10.1.1.3 icmp_seq=3 Destination Host Unreachable
        From 10.1.1.3 icmp_seq=4 Destination Host Unreachable

        --- 10.1.1.2 ping statistics ---
        4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3002ms
        pipe 4

Les VLAN sont configurés.

---

## II. Manipulation simple de routeurs

### 1. Mise en place du lab

**> Vérification :**

Les clients et serveurs peuvent joindre leurs gateways respectives :
c1 ping r1 :

    R1#ping 10.2.1.10
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.2.1.10, timeout is 2 seconds:
    .....
    Success rate is 0 percent (0/5)

r1 ping c2 :

    R1#ping 10.2.1.11
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.2.1.11, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 36/40/52 ms



s1 ping r2 :

    PING 10.2.2.254 (10.2.2.254) 56(84) bytes of data.
    64 bytes from 10.2.2.254: icmp_seq=1 ttl=255 time=6.85 ms
    64 bytes from 10.2.2.254: icmp_seq=2 ttl=255 time=4.29 ms
    64 bytes from 10.2.2.254: icmp_seq=3 ttl=255 time=6.11 ms
    --- 10.2.2.254 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2001ms
    rtt min/avg/max/mdev = 4.295/5.754/6.851/1.074 ms


Les routeurs peuvent discuter entre eux :

r1 ping r2 :
 
    R1#ping 10.2.12.2
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 10.2.12.2, timeout is 2 seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/4 ms






### 2. Configuration du routage statique

**> Vérification :**

Ajout des routes lab2.net1 :
server1:

    10.2.1.0/24 via 10.2.2.254 dev enp0s3 proto static metric 100 

router2:

    10.2.1.0/24 [1/0] via 10.2.12.1


Ajout des routes lab2-net2 :

client2:

    10.2.2.0/24 via 10.2.1.254 dev enp0s3 proto static metric 100 


router1:

    10.2.2.0/24 [1/0] via 10.2.12.2


**> Vérification :**

-- c2 traceroute s1 :

    traceroute to 10.2.2.10 (10.2.2.10), 30 hops max, 60 byte packets
     1  10.2.1.254 (10.2.1.254)  13.429 ms  13.775 ms  13.660 ms
     2  * * *
     3  10.2.2.10 (10.2.2.10)  33.620 ms  33.600 ms  33.500 ms

**> Proposer une topologie :**
Afin que les 2 clients aient la même gateway, il suffit de connecter les 2 clients à un switch, qui lui même est connecté au router1.




---
## III. Mise en place d'OSPF

### 1. Mise en place du lab

**> Vérification :**

Les clients et serveurs peuvent joindre leurs gateways respectives :

-- ping c1 vers R4:

    PING 10.3.101.254 (10.3.101.254) 56(84) bytes of data.
    64 bytes from 10.3.101.254: icmp_seq=1 ttl=255 time=11.7 ms
    64 bytes from 10.3.101.254: icmp_seq=2 ttl=255 time=8.45 ms
    64 bytes from 10.3.101.254: icmp_seq=3 ttl=255 time=5.04 ms

    --- 10.3.101.254 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 5.048/8.404/11.709/2.720 ms


-- ping s1 vers R1:

    PING 10.3.102.254 (10.3.102.254) 56(84) bytes of data.
    64 bytes from 10.3.102.254: icmp_seq=1 ttl=255 time=6.59 ms
    64 bytes from 10.3.102.254: icmp_seq=2 ttl=255 time=13.8 ms
    64 bytes from 10.3.102.254: icmp_seq=3 ttl=255 time=5.43 ms

    --- 10.3.102.254 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2003ms
    rtt min/avg/max/mdev = 5.437/8.626/13.843/3.719 ms


Les routeurs peuvent discuter entre eux (point à point) : 

**>Exemples:**
-- Interfaces R1:

    R1#sh ip int br
    Interface                  IP-Address      OK? Method Status                Protocol
    FastEthernet0/0            10.3.100.1      YES NVRAM  up                    up      
    FastEthernet1/0            10.3.100.22     YES NVRAM  up                    up      
    FastEthernet2/0            10.3.102.254    YES NVRAM  up                    up 

-- Interfaces R4:

    R4#sh ip int br
    Interface                  IP-Address      OK? Method Status                Protocol
    FastEthernet0/0            10.3.100.13     YES NVRAM  up                    up      
    FastEthernet1/0            10.3.100.10     YES NVRAM  up                    up      
    FastEthernet2/0            10.3.101.254    YES manual up                    up 

### 2. Configuration de OSPF


**> Vérification :**

Tous les routeurs peuvent se joindre :

-- R1 Command [sh ip protocols]:

    R1#sh ip proto
    Routing Protocol is "ospf 1"
      Outgoing update filter list for all interfaces is not set
      Incoming update filter list for all interfaces is not set
      Router ID 10.3.102.254
      It is an area border router
      Number of areas in this router is 2. 2 normal 0 stub 0 nssa
      Maximum path: 4
      Routing for Networks:
        10.3.100.0 0.0.0.3 area 0
        10.3.100.20 0.0.0.3 area 0
        10.3.102.0 0.0.0.255 area 2
     Reference bandwidth unit is 100 mbps
      Routing Information Sources:
        Gateway         Distance      Last Update
        5.5.5.5              110      00:03:06
        4.4.4.4              110      00:03:06
        3.3.3.3              110      00:03:06
        2.2.2.2              110      00:03:06
      Distance: (default is 110)


-- R4 Command [sh ip protocols]:

    R4#sh ip proto
    Routing Protocol is "ospf 1"
      Outgoing update filter list for all interfaces is not set
      Incoming update filter list for all interfaces is not set
      Router ID 4.4.4.4
      It is an area border router
      Number of areas in this router is 2. 2 normal 0 stub 0 nssa
      Maximum path: 4
      Routing for Networks:
        10.3.100.8 0.0.0.3 area 0
        10.3.100.12 0.0.0.3 area 0
        10.3.101.0 0.0.0.255 area 1
     Reference bandwidth unit is 100 mbps
      Routing Information Sources:
        Gateway         Distance      Last Update
        6.6.6.6              110      00:06:48
        5.5.5.5              110      00:06:38
        2.2.2.2              110      00:06:38
        10.3.102.254         110      00:06:38
      Distance: (default is 110)


---

## IV. Lab Final


#### 1. Comporter plusieurs routeurs et switches Cisco.
              .-~~~-.
      .- ~ ~-(       )_ _
     /                     ~ -.
    |       NAT1 et NAT2       \
     \                         .'
       ~- . _____________ . -~
         + 0                0 +
         |                    |
         |                    |
         |                    |
         | 0.0            0.0 |
    +----+---+          +-----+--+
    |        |          |        |
    |   R3   |          |   R2   |
    |        |          |        |
    +--------+          +------+++
    1.1 || 1.0  \\     //  1.1 || 1.0
        ||       \\   //       ||
        ||        \\ //        ||
        ||          X          ||
        ||       //   \\       ||
        ||      //     \\      ||
    1.1 || 1.0 //       \\ 1.1 || 1.0
    +---++---+           +-----++-+
    |        |           |        |
    |   R1   |           |   R4   |
    |        |           |        |
    +----+---+           +--------+
    3.0  |                    | 3.0
         |                    |
         |                    |
         |                    |
    0    |                    |   0
    +----+---+           +----+---+
    |        | 1       1 |        |
    |   SW1  |-----------|   SW2  |
    |        |           |        |
    +----+---+           +----+---+
    2    |                    |   2
         |                    |
         |                    |
         |                    |
    1    |                    |   1
    +----+---+           +----+---+
    |        |           |        |
    |   cl1  |           |   s1   |
    |        |           |        |
    +--------+           +--------+

![Tableau d'adressage](https://github.com/GabrielClmcn/B2-Reseau/blob/master/tp/tp3/TableauDadressageTP3.JPG)