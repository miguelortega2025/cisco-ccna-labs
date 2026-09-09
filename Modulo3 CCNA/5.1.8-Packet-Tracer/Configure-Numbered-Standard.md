# Packet Tracer - Configurar ACL IPv4 Estándar Numeradas

## 📋 Tabla de Asignación de Direcciones

| Dispositivo | Interfaz | Dirección IP | Máscara de Subred | Puerta de Enlace Predeterminado |
|-------------|----------|--------------|-------------------|--------------------------------|
| **R1** | G0/0 | 192.168.10.1 | 255.255.255.0 | N/D |
| | G0/1 | 192.168.11.1 | 255.255.255.0 | |
| | S0/0/0 | 10.1.1.1 | 255.255.255.252 | |
| | S0/0/1 | 10.3.3.1 | 255.255.255.252 | |
| **R2** | G0/0 | 192.168.20.1 | 255.255.255.0 | N/D |
| | S0/0/0 | 10.1.1.2 | 255.255.255.252 | |
| **R3** | S0/0/1 | 10.2.2.1 | 255.255.255.252 | N/D |
| | G0/0 | 192.168.30.1 | 255.255.255.0 | |
| | S0/0/0 | 10.3.3.2 | 255.255.255.252 | |
| | S0/0/1 | 10.2.2.2 | 255.255.255.252 | |
| **PC1** | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| **PC2** | NIC | 192.168.11.10 | 255.255.255.0 | 192.168.11.1 |
| **PC3** | NIC | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| **Servidor Web** | NIC | 192.168.20.254 | 255.255.255.0 | 192.168.20.1 |

---

## 🎯 Objetivos

- **Parte 1:** Planificar una implementación de ACL
- **Parte 2:** Configurar, aplicar y verificar una ACL estándar

---

## 📖 Aspectos Básicos/Situación

Las listas de control de acceso (ACL) estándar son scripts de configuración del router que controlan si un router permite o deniega paquetes según la dirección de origen. Esta actividad se concentra en definir criterios de filtrado, configurar ACL estándar, aplicar ACL a interfaces de router y verificar y evaluar la implementación de la ACL.

Los routers ya están configurados, incluidas las direcciones IP y el enrutamiento del Protocolo de Enrutamiento de la Puerta de Enlace Interior Mejorado (EIGRP).

---

## 🛠️ Instrucciones

### Parte 1: Planifique una implementación de ACL

#### Paso 1: Investigue la configuración actual de red

Antes de aplicar cualquier ACL a una red, es importante confirmar que tenga conectividad completa. Elija una computadora y haga ping a otros dispositivos en la red para verificar que la red tenga plena conectividad.

**Comandos de verificación:**
```bash
# Desde PC1
ping 192.168.11.10   # → PC2
ping 192.168.30.10   # → PC3
ping 192.168.20.254  # → Servidor Web

# Desde PC2
ping 192.168.20.254  # → Servidor Web
ping 192.168.30.10   # → PC3

# Desde PC3
ping 192.168.20.254  # → Servidor Web
