
# TP2 - CCNA2

## I. Mise en place du lab

- 1 Création des VMs et adressage IP :
client1 :
ping c1 vers r1:

		$ ping 10.2.1.254
		PING 10.2.1.254 (10.2.1.254) 56(84) bytes of data.
		64 bytes from 10.2.1.254: icmp_seq=1 ttl=64 time=0.444 ms
		64 bytes from 10.2.1.254: icmp_seq=2 ttl=64 time=0.868 ms
		64 bytes from 10.2.1.254: icmp_seq=3 ttl=64 time=0.746 ms
		64 bytes from 10.2.1.254: icmp_seq=4 ttl=64 time=0.862 ms

		--- 10.2.1.254 ping statistics ---
		4 packets transmitted, 4 received, 0% packet loss, time 3003ms
		rtt min/avg/max/mdev = 0.444/0.730/0.868/0.172 ms

ping r1 vers r2:

        $ ping 10.2.12.3
	    PING 10.2.12.3 (10.2.12.3) 56(84) bytes of data.
        64 bytes from 10.2.12.3: icmp_seq=1 ttl=64 time=0.426 ms
        64 bytes from 10.2.12.3: icmp_seq=2 ttl=64 time=1.01 ms
        64 bytes from 10.2.12.3: icmp_seq=3 ttl=64 time=0.891 ms
        64 bytes from 10.2.12.3: icmp_seq=4 ttl=64 time=0.791 ms

        --- 10.2.12.3 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3005ms
        rtt min/avg/max/mdev = 0.426/0.780/1.015/0.222 ms

ping r2 vers s1:

        $ ping -c 4 10.2.2.10
        PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
        64 bytes from 10.2.2.10: icmp_seq=1 ttl=64 time=0.462 ms
        64 bytes from 10.2.2.10: icmp_seq=2 ttl=64 time=0.491 ms
        64 bytes from 10.2.2.10: icmp_seq=3 ttl=64 time=0.791 ms
        64 bytes from 10.2.2.10: icmp_seq=4 ttl=64 time=0.595 ms

        --- 10.2.2.10 ping statistics ---
        4 packets transmitted, 4 received, 0% packet loss, time 3001ms
        rtt min/avg/max/mdev = 0.462/0.584/0.791/0.132 ms

- 2 Routage statique :

	-- Activation permanente de l'ipv4 forwarding sur router1 et router2 :

		$ sudo vi /etc/sysctl.conf
		net.ipv4.conf.all.forwarding=1
	
	-- Ajout des routes statiques :
	- router 1  vers server 1:
		Créer un fichier route-enp0s9

			$ cat /etc/sysconfig/network-scripts/route-enp0s9
			10.2.2.0/24 via 10.2.12.3 dev enp0s9
			$ sudo service network restart
			
	- router 2 :
		Créer un fichier route-enp0s9

			$ cat /etc/sysconfig/network-scripts/route-enp0s9
			10.2.1.0/24 via 10.2.12.2 dev enp0s9
			$ sudo service network restart

	- client 1 :
		Créer un fichier route-enp0s8

			$ cat /etc/sysconfig/network-scripts/route-enp0s8
			10.2.2.0/24 via 10.2.1.254 dev enp0s8
			$ sudo service network restart

	- server 1 :
		Créer un fichier route-enp0s8

			$ cat /etc/sysconfig/network-scripts/route-enp0s8
			10.2.1.0/24 via 10.2.2.254 dev enp0s8
			$ sudo service network restart


	--Ping client1 vers serveur1  :

		$ ping 10.2.2.10
		PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
		64 bytes from 10.2.2.10: icmp_seq=1 ttl=62 time=0.778 ms
		64 bytes from 10.2.2.10: icmp_seq=2 ttl=62 time=1.89 ms
		64 bytes from 10.2.2.10: icmp_seq=3 ttl=62 time=1.13 ms
		64 bytes from 10.2.2.10: icmp_seq=4 ttl=62 time=0.854 ms

		--- 10.2.2.10 ping statistics ---
		4 packets transmitted, 4 received, 0% packet loss, time 3005ms
		rtt min/avg/max/mdev = 0.778/1.166/1.897/0.444 ms

	--Ping router1 vers serveur1  :

		$ ping 10.2.2.10
		PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
		
		--- 10.2.2.10 ping statistics ---
		4 packets transmitted, 0 received, 100% packet loss, time 3000ms

    router1 ne peut pas ping server1, car il n'a pas la route nécessaire pour communiquer.

---
- 3 Visualisation du routage avec Wireshark :

    wireshark files :
    [capture_net12.pcap](./capture_net12.pcap)
    [capture_net2.pcap](./capture_net2.pcap)




---
## II. NAT et services d'infra

- 1 Mise en place du NAT : 

    --Ping 8.8.8.8 via router2
    (client1 et server1 ping aussi)
    
            $ PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
            64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=19.7 ms
            64 bytes from 8.8.8.8: icmp_seq=2 ttl=61 time=21.2 ms
            64 bytes from 8.8.8.8: icmp_seq=3 ttl=61 time=21.2 ms
            64 bytes from 8.8.8.8: icmp_seq=4 ttl=61 time=19.2 ms
            
            --- 8.8.8.8 ping statistics ---
            4 packets transmitted, 4 received, 0% packet loss, time 3008ms
            rtt min/avg/max/mdev = 19.277/20.359/21.256/0.893 ms


- 2 Serveur DHCP :

        $ cat /etc/hostname
        client2.net.b2

        $ dhclient -v
        Internet Systems Consortium DHCP Client 4.2.5
        Copyright 2004-2013 Internet Systems Consortium.
        All rights reserved.
        For info, please visit https://www.isc.org/software/dhcp/

        Listening on LPF/enp0s8/08:00:27:9f:e9:c6
        Sending on   LPF/enp0s8/08:00:27:9f:e9:c6
        Sending on   Socket/fallback
        DHCPREQUEST on enp0s8 to 255.255.255.255 port 67 (xid=0x7ddae8cf)
        DHCPNAK from 192.168.56.100 (xid=0x7ddae8cf)
        bound to 10.2.1.50 -- renewal in 252 seconds.

        $ ip a | grep 'inet '
        inet 127.0.0.1/8 scope host lo
        inet 10.2.1.50/24 brd 10.2.1.255 scope global dynamic enp0s8

        $ ip r s
        default via 10.2.1.254 dev enp0s8 proto static metric 100
        10.2.1.0/24 dev enp0s8 kernel scope link src 10.2.1.50


- 3 Serveur NTP :
    
    --Commande chronyc sources:
    
            >210 Number of sources = 4
        MS Name/IP address         Stratum Poll Reach LastRx Last sample               
        ===============================================================================
        ^+ ns358812.ip-91-121-154.eu     2   6    17     5  -4911us[-4143us] +/-   90ms
        ^+ pob01.aplu.fr                 2   6    17     5  +1004us[+1776us] +/-   43ms
        ^* herbrand.noumicek.cz          2   6    17     4   +768us[+1531us] +/-   43ms
        ^+ ns3.stoneartprod.xyz          2   6    17     5  +1311us[+2088us] +/-   47ms

    --Commande chronyc tracking:
    
            >Reference ID    : 33FFC594 (ovh.sqlserver.express)
        Stratum         : 3
        Ref time (UTC)  : Sun Mar 10 18:46:28 2019
        System time     : 0.000000184 seconds fast of NTP time
        Last offset     : -0.000760409 seconds
        RMS offset      : 0.000760409 seconds
        Frequency       : 33.461 ppm slow
        Residual freq   : -21.684 ppm
        Skew            : 0.389 ppm
        Root delay      : 0.046656054 seconds
        Root dispersion : 0.003042061 seconds
        Update interval : 1.9 seconds
        Leap status     : Normal


    
- 4 Serveur Web :

    -- Status du server web:
    
            $ systemctl status nginx
           >Active: active (running) since dim. 2019-03-10 20:07:57 CET; 4s ago
          Process: 1811 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
          Process: 1809 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
          Process: 1807 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
          Main PID: 1813 (nginx)
           CGroup: /system.slice/nginx.service
                   ├─1813 nginx: master process /usr/sbin/nginx
                   └─1815 nginx: worker process
        mars 10 20:07:57 server1.net2.b2 systemd[1]: Starting The nginx HTTP and rev....
        mars 10 20:07:57 server1.net2.b2 nginx[1809]: nginx: the configuration file ...k
        mars 10 20:07:57 server1.net2.b2 nginx[1809]: nginx: configuration file /etc...l
        mars 10 20:07:57 server1.net2.b2 systemd[1]: Failed to read PID from file /r...t
        mars 10 20:07:57 server1.net2.b2 systemd[1]: Started The nginx HTTP and reve....
        Hint: Some lines were ellipsized, use -l to show in full.


    --Commande ss -altnp4:

        State      Recv-Q Send-Q Local Address:Port               Peer Address:Port
        LISTEN     0      128          *:80                       *:*                   users:(("nginx",pid=1815,fd=6),("nginx",pid=1813,fd=6))
        LISTEN     0      128          *:22                       *:*                   users:(("sshd",pid=947,fd=3))
        LISTEN     0      100    127.0.0.1:25                     *:*                   users:(("master",pid=1047,fd=13))



    -- curl du server :

        $ curl 10.2.2.10
        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

        <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
            <head>
                <title>Test Page for the Nginx HTTP Server on Fedora</title>
                <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
                <style type="text/css">
                    /*<![CDATA[*/
                    body {
                        background-color: #fff;
                        color: #000;
                        font-size: 0.9em;
                        font-family: sans-serif,helvetica;
                        margin: 0;
                        padding: 0;
                    }
                    :link {
                        color: #c00;
                    }
                    :visited {
                        color: #c00;
                    }
                    a:hover {
                        color: #f50;
                    }
                    h1 {
                        text-align: center;
                        margin: 0;
                        padding: 0.6em 2em 0.4em;
                        background-color: #294172;
                        color: #fff;
                        font-weight: normal;
                        font-size: 1.75em;
                        border-bottom: 2px solid #000;
                    }
                    h1 strong {
                        font-weight: bold;
                        font-size: 1.5em;
                    }
                    h2 {
                        text-align: center;
                        background-color: #3C6EB4;
                        font-size: 1.1em;
                        font-weight: bold;
                        color: #fff;
                        margin: 0;
                        padding: 0.5em;
                        border-bottom: 2px solid #294172;
                    }
                    hr {
                        display: none;
                    }
                    .content {
                        padding: 1em 5em;
                    }
                    .alert {
                        border: 2px solid #000;
                    }

                    img {
                        border: 2px solid #fff;
                        padding: 2px;
                        margin: 2px;
                    }
                    a:hover img {
                        border: 2px solid #294172;
                    }
                    .logos {
                        margin: 1em;
                        text-align: center;
                    }
                    /*]]>*/
                </style>
            </head>

            <body>
                <h1>Welcome to <strong>nginx</strong> on Fedora!</h1>

                <div class="content">
                    <p>This page is used to test the proper operation of the
                    <strong>nginx</strong> HTTP server after it has been
                    installed. If you can read this page, it means that the
                    web server installed at this site is working
                    properly.</p>

                    <div class="alert">
                        <h2>Website Administrator</h2>
                        <div class="content">
                            <p>This is the default <tt>index.html</tt> page that
                            is distributed with <strong>nginx</strong> on
                            Fedora.  It is located in
                            <tt>/usr/share/nginx/html</tt>.</p>

                            <p>You should now put your content in a location of
                            your choice and edit the <tt>root</tt> configuration
                            directive in the <strong>nginx</strong>
                            configuration file
                            <tt>/etc/nginx/nginx.conf</tt>.</p>

                        </div>
                    </div>

                    <div class="logos">
                        <a href="http://nginx.net/"><img
                            src="nginx-logo.png" 
                            alt="[ Powered by nginx ]"
                            width="121" height="32" /></a>

                        <a href="http://fedoraproject.org/"><img 
                            src="poweredby.png" 
                            alt="[ Powered by Fedora ]" 
                            width="88" height="31" /></a>
                    </div>
                </div>
            </body>
        </html>
