# Radioenlaces e redes metropolitanas sen fíos
Neste repositorio tes os scripts de configuración dos equipos empregados no curso "Radioenlaces e redes metropolitanas sen fíos". Link aos apuntes do curso: <https://danielrios.me/apuntes/2021radioenlaces/index.html>

Mapa de rede:

A rede se consta de tres nodos que forman a rede de transporte (_"backhaul"_ N1, N2, N3, de cor amarela). Dous deses nodos (N1, N2) teñen conectividade directa a Internet. O nodo N3 se pode configurar provisionalmente con acceso a Internet para traballar con comodidade. 

Os radioenlaces da rede de transporte utilizan antenas Ubiquiti Litebeam (300-600 Mbps). Todos os equipamentos de radio funcionan como _bridges_ transparentes. Cando se logre enlazar N3 con N1 e N2, se establecen portas de enlace a ambos nodos e se retira o cable. Para que N3 priorice un nodo sobre outro se deben axustar as distancias administrativas _(failover)_. Alternativamente, é posible configurar [balanceo de carga](https://help.mikrotik.com/docs/display/ROS/Load+Balancing).

Rematada a rede de transporte, pasamos á rede de acceso (_"last mile"_, de cor verde), que proporciona acceso aos locais dos clientes ou suscriptores da nosa rede. Os radioenlaces se fan con equipos MikroTik de gama baixa (<100 Mbps). Nos locais de cliente se conecta a antenas de última milla co porto WAN dun router neutro que se instalará no interior da casa do suscriptor.

Durante a realización das prácticas haberá ocasións nas que teñas problemas para acceder ós equipos. Para reducir esa fricción, o mellor é que prepares o teu PC con rutas estáticas ás subredes desta práctica. Supoñendo que a túa rede local (en cor vermella) ten a dirección de rede `192.168.0.0/24`, e que os nodos N1, N2, N3 van ter as IP `.201`, `.202`e `203`; as rutas estáticas serían:

- Windows (`cmd` como administrador). O modificador `-p`fai que as rutas persistan despois de reiniciar o PC:
	route ADD 10.0.0.0 MASK 255.255.0.0 192.168.0.201
	route ADD 10.0.0.0 MASK 255.255.0.0 192.168.0.202 METRIC 5
	route ADD 10.0.0.0 MASK 255.255.0.0 192.168.0.203 METRIC 10
	route ADD 172.16.1.0 MASK 255.255.255.0 192.168.0.201
	route ADD 172.16.2.0 MASK 255.255.255.0 192.168.0.202
	route ADD 172.16.3.0 MASK 255.255.255.0 192.168.0.203
	route print


- macOS:
	sudo route -n add -net 10.0.0.0/16 192.168.0.201
	sudo route -n add -net 10.0.0.0/16 192.168.0.202 metric 5
	sudo route -n add -net 10.0.0.0/16 192.168.0.203 metric 10
	sudo route -n add -net 172.16.1.0/24 192.168.0.201
	sudo route -n add -net 172.16.2.0/24 192.168.0.202
	sudo route -n add -net 172.16.3.0/24 192.168.0.203
	netstat -nr


- Linux:
	sudo ip route add 10.0.0.0/16 via 192.168.0.201
	sudo ip route add 10.0.0.0/16 via 192.168.0.202 metric 5
	sudo ip route add 10.0.0.0/16 via 192.168.0.203 metric 10
	sudo ip route add 172.17.1.0/24 via 192.168.0.201
	sudo ip route add 172.17.2.0/24 via 192.168.0.202
	sudo ip route add 172.17.3.0/24 via 192.168.0.203
	ss -natup

Lembra eliminar as rutas despois de rematar a práctica.



