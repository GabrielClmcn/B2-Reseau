
# TP1 réseau
## 1. Mise en place

configure Adapter Manually, name: vboxnet1, IPv4 address: 10.1.1.1, IPv4 address: 255.255.255.0, no DHCP Server.

configure Adapter Manually, name: vboxnet2, IPv4 address: 10.1.2.1, IPv4 network mask: 255.255.255.252, no DHCP Server.

combien y a-t-il d'adresses disponibles dans un /24 ?
        256-2 = 254 : l’adresse de réseau et de broadcast ne sont pas utilisable.

Combien y a-t-il d'adresses disponibles dans un /30?
        4-2 = 2

Quelle est l'utilité d'un /30 ?
Seulement deux adresses disponible, celà nous permet de faire une interconnexion entre deux réseau.

### Configuration de la machine virtuelle:
On copie la configuration de notre carte enp0s8 vers enp0s9 :

    sudo cp /etc/sysconfig/network-scripts/ifcg-enp0s8 /etc/sysconfig/network-scripts/ifcg-enp0s9

config de enp0s8 :
vi /etc/sysconfig/network-scripts/ifcg-enp0s8

	TYPE=Ethernet BOOTPROTO=static NAME=enp0s8 DEVICE=enp0s8 ONBOOT=yes IPADDR=10.1.1.2 NETMASK=255.255.255.0

config de enp0s9 :
vi /etc/sysconfig/network-scripts/ifcg-enp0s9

	TYPE=Ethernet BOOTPROTO=static NAME=enp0s9 DEVICE=enp0s9 ONBOOT=yes IPADDR=10.1.2.2 NETMASK=255.255.255.252

On restart les interfaces pour rendre les nouvelles configurations actives :

	sudo ifdown enps0s8 && sudo ifdown enp0s9 sudo ifup enp0s8 && sudo ifup enp0s9

Changement du nom de domaine :

	vi /etc/hostnames
	client1.tp1.b2

 ## 2 Basics
### Routes

    ip r s
    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 101
    
Chacunes des lignes représente une route :

    sudo ip route del 10.1.2.2/30
    ping 10.1.2.2
        PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data. 64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.080 ms --- 10.1.2.2 ping statistics --- 1 packets transmitted, 1 received, 0% packet loss, time 1000ms rtt min/avg/max/mdev = 0.080/0.086/0.093/0.011 ms

    sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enps0s9
    ping 10.1.2.2
	    PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data. 64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.084 ms --- 10.1.2.2 ping statistics --- 1 packets transmitted, 1 received, 0% packet loss, time 1261ms rtt min/avg/max/mdev = 0.084/0.087/0.091/0.010 ms

    ip r s
	    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 10.1.2.0/30 via 10.1.2.2 dev enp0s9

 ## Table ARP

    ip n s
        10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 REACHABLE 10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE

    sudo ip neigh f all ip n s
        10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE

    ping 10.1.2.1 ip n s
        10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE 10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 REACHABLE

Nous avons vider la table ARP et ensuite nous les avons rajouté lors d'un ping où la machine a fait une requête ARP et a reçu une réponse. Ce qui a permit à la table de se mettre à jour.

Capture Réseau :

Sur deux terminaux (T = terminal) :

    ssh gabriel@10.1.1.2

    ssh gabriel@10.1.2.2
	    sudo ip n f all


T1:

	sudo tcpdump -i enp0s8 -w ping.pcap

T2: 

	ping 10.1.2.1
T1 : arrêt de la capture de packet

	10 packets received by filter 0 packets dropped by kernel
	scp gabriel@10.1.1.2:/home/gabriel/ping.pcap /home/gabriel/etudes/b2/reseau/tp1/

voir le fichier : ping.pcap


# II. Communication simple entre deux machines
### Mise en place

configuration de la machine virtuelle:

  fichier de config de la carte connecté au réseau net1:
  
	  vi /etc/sysconfig/network-scripts/ifcg-enp0s8
        TYPE=Ethernet BOOTPROTO=static NAME=enp0s8 DEVICE=enp0s8 ONBOOT=yes IPADDR=10.1.1.3 NETMASK=255.255.255.0

Changement du nom de domaine :

	vi /etc/hostnames
	client1.tp1.b2

### Basics
ping et ARP (c1 = client1; c2 = client2)

   c1 = 10.1.2.2
   c2 = 10.1.1.3
    
	  c2: ping 10.1.1.3 ip n s
        10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 STALE 10.1.1.3 dev enp0s8 lladdr 08:00:27:6c:53:a8 REACHABLE 10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE

        c2: 
	        sudo ip n f all
        
nouveau shell a fermer après la commande: ssh 10.1.1.2 sudo ip neigh f all

	        c2: sudo tcpdump -i enp0s8 -w ping.pcap

   ouvrir un autre shell t2 sur c2

        c2.t2: ping -c 4 10.1.1.3
        c2: arreter la capture de packet

        10 packets received by filter 0 packets dropped by kernel

        scp gabriel@10.1.1.3:/home/gabriel/ping-2.pcap /home/gabriel/etudes/b2/reseau/tp1/

ping-2.pcap

---

#### UDP

- Ouvrir 2 terminaux en connexion ssh sur chaque client

- Sur chaque machine ajouter le port udp

sudo firewall-cmd --zone=public --add-port=8888/udp --permanent

sudo firewall-cmd --reload

-- Client1 | Term1:
 nc -u -l 8888

-- Client2 | Term1: 
nc -u 10.1.1.2 8888

-- Client1 | Term2:
 ss -unp

    Recv-Q Send-Q Local Address:Port Peer Address:Port
    0 0 10.1.1.2:8888 10.1.1.3:43163 users:(("nc",pid=2462,fd=4))



-- Client2 | Term2:
 ss -unp

    Recv-Q Send-Q Local Address:Port Peer Address:Port
    0 0 10.1.1.3:43163 10.1.1.2:8888 users:(("nc",pid=3022,fd=3))

- Client1 | Term3 = 10.1.2.2

-- Client1 | Term1:
 nc -u -l 8888

-- Client2 | Term2:
 nc -u 10.1.1.2 8888

-- Client1 | Term3:
 sudo tcpdump -i enp0s8 -w nc-udp.pcap

-- Client1 | Term1:
test

-- arrêter la capture de packet
  scp 10.1.1.2:/home/gabriel/nc-udp.pcap /home/gabriel/etudes/b2/reseau/tp1/



c-udp.pcap
---
#### TCP

- Client1 | Term1 = 10.1.1.2

- Client2 | Term1 = 10.1.1.3

- Term2 = deuxième shell de t1

- Client1 | Term3 = 10.1.2.2

--  Sur chaque machine ajouter le port TCP

sudo firewall-cmd --zone=public --add-port=8888/tcp --permanent && sudo firewall-cmd --reload

-- Client1 | Term1:
 nc -l 8888

-- Client2 | Term1:
 nc 10.1.1.2 8888

-- Client1 | Term2: 
ss -tnp

    State Recv-Q Send-Q Local Address:Port Peer Address:Port
    ESTAB 0 0 10.1.2.2:22 10.1.2.1:46006
    ESTAB 0 0 10.1.1.2:8888 10.1.1.3:57706 users:(("nc",pid=5451,fd=5))
    ESTAB 0 0 10.1.1.2:22 10.1.1.1:36416

-- Client1 | Term2: 
ss -tnp

    State Recv-Q Send-Q Local Address:Port Peer Address:Port
    ESTAB 0 0 10.1.1.3:22 10.1.1.1:38068
    ESTAB 0 0 10.1.1.3:22 10.1.1.1:42006
    ESTAB 0 0 10.1.1.3:57706 10.1.1.2:8888 users:(("nc",pid=4671,fd=3))



-- Client1 | Term3:
 sudo tcpdump -i enp0s8 -w firewall.pcap

-- Client1 | Term1:
 nc -l 8888

-- Client2 | Term2:
 nc 10.1.1.2 8888

-- fermer netcat

-- fermer la capture de paquet

scp 10.1.1.2:/home/gabriel/nc-tcp.pcap /home/gabriel/etudes/b2/reseau/tp1/

  nc-tcp.pcap

---
### Firewall

-- Client1 | Term1:

sudo firewall-cmd --remove-port=8888/udp --permanent

sudo firewall-cmd --reload

-- Client1 | Term3:

sudo tcpdump -i enp0s8 -w nc-udp.pcap

-- Client1 | Term1:

nc -u -l 8888

-- Arrêter la capture du paquet

scp 10.1.1.2:/home/gabriel/firewall.pcap /home/gabiel/etudes/b2/reseau/tp1/

 Client1:

systemctl -w net.ipv4.ip_forward=1

    net.ipv4.ip_forward = 1



Client2:

sudo ip route add 10.1.2.0/30 via 10.1.1.2 dev enp0s8

ping 10.1.2.2

    PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
    64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.933 ms
    --- 10.1.2.2 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 1001ms
    rtt min/avg/max/mdev = 0.543/0.738/0.933/0.195 ms

 ip show route

    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.3 metric 101
    10.1.2.0/30 via 10.1.1.2 dev enp0s8

