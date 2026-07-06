# Lab - HSRP Configuration

## Objetivo

Configurar HSRPv2 entre dos routers.

## Topología

![Topología](images/Pasted%20image%2020260706165620.png)

## Direccionamiento

### Subred LAN

| Red | Gateway Virtual (VIP) | Gateway R1 | Gateway R2 |
|-----|----------------------|-------------|-------------|
| 10.0.1.0/24 | 10.0.1.254 | 10.0.1.253 | 10.0.1.252 |

## Configuración realizada

### Configuración HSRPv2 en R1

```bash
hostname R1
!
int g0/0
ip address 10.0.1.253 255.255.255.0
no shutdown
!
standby version 2
standby 1 priority 200
standby 1 preempt
standby 1 ip 10.0.1.254
```

### Configuración HSRPv2 en R2

```bash
hostname R2
!
int g0/0
ip address 10.0.1.252 255.255.255.0
no shutdown
!
standby version 2
standby 1 priority 50
standby 1 ip 10.0.1.254
```

## Verificación

### Comandos ejecutados

```bash
show standby
ping 8.8.8.8
arp -a
```

### Resultados

Se configuró la VIP `10.0.1.254` como gateway predeterminado de los PCs. Al hacer ping a `8.8.8.8`, se verificó que la MAC address aprendida en la tabla ARP corresponde a la MAC virtual de HSRP.

![ARP inicial - MAC HSRPv1](images/Pasted%20image%2020260706170727.png)

Inicialmente aparecía la MAC `0000.0c07.ac01` (propia de HSRPv1) debido a limitaciones de Packet Tracer.

![ARP después de reinicio - MAC HSRPv2 correcta](images/Pasted%20image%2020260706171037.png)

Luego de reiniciar R1, se mostró correctamente la MAC `0000.0c9f.f001` correspondiente a HSRPv2.

![MAC HSRPv2 correcta](images/Pasted%20image%2020260706172120.png)

### Failover

Se desactivó R1 para simular una falla. R2 asumió el rol **Active** y envió gratuitous ARPs para actualizar la red.

![R2 activo tras falla de R1](images/Pasted%20image%2020260706173225.png)

![ARP después del failover](images/Pasted%20image%2020260706173330.png)

### Recuperación

Se encendió R1 nuevamente. Gracias a la configuración `standby 1 preempt`, R1 recuperó el rol **Active** automáticamente.

![R1 recupera rol Active](images/Pasted%20image%2020260706173533.png)

![R1 activo nuevamente](images/Pasted%20image%2020260706172600.png)

## Problemas encontrados

- Packet Tracker mostraba la MAC de HSRPv1 (`0000.0c07.ac01`) en lugar de la de HSRPv2 (`0000.0c9f.f001`) debido a limitaciones del simulador. Se solucionó reiniciando el router.
- Al desactivar R1, R2 no tomaba el rol Active de inmediato hasta que se verificó la configuración de `preempt`.

## Aprendizajes

- Configuración de HSRPv2 con prioridades personalizadas.
- Uso de `preempt` para que el router con mayor prioridad recupere el rol Active tras una recuperación.
- La MAC virtual de HSRPv2 sigue el formato `0000.0c9f.XXXX`.
- Verificación de failover mediante gratuitous ARP y tabla ARP de los PCs.

## Ficha del laboratorio

| Campo       | Valor               |
| ----------- | ------------------- |
| Dificultad  | ★★☆☆☆               |
| Tecnologías | HSRPv2, Redundancia, Cisco IOS |
| Software    | Cisco Packet Tracer |
| Estado      | ✅ Completado        |
