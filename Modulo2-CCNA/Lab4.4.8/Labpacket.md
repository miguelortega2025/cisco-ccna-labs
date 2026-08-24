# Bitácora de Aprendizaje: Packet Tracer - Troubleshoot Inter-VLAN Routing

## Resumen del Laboratorio

En esta actividad de **Packet Tracer**, se trabajó en la resolución de problemas de conectividad entre VLANs utilizando **Router-on-a-Stick** (enrutamiento inter-VLAN). El objetivo era identificar y corregir errores de configuración en un router y un switch para permitir la comunicación entre PC1 (VLAN 10) y PC3 (VLAN 30).

---

## Problemas Identificados y Soluciones

### 1. Encapsulación incorrecta en subinterfaces del router

**Problema:**  
Las subinterfaces `G0/1.10` y `G0/1.30` no tenían el comando `encapsulation dot1q` con el ID de VLAN correspondiente, lo que impedía el enrutamiento entre VLANs.

**Solución aplicada:**
```bash
R1(config)# interface g0/1.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# ip address 172.17.10.1 255.255.255.0

R1(config)# interface g0/1.30
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# ip address 172.17.30.1 255.255.255.0
