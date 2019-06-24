Zadanie 1
---------

![zadanie 1](zadanie-1.svg)

1. Zaprojektuj oraz przygotuj prototyp rozwiązania z wykorzystaniem oprogramowania ``VirtualBox`` lub podobnego. 
Zaproponuj rozwiązanie spełniające poniższe wymagania:
   * Usługodawca zapewnia domunikację z siecią internet poprzez interfejs ``eth0`` ``PC0``
   * Zapewnij komunikację z siecią internet na poziomie ``LAN1`` oraz ``LAN2``
   * Dokonaj takiego podziału sieci o adresie ``172.22.128.0/17`` aby w ``LAN1`` można było zaadresować ``500`` adresów natomiast w LAN2 ``5000`` adresów    
   * Przygotuj dokumentację powyższej architektury w formie graficznej w programie ``DIA``
 
--------
Rozwiązanie
-----------
1. Ustalenie masek
  - dla 500 urządzeń 255.255.254.0 czyli /23
  - dla 5000 urządzeń 255.255.224.0 czyli /19
  
2. Ustalenie adresów sieci
  IP bazowy to 172.22.128.0
  - lan1 172.22.128.0/23
  - lan2 172.22.160.0/19
  Lan2 ma adres 172.22.160.0 gdyż pula kolejnych dostępnych adresów przy netmasce /19 zaczyna się właśnie od 160.

3. Dodanie adresów ip (ip addr add xxx.xxx.xxx.xxx/xx dev nazwa interfejsu)
  - PC0
  enp0s8
    address 172.22.128.1/23
  enp0s9
    address 172.22.160.1/19
  - PC1
    address 172.22.128.2/23
  - PC2
    address 172.22.160.2/19

włączenie interfejsów (ip link set nazwa_interfejsu up)

4. Ustalenie routingu (w /etc/network/interfaces)
  - PC1
    ip route add default via 172.22.128.1
  - PC2
    ip route add default via 172.22.160.1
    
5. Włączenie forwardowania pakietów na PC0
    echo 1 > /proc/sys/net/ipv4/ip_forward
  Aby forwarding był stale włączony należy odkomentować linijkę 
    net.ipv4.ip_forward=1 
  w pliku /etc/sysctl.d/99-sysctl.conf
  
6. Dodanie reguły masquerade na PC0
    iptables -t nat -A POSTROUTING -s 172.22.128.0/23 -o enp0s3 -j MASQUERADE
    iptables -t nat -A POSTROUTING -s 172.22.160.0/19 -o enp0s3 -j MASQUERADE
  Aby reguły były zapamiętane należy je zapisać
    ipatables-save > /etc/iptables.up.rules
  Potem w pliku /etc/network/interfaces dodać komendę
    post-up iptables-restore < /etc/iptables.up.rules
  żeby po starcie systemu zapisane reguły zostały wczytane
  
7. Dodanie adresów dns do PC1 i PC2
  /etc/resolv.conf
  nameserver 8.8.8.8
