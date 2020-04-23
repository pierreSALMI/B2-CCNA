# TP4 : Buffet à volonté

## Global

### Config
Les configurations sont là:
* [Routeur](/tp4/conf/R1.txt)
* [Client-sw1](conf/client-sw1.txt)
* [Client-sw2](conf/client-sw2.txt)
* [Client-sw3](conf/client-sw3.txt)
* [Infra-sw1](conf/infra-sw1.txt)

### DHCP
```
guest1> ip dhcp
    DDORA IP 10.5.20.84/24 GW 10.5.20.254

guest1> show ip

    NAME        : guest1[1]
    IP/MASK     : 10.5.20.84/24
    GATEWAY     : 10.5.20.254
    DNS         : 10.5.30.10
    DHCP SERVER : 10.5.20.253
```

### Ping
* Guest1 ---> Admin1
```
guest1> ping 10.5.10.11
    10.5.10.11 icmp_seq=1 timeout
    84 bytes from 10.5.10.11 icmp_seq=2 ttl=63 time=15.559 ms
    84 bytes from 10.5.10.11 icmp_seq=3 ttl=63 time=13.397 ms
    84 bytes from 10.5.10.11 icmp_seq=4 ttl=63 time=15.539 ms
    84 bytes from 10.5.10.11 icmp_seq=5 ttl=63 time=17.593 ms
```

* Guest1 ---> Internet
```
guest1> ping 1.1.1.1
    84 bytes from 1.1.1.1 icmp_seq=1 ttl=54 time=30.307 ms
    84 bytes from 1.1.1.1 icmp_seq=2 ttl=54 time=28.250 ms
    84 bytes from 1.1.1.1 icmp_seq=3 ttl=54 time=22.620 ms
    84 bytes from 1.1.1.1 icmp_seq=4 ttl=54 time=23.921 ms
    84 bytes from 1.1.1.1 icmp_seq=5 ttl=54 time=29.459 ms
```

* Guest1 ---> Guest2 (with DNS)
```
guest1> ping client2.tp5.b2
    client2.tp5.b2 resolved to 10.5.20.12
    84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.612 ms
    84 bytes from 10.5.20.12 icmp_seq=2 ttl=64 time=0.400 ms
    84 bytes from 10.5.20.12 icmp_seq=3 ttl=64 time=0.423 ms
    84 bytes from 10.5.20.12 icmp_seq=4 ttl=64 time=0.416 ms
    84 bytes from 10.5.20.12 icmp_seq=5 ttl=64 time=0.770 ms
```


## 2. Sécurité attack & defense : spoofing, VLAN attack

### A. Offensive

#### ARP Spoofing

Table ARP Client (avant)
```
guest1> show arp

ca:01:05:b0:00:1d  10.5.20.254 expires in 109 seconds
```

```
[saru@attack ~]$ sudo sysctl net.ipv4.ip_nonlocal_bind=1

[saru@attack ~]$ sudo arping -c 5 -U -s 10.5.20.5 -I enp0s3 10.5.20.84
ARPING 10.5.20.84 from 10.5.20.5 enp0s3
Unicast reply from 10.5.20.84 [00:50:79:66:68:01]  2.390ms
Unicast reply from 10.5.20.84 [00:50:79:66:68:01]  2.518ms
Unicast reply from 10.5.20.84 [00:50:79:66:68:01]  1.808ms
Unicast reply from 10.5.20.84 [00:50:79:66:68:01]  10.742ms
Sent 5 probes (1 broadcast(s))
Received 4 response(s)
```

Table ARP Client (apres)
```
guest1> show arp

ca:01:05:b0:00:1d  10.5.20.254 expires in 80 seconds
08:00:27:45:84:4f  10.5.20.5 expires in 116 seconds
```

##### DHCP spoofing

Configuration de base client1:
```
guest1> show ip

    NAME        : guest1[1]
    IP/MASK     : 10.5.20.84/24
    GATEWAY     : 10.5.20.254
    DNS         : 10.5.30.10
    DHCP SERVER : 10.5.20.253
    DHCP LEASE  : 594, 600/300/525
    DOMAIN NAME : tp5.b2
```

On va utiliser notre attaquant comme serveur dhcp car il est déjà connecté.
La plage d'adressage sera de 1 à 10 au lieu de 10 à 100.
On spam aussi le `DHCP` de requête ARP pour avoir la priorité avec le DHCP de l'attaquant. (je ne sais pas si ça sert vraiment)

On relance :
```
guest1> ip dhcp
    DDORA IP 10.5.20.2/24 GW 10.5.20.254

guest1> show ip

    NAME        : guest1[1]
    IP/MASK     : 10.5.20.2/24
    GATEWAY     : 10.5.20.254
    DNS         : 10.5.20.66
    DHCP SERVER : 10.5.20.66
    DHCP LEASE  : 594, 600/300/525
    DOMAIN NAME : getHacked.b2
```

#### DNS Spoofing
On va rediriger tous les pings vers l'attaquant
```
guest1> ping client2.tp5.b2
    client2.tp5.b2 resolved to 10.5.20.66
    84 bytes from 10.5.20.12 icmp_seq=1 ttl=64 time=0.637 ms
    84 bytes from 10.5.20.12 icmp_seq=2 ttl=64 time=0.548 ms
    84 bytes from 10.5.20.12 icmp_seq=3 ttl=64 time=0.495 ms
    84 bytes from 10.5.20.12 icmp_seq=4 ttl=64 time=0.690 ms
    84 bytes from 10.5.20.12 icmp_seq=5 ttl=64 time=0.553 ms
```

### B. Defensive

#### ARP Spoofing
```
client-sw3#show ip source binding
    MacAddress          IpAddress        Lease(sec)  Type           VLAN  Interface
    ------------------  ---------------  ----------  -------------  ----  --------------------
    00:50:79:66:68:01   10.5.20.86       infinite    static          20    Ethernet0/0
    08:00:27:4D:A0:6D   10.5.20.253      infinite    static          20    Ethernet0/1
    Total number of bindings: 2
```

#### DHCP Spoofing

Configuration guest1 (avant)
```
guest1> ip dhcp
    DORA IP 10.5.20.2/24 GW 10.5.20.254

    guest1> show ip

    NAME        : guest1[1]
    IP/MASK     : 10.5.20.2/24
    GATEWAY     : 10.5.20.254
    DNS         : 10.5.20.66
    DHCP SERVER : 10.5.20.66
    DHCP LEASE  : 596, 600/300/525
    DOMAIN NAME : getHacked.b2
```

Ajout du `dhcp` au switch
```
client-sw3#show ip dhcp snooping

    DHCP snooping trust/rate is configured on the following Interfaces:

    Interface                    Trusted     Rate limit (pps)
    ------------------------     -------     ----------------
    Ethernet0/1                  yes         unlimited
```

Configuration guest1 (apres)
```
guest1> ip dhcp
    DDORA IP 10.5.20.87/24 GW 10.5.20.254

    guest1> show ip

    NAME        : guest1[1]
    IP/MASK     : 10.5.20.87/24
    GATEWAY     : 10.5.20.254
    DNS         : 10.5.30.10
    DHCP SERVER : 10.5.20.253
    DHCP LEASE  : 594, 600/300/525
    DOMAIN NAME : tp5.b2
```

#### Messing up with VLANs

Déjà expliqué dans le sujet.
Du coup j'ai décidé de la désactiver et de faire IP source guard.
Je te jure.



C'était sympa à faire j'essairai d'utiliser scapy la prochaine fois.