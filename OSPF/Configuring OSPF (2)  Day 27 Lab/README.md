# Lab - Configuring OSPF (2) - Day 27 Lab

## Objetivo

Configurar OSPF en una topología de cuatro routers.

## Topología

R1 conectado a R2 (G0/0), R3 (F1/0) e ISPR1 (G3/0).  
R2 conectado a R1 (G0/0) y R4 (F1/0).  
R3 conectado a R1 (F1/0) y R4 (F2/0).  
R4 conectado a R2 (F1/0), R3 (F2/0) y SW1 (G0/0).

## Direccionamiento

| Dispositivo | Interfaz | IP / Prefijo | Descripción |
|-------------|----------|--------------|-------------|
| R1 | G0/0 | 10.0.12.1/30 | ## to R2 |
| R1 | F1/0 | 10.0.13.1/30 | ## to R3 |
| R1 | G3/0 | 203.0.113.1/30 | ## to ISPR1 |
| R1 | Lo0 | 1.1.1.1/32 | |
| R2 | G0/0 | 10.0.12.2/30 | ## to R1 |
| R2 | F1/0 | 10.0.24.1/30 | ## to R4 |
| R2 | Lo0 | 2.2.2.2/32 | |
| R3 | F1/0 | 10.0.13.2/30 | ## to R1 |
| R3 | F2/0 | 10.0.34.1/30 | ## to R4 |
| R3 | Lo0 | 3.3.3.3/32 | |
| R4 | F2/0 | 10.0.34.2/30 | ## to R3 |
| R4 | F1/0 | 10.0.24.2/30 | ## to R2 |
| R4 | G0/0 | 192.168.4.254/24 | ## to SW1 |
| R4 | Lo0 | 4.4.4.4/32 | |
| ISPR1 | - | 203.0.113.2/30 | |

## Configuración realizada

### Configuración básica de interfaces

#### R1

```cmd
R1(config)#int f1/0
R1(config-if)#ip address 10.0.13.1 255.255.255.252
R1(config-if)#desc ## to R3 ##
R1(config-if)#no shutdown
!
R1(config)#int g0/0
R1(config-if)#ip address 10.0.12.1 255.255.255.252
R1(config-if)#desc ## to R2 ##
R1(config-if)#no shutdown
!
R1(config-if)#int g3/0
R1(config-if)#ip address 203.0.113.1 255.255.255.252
R1(config-if)#desc ## to ISPR1 ##
R1(config-if)#no shutdown
```

#### R2

```cmd
Router(config)#hostname R2
R2(config)#int g0/0
R2(config-if)#ip add 10.0.12.2 255.255.255.252
R2(config-if)#desc ## to R1 ##
R2(config-if)#no shutdown
!
R2(config-if)#int f1/0
R2(config-if)#ip address 10.0.24.1 255.255.255.252
R2(config-if)#desc ## to R4 ##
R2(config-if)#no shutdown
```

#### R3

```cmd
Router(config)#hostname R3
R3(config)#int f1/0
R3(config-if)#ip address 10.0.13.2 255.255.255.252
R3(config-if)#desc ## to R1 ##
R3(config-if)#no shutdown
!
R3(config-if)#int f2/0
R3(config-if)#ip address 10.0.34.1 255.255.255.252
R3(config-if)#desc ## to R4 ##
R3(config-if)#no shutdown
```

#### R4

```cmd
Router(config)#hostname R4
R4(config)#int f2/0
R4(config-if)#ip address 10.0.34.2 255.255.255.252
R4(config-if)#desc ## to R3 ##
R4(config-if)#no shutdown
!
R4(config-if)#int f1/0
R4(config-if)#ip address 10.0.24.2 255.255.255.252
R4(config-if)#desc ## to R2 ##
R4(config-if)#no shutdown
!
R4(config-if)#int g0/0
R4(config-if)#ip address 192.168.4.254 255.255.255.0
R4(config-if)#desc ## to SW1 ##
R4(config-if)#no shutdown
```

### Loopback Addresses

```cmd
R1(config)#int l0
R1(config-if)#ip address 1.1.1.1 255.255.255.255
!
R2(config)#int l0
R2(config-if)#ip address 2.2.2.2 255.255.255.255
!
R3(config)#int l0
R3(config-if)#ip address 3.3.3.3 255.255.255.255
!
R4(config)#int l0
R4(config-if)#ip address 4.4.4.4 255.255.255.255
```

### Configuración OSPF

Habilitar OSPF directamente en cada interfaz y configurar interfaces pasivas según corresponda. Viendo el video de Jeremy me he dado cuenta que puedo hacer `interface range g0/0, f1/0, l0`, y luego `ìp ospf process ID area area`. Pero bueno, ya está hecho😅. 

#### R1 — OSPF Process 1, Area 0

```cmd
R1(config-if)#int g0/0
R1(config-if)#ip ospf 1 area 0
!
R1(config-if)#int f1/0
R1(config-if)#ip ospf 1 area 0
!
R1(config)#int l0
R1(config-if)#ip ospf 1 area 0
!
R1(config-router)#passive-interface default
R1(config-router)#no passive-interface f1/0
R1(config-router)#no passive-interface g0/0
```

#### R2 — OSPF Process 2, Area 0

```cmd
R2(config)#int f1/0
R2(config-if)#ip ospf 2 area 0
!
R2(config-if)#int g0/0
R2(config-if)#ip ospf 2 area 0
!
R2(config-if)#int l0
R2(config-if)#ip ospf 2 area 0
!
R2(config-if)#router ospf 2
R2(config-router)#passive-interface loopback 0
```

#### R3 — OSPF Process 3, Area 0

```cmd
R3(config)#int f1/0
R3(config-if)#ip ospf 3 area 0
!
R3(config-if)#int f2/0
R3(config-if)#ip ospf 3 area 0
!
R3(config-if)#int l0
R3(config-if)#ip ospf 3 area 0
!
R3(config-if)#router ospf 3
R3(config-router)#passive-interface l0
```

#### R4 — OSPF Process 4, Area 0

```cmd
R4(config)#int f2/0
R4(config-if)#ip ospf 4 area 0
!
R4(config)#int f1/0
R4(config-if)#ip ospf 4 area 0
!
R4(config)#int l0
R4(config-if)#ip ospf 4 area 0
!
R4(config-if)#int g0/0
R4(config-if)#ip ospf 4 area 0
!
R4(config-if)#router ospf 4
R4(config-router)#passive-interface default
R4(config-router)#no passive-interface f1/0
R4(config-router)#no passive-interface f2/0
```

### Reference Bandwidth

Configurar el reference bandwidth en todos los routers para que una interfaz FastEthernet tenga un costo de 100:

```cmd
R1(config-router)#auto-cost reference-bandwidth 10000
!
R2(config-router)#auto-cost reference-bandwidth 10000
!
R3(config-router)#auto-cost reference-bandwidth 10000
!
R4(config-router)#auto-cost reference-bandwidth 10000
```

### Default Route — R1 como ASBR

```cmd
R1(config)#ip route 0.0.0.0 0.0.0.0 203.0.113.2
R1(config)#router ospf 1
R1(config-router)#default-information originate
```

### Resultados

- ✅ Todos los routers establecen adyacencias OSPF en área 0.
- ✅ Las rutas OSPF se propagan correctamente entre los routers.
- ✅ La ruta por defecto (0.0.0.0/0) se anuncia desde R1 al dominio OSPF.
- ✅ Las interfaces FastEthernet tienen costo 100 (reference-bandwidth 10000).
- ✅ Las interfaces loopback están configuradas como pasivas donde corresponde.

## Aprendizajes

| Campo       | Valor                   |
| ----------- | ----------------------- |
| Dificultad  | ⭐⭐⭐☆☆                  |
| Tecnologías | OSPF, Cisco IOS         |
| Software    | Cisco Packet Tracer     |
| Estado      | ✅ Completado            |
