# Bitácora de Aprendizaje: Router-on-a-Stick Inter-VLAN Routing

##  Información General
- **Actividad:** Packet Tracer 4.2.7 - Configure Router-on-a-Stick Inter-VLAN Routing
- **Fecha:** [Fecha de realización]
- **Dispositivos:** R1 (Router), S1 (Switch), PC1, PC3
- **Habilidades:** VLANs, subinterfaces, trunking, enrutamiento entre VLANs

---

##  Objetivos
1. Crear y asignar VLANs en un switch.
2. Configurar subinterfaces en un router para enrutamiento inter-VLAN.
3. Habilitar trunking en el enlace switch-router.
4. Verificar conectividad entre VLANs.

---

##  Conceptos Clave

### ¿Qué es Router-on-a-Stick?
- Técnica de enrutamiento entre VLANs utilizando **una sola interfaz física** en el router.
- La interfaz física se divide en **subinterfaces lógicas** (una por VLAN).
- Cada subinterfaz se etiqueta con **802.1Q** para identificar la VLAN.

### ¿Por qué usar este método?
- Económico (no requiere switch multicapa).
- Fácil de escalar para pocas VLANs.
- Aprovecha puertos existentes.

---

##  Configuración Paso a Paso

### 1. Crear VLANs en S1
```bash
S1(config)# vlan 10
S1(config-vlan)# name VLAN0010
S1(config-vlan)# exit

S1(config)# vlan 30
S1(config-vlan)# name VLAN0030
S1(config-vlan)# exit
