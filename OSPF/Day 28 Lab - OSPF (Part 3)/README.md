# Lab - Configuring OSPF (3) - Day 28 Lab

## Objetivo

Troubleshooting OSPF
## Topología

R1 conectado a R2 (S0/0/0) y a una red LAN (G0/0).  
R2 conectado a R1 (S0/0/0), R4 y R5.  
R3 conectado a R4.  
R4 conectado a R2, R3 y R5.  
R5 conectado a R2, R4 y a una red externa.  
PC1 y PC2 conectados a la LAN de R1.

## Configuración realizada

### Conexión serial R1-R2

#### R1

```cmd
R1(config)#int s 0/0/0
R1(config-if)#ip address 192.168.12.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#clock rate 128000
R1(config-if)#encapsulation ppp
!
R1(config-if)#ip ospf 1 area 0
R1(config-if)#ip ospf network point-to-point
```

#### R2

```cmd
R2(config)#int s0/0/0
R2(config-if)#ip address 192.168.12.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#encapsulation ppp
!
R2(config-if)#ip ospf 1 area 0
R2(config-if)#ip ospf network point-to-point
```

### Problema 1: R3 y R4 no intercambian LSAs

Solo R3 tiene una ruta a 10.0.2.0/24. El problema es que no tienen el mismo network type: R3 tiene point-to-point mientras que R4 tiene broadcast. Se reconocen como vecinos pero no pueden intercambiar LSAs.

**Solución:**

```cmd
R4(config-if)#ip ospf network point-to-point
```

### Problema 2: R2/R4 y R5 no forman adyacencia OSPF

Los intervalos de Hello y Dead son diferentes. R5 tiene intervalos de 5 y 20 segundos respectivamente.

**Solución:**

```cmd
R5(config-if)#ip ospf hello-interval 10
R5(config-if)#ip ospf dead-interval 40
```

### Problema 3: PC1 y PC2 no pueden hacer ping a 8.8.8.8

**Problema A:** R5 no tenía ninguna default route configurada.

**Solución:**

```cmd
R5(config)#ip route 0.0.0.0 0.0.0.0 203.0.113.2
R5(config-router)#default-information originate
```

**Problema B:** La interfaz G0/0 de R1 (10.0.1.254/24) no estaba en OSPF.

**Solución:**

```cmd
R1(config)#int g0/0
R1(config-if)#ip ospf 1 area 0
R1(config-router)#passive-interface g0/0
```

### Resultados

- ✅ La conexión serial entre R1 y R2 está operativa con clock rate 128000 y encapsulación PPP.
- ✅ R3 y R4 pueden intercambiar LSAs correctamente (mismo network type: point-to-point).
- ✅ R2/R4 y R5 forman adyacencias OSPF (intervalos Hello/Dead sincronizados).
- ✅ PC1 y PC2 pueden hacer ping a 8.8.8.8 (default route en R5 y G0/0 de R1 en OSPF).

## Problemas encontrados

1. **Network type mismatch:** R3 y R4 tenían diferente network type (point-to-point vs broadcast), lo que impedía el intercambio de LSAs.
2. **Hello/Dead interval mismatch:** R5 tenía intervalos diferentes a R2 y R4, impidiendo la formación de adyacencias.
3. **Falta de default route:** R5 no tenía ruta por defecto hacia el ISP, por lo que el tráfico externo no podía salir.
4. **Interfaz no anunciada en OSPF:** La interfaz G0/0 de R1 no estaba incluida en el proceso OSPF, por lo que las PCs no podían alcanzar la red externa.

## Aprendizajes

| Campo       | Valor                   |
| ----------- | ----------------------- |
| Dificultad  | ⭐⭐⭐⭐☆                  |
| Tecnologías | OSPF, Cisco IOS, PPP    |
| Software    | Cisco Packet Tracer     |
| Estado      | ✅ Completado            |
