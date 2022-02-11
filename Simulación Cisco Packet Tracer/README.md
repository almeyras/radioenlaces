## 1 - Rede debuxada sen configurar
Este primeiro ficheiro presenta a rede completamente debuxada, pero só ten configurados os seguintes equipos:
- Routers CPE de cada cliente (NAT, AP Wi-Fi, IP WAN con ruta predeterminada ó nodo máis próximo: `172.16.X.1`). 
- Redes locais de cada cliente `192.168.0.0/24`
- Dúas conexións a punto neutro, o cal ten unha rede enriba que representa o servidor DNS de Google, é dicir `8.8.8.8`. A interconexión se fai por unha rede de tránsito `192.168.0.0/24`, que coincide coas redes dos clientes, pero isto non será problema gracias á NAT.

É importante configurar o **router do punto neutro** para interconectar `8.8.8.8` coa rede WMAN a través da rede de tránsito `192.168.0.0/24`:

```
enable
configure terminal
!
hostname NEUTRO
no ip domain-lookup
logging synchronous
!
interface g0/1
ip address 192.168.0.1 255.255.255.0
no shutdown
!
int g0/0
ip address 8.8.8.1 255.255.255.0
no shutdown
!
exit
!
ip dhcp excluded-address 192.168.0.1 192.168.0.99
ip dhcp pool POOL
network 192.168.0.0 255.255.255.0
default-router 192.168.0.1
dns-server 8.8.8.8
exit
!
end

copy running-config startup-config
```

> Repara en que a rede `192.168.0.0/24` é idéntica no circuito de tránsito en as tres subredes NAT dos clientes. Calquera equipo vai poder alcanzar `8.8.8.8` gracias á maxia da NAT.

## 2 - Interfaces


**N1**:
```
!
enable
configure terminal
!
hostname N1
no ip domain-lookup
logging synchronous
!
interface GigabitEthernet 0/0
ip address 10.0.1.1 255.255.0.0
no shutdown
!
interface GigabitEthernet 0/1
ip address 172.16.1.1 255.255.255.0
no shutdown
!
interface g0/0/0
! Mejor no usar DHCP o las rutas del punto neutro fallarán
!ip address dhcp
ip address 192.168.0.201 255.255.255.0
no shutdown
end
copy running-config startup-config
```

**N2**:
```
!
enable
configure terminal
!
hostname N2
no ip domain-lookup
logging synchronous
!
interface GigabitEthernet 0/0
ip address 10.0.2.1 255.255.0.0
no shutdown
!
interface GigabitEthernet 0/1
ip address 172.16.2.1 255.255.255.0
no shutdown
!
interface g0/0/0
! Mejor no usar DHCP o las rutas del punto neutro fallarán
!ip address dhcp
ip address 192.168.0.202 255.255.255.0
no shutdown
end
!
copy running-config startup-config
```

**N3**:
```
!
enable
configure terminal
!
hostname N3
no ip domain-lookup
logging synchronous
!
interface GigabitEthernet 0/0
ip address 10.0.3.1 255.255.0.0
no shutdown
!
interface GigabitEthernet 0/1
ip address 172.16.3.1 255.255.255.0
no shutdown
exit
!
exit
copy running-config startup-config

```

## 3 - STP

Este apartado evita a xeracion de tormentas broadcast por bucles na rede clase A, que é mallada. **N1bridgeA, N2bridgeA, N3bridgeA** (switches que simulan os bridges clase S que deberían ir nos propios routers N1, N2, N3):

```
!
enable
configure terminal
no ip domain-lookup
logging synchronous
!
spanning-tree mode rapid-pvst
!
interface vlan 1
spanning-tree VLAN 1 root primary
end
!
copy running-config startup-config

```


## 4 - Rutas OSPF
Este apartado garantiza a conectividade lóxica entre as redes clase A y B. É unha opción moito máis recomendable que as rutas estáticas xa que permite escalar a rede con maior facilidade.

**Proba a facer `ping 172.16.x.x` desde un equipo que non linde con dita rede antes e despois de converxer o enrutamento OSPF**


**N1**:
```
enable
configure terminal
router ospf 1
router-id 10.0.1.1
ip ospf priority 1 
! redes que se anuncian
network 10.0.0.0 0.0.255.255 area 0
network 172.16.1.0 0.0.0.255 area 0
passive-interface default
! interfaces por donde se propagan rutas
no passive-interface g0/1
no passive-interface g0/0
end
!
copy running-config startup-config
```
**N2**:
```
enable
configure terminal
router ospf 1
router-id 10.0.2.1
ip ospf priority 1 
! redes que se anuncian
network 10.0.0.0 0.0.255.255 area 0
network 172.16.2.0 0.0.0.255 area 0
passive-interface default
! interfaces por donde se propagan rutas
no passive-interface g0/1
no passive-interface g0/0
end
!
copy running-config startup-config
```

**N3**:
```
enable
configure terminal
router ospf 1
router-id 10.0.3.1
ip ospf priority 1 
! redes que se anuncian
network 10.0.0.0 0.0.255.255 area 0
network 172.16.3.0 0.0.0.255 area 0
passive-interface default
! interfaces por donde se propagan rutas
no passive-interface g0/1
no passive-interface g0/0
end
!
copy running-config startup-config
```


## 5 - Rutas de último recurso
Este apartado serve para que os equipos podan acceder a Internet.

N1 e N2 (acceden a Internet a través do punto neutro ou do nodo adxacente se o tránsito falla). Empezamos por **N1**:
```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 192.168.0.1
ip route 0.0.0.0 0.0.0.0 10.0.2.1 5
exit
copy running-config startup-config
```

**N2**:
```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 192.168.0.1
ip route 0.0.0.0 0.0.0.0 10.0.1.1 5
exit
copy running-config startup-config
```

**N3** (accede a Internet a través de N1 ou N2):
```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 10.0.1.1
ip route 0.0.0.0 0.0.0.0 10.0.2.1 5
exit
copy running-config startup-config
```

Os routers CPE xa veñen configurados con porta de enlace a IP clase B do seu propio nodo de orixe (N1, N2 ou N3).



## 6 - NAT
Después de este paso, el tráfico de N1 y N2 podrá llegar a Internet.

**N1** e **N2**:
```
enable
configure terminal
!
access-list 1 permit 10.0.0.0 0.0.255.255
access-list 1 permit 172.16.0.0 0.0.255.255
do show access-list
!
ip nat inside source list 1 interface g0/0/0 overload
do show ip nat translations
do show ip nat statistics
!
interface g0/0
ip nat inside
interface g0/1
ip nat inside
interface g0/0/0
ip nat outside
end
!
copy running-config startup-config
```

## Cómo borrar configuración de routers/switches
Para restaurar la configuración predeterminada de los equipos:

```
enable
write erase
reload
```
