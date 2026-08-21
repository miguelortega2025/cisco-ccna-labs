# Bitácora de Aprendizaje: Implementación de VLANs y Trunking (Packet Tracer)

## Descripción del Laboratorio
Este laboratorio se centra en la configuración de VLANs, asignación de puertos, y la creación de enlaces troncales (Trunking) en switches Cisco Catalyst 2960. El objetivo era simular un entorno de sucursal con 3 switches (SWA, SWB y SWC) y 7 PCs, implementando VLANs de datos, voz y gestión.

## Objetivos Cumplidos
- Creación y nombrado correcto de VLANs (10, 20, 30, 40, 99, 100).
- Asignación de puertos de acceso a VLANs específicas.
- Configuración de puertos de voz (Voice VLAN).
- Configuración de interfaces virtuales de gestión (SVI) y restricción de pings entre ellas.
- Implementación de Trunking **Estático** (SWA - SWB) y **Dinámico** (SWA - SWC).
- Eliminación de conflictos de VLAN nativa.

---

##  Lecciones Aprendidas y Errores Comunes

### 1. La Importancia del Orden en la Configuración de Trunks
**Error:** Al configurar el puerto troncal `G0/1` en SWA, el comando `switchport nonegotiate` y la VLAN nativa no se aplicaban correctamente.
**Solución:** El orden de los comandos es crucial en IOS. Primero se define el modo del puerto (`switchport mode trunk`) y luego se aplican las propiedades del troncal (`switchport trunk native vlan` y `switchport nonegotiate`).

### 2. Los Nombres de las VLANs son Sensibles a Mayúsculas/Minúsculas
**Error:** La VLAN 99 (Management) aparecía con una "X" roja en la verificación de Packet Tracer.
**Solución:** El nombre `Management` debe escribirse exactamente con la "M" mayúscula. Packet Tracer es estricto con el cumplimiento de las especificaciones.

### 3. El Conflicto de VLAN Nativa (Native VLAN Mismatch)
**Error:** El puerto `G0/2` en SWC mostraba un error en "Native VLAN" y no completaba el enlace troncal.
**Solución:** La VLAN nativa por defecto en Cisco es la VLAN 1. Si un extremo del troncal tiene la VLAN 100 como nativa y el otro no, se genera un conflicto. Es **obligatorio** configurar `switchport trunk native vlan 100` en **ambos extremos** del enlace (SWA y SWC, y SWA y SWB).

### 4. Negociación DTP (Dynamic Trunking Protocol)
**Aprendizaje:** Para que un troncal se forme dinámicamente:
- Si un switch está en modo `dynamic auto` (default en 2960), el otro extremo debe estar en modo `dynamic desirable` para que negocien.
- Para troncales estáticos, se debe deshabilitar DTP usando `switchport nonegotiate` en ambos extremos para evitar que el puerto intente negociar y cambie de estado.

### 5. Restricción de Acceso entre SVIs (Gestión)
**Aprendizaje:** Aunque los switches estén en la misma subred (192.168.99.0/24), el enunciado pedía que no pudieran hacer ping entre sí. Esto se logró aplicando una **ACL (Access Control List)** extendida en la interfaz VLAN 99 de cada switch para bloquear el tráfico ICMP entrante.

---

## 🛠️ Comandos Clave Utilizados

### Configuración de VLANs
```bash
vlan 10
 name Admin
vlan 99
 name Management
vlan 100
 name Native
