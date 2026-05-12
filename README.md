ARQUITECTURA DE RED SEGURA

Credenciales de Acceso para Auditoria Técnica

** Acceso EXEC consola y privilegiado (R - SW): cisco
** Acceso remoto seguro (SSHv2): marquez / cisco123 
** Acceso WiFi (VLAN 88): SSID Red_Corporativa_Garcia /  Contraseña: Marquez_CCNA_2026


![Diseño de la red](topología.png)


## Configuraciones reales extraídas (running-config) ##

A continuación, se presentan los bloques de configuración vivos extraídos directamente de la memoria de los dispositivos 
tras aplicar las políticas de endurecimiento (CCNA).

    
    ##### Switch de Acceso  ( SW-Garcia-NET) #####

!
hostname SW-Garcia-NET
!
enable password 7 0822455D0A16
!
!
!
ip ssh version 2
ip domain-name marquez.local
!
username Marquez secret 5 $1$mERr$5.a6P4JqbNiMX01usIfka/
!
!____ Sección de optimización STPM (SPANNING TREE)___
!
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 77,88,99 priority 28672
!
!___ Asignación de puertos de acceso (vlan 77 y 88)___
!
interface range FastEthernet0/1-19
 switchport access vlan 77
 switchport mode access
 spanning-tree portfast
!
!
interface FastEthernet0/20
 switchport access vlan 88
 switchport mode access
 spanning-tree portfast
!
!
interface range FastEthernet0/21-23
 switchport access vlan 77
 switchport mode access
 spanning-tree portfast
!
!
!___ Se asigna el puerto 24 a troncal por emergencia si se cae la G0/1___
!
interface FastEthernet0/24  
 switchport mode trunk
 shutdown
!
!___Enlace troncal activo hacia el router core___
!
interface GigabitEthernet0/1
 switchport mode trunk
!
!___ Interfaz G0/2 desactivada___
!
interface GigabitEthernet0/2
!
!___ Se deshabilita la vlan1 nativa por insegura___
!
interface Vlan1
 no ip address
 shutdown
!
!___ Interfaz virtual de administración (SVI)___
!
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
!
ip default-gateway 192.168.99.1
!
!___ Se crea un banner de advertencia___
!
banner motd ^CACCESO NO AUTORIZADO^C
!
!___ Implementé contraseña y cifrado de consola ___
!
line con 0
 password 7 0822455D0A16
 login
!
!___ Control de acceso cifrado al SVI___
!
line vty 0 4
 password 7 0822455D0A16
 login local
 transport input ssh
line vty 5 15
 login
!
!
end





    #####  Router acceso ( R-Marquez-NET )  #####

!
hostname R-Marquez-NET
!
!
!
enable password 7 0822455D0A16
!
!___ Configuración de direccionamiento dinámico ___
!
ip dhcp excluded-address 192.168.77.1 192.168.77.10
ip dhcp excluded-address 192.168.88.1 192.168.88.10
ip dhcp excluded-address 192.168.99.1 192.168.99.10
!
!
ip dhcp pool Datos
 network 192.168.77.0 255.255.255.0
 default-router 192.168.77.1
 dns-server 8.8.8.8
ip dhcp pool WiFi
 network 192.162.88.0 255.255.255.0
 default-router 192.168.88.1
 dns-server 8.8.8.8
!
!___ Interfaz física principal__
!
interface GigabitEthernet0/0/0
 no ip address
 duplex auto
 speed auto
!
!___ Subinterfaz ( Router-on-a-stick)___
!
interface GigabitEthernet0/0/0.77
 encapsulation dot1Q 77
 ip address 192.168.77.1 255.255.255.0
 ip access-group 101 in
!
interface GigabitEthernet0/0/0.88
 encapsulation dot1Q 88
 ip address 192.168.88.1 255.255.255.0
 ip access-group 101 in
!
interface GigabitEthernet0/0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
!
!___ Interfaces inactivas por seguridad___
!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
!___ Parámetros globales del sistema___
!
ip classless
!
ip flow-export version 9
!
!
access-list 101 permit ip host 192.168.77.10 host 192.168.99.2
access-list 101 permit ip host 192.168.77.9 host 192.168.99.2
access-list 101 deny ip 192.168.77.0 0.0.0.255 192.168.99.0 0.0.0.255
access-list 101 deny ip 192.168.88.0 0.0.0.255 192.168.99.0 0.0.0.255
access-list 101 permit ip any any
!
!
!___ Líneas físicas y virtuales de acceso___
!
line con 0
 password 7 0822455D0A16
 login
!
line aux 0
!
line vty 0 4
 password 7 0822455D0A16
 login
!
!
!
end





