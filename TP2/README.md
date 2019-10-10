# TP2

## I. Simplest setup

* Topologie 
![topo](images/topologie.PNG)

* Communication

Ping PC1 --> PC2
![ping](images/ping-pc1-pc2.PNG)

Le protocole utilisé par le `ping` est `ICMP`

L'échange ARP vue sur Wireshark correspond aux tables ARP des machines
```
PC-1> show arp

00:50:79:66:68:01  10.2.1.2 expires in 112 seconds
```

```
PC-2> show arp

00:50:79:66:68:00  10.2.1.1 expires in 110 seconds
```

Le switch sert à faire passer l'information il n'a pas besoin d'ip.


## II. More switch

![ping](images/topologie2.PNG)
PC1 = PC3   /   PC2 = PC4   /   PC3 = PC5

* Communication


```
PC-3> ping 10.2.2.2
84 bytes from 10.2.2.2 icmp_seq=1 ttl=64 time=4.706 ms
84 bytes from 10.2.2.2 icmp_seq=2 ttl=64 time=2.039 ms
84 bytes from 10.2.2.2 icmp_seq=3 ttl=64 time=4.482 ms
84 bytes from 10.2.2.2 icmp_seq=4 ttl=64 time=1.520 ms
84 bytes from 10.2.2.2 icmp_seq=5 ttl=64 time=3.536 ms
```

```
PC-4> ping 10.2.2.3
84 bytes from 10.2.2.3 icmp_seq=1 ttl=64 time=12.107 ms
84 bytes from 10.2.2.3 icmp_seq=2 ttl=64 time=1.999 ms
84 bytes from 10.2.2.3 icmp_seq=3 ttl=64 time=1.745 ms
84 bytes from 10.2.2.3 icmp_seq=4 ttl=64 time=1.568 ms
84 bytes from 10.2.2.3 icmp_seq=5 ttl=64 time=1.789 ms
```

```
PC-5> ping 10.2.2.1
84 bytes from 10.2.2.1 icmp_seq=1 ttl=64 time=1.522 ms
84 bytes from 10.2.2.1 icmp_seq=2 ttl=64 time=1.253 ms
84 bytes from 10.2.2.1 icmp_seq=3 ttl=64 time=1.700 ms
84 bytes from 10.2.2.1 icmp_seq=4 ttl=64 time=1.458 ms
84 bytes from 10.2.2.1 icmp_seq=5 ttl=64 time=1.303 ms
```

```
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6802    DYNAMIC     Et0/1
   1    0050.7966.6803    DYNAMIC     Et0/0
   1    0050.7966.6804    DYNAMIC     Et0/2
   1    aabb.cc00.0300    DYNAMIC     Et0/0
   1    aabb.cc00.0400    DYNAMIC     Et0/0
   1    aabb.cc00.0410    DYNAMIC     Et0/2
Total Mac Addresses for this criterion: 6
```
Vlan: Nous n'avons pas modifier le Vlan donc le Vlan par defaut est 1.