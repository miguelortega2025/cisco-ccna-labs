# Packet Tracer Lab: Troubleshoot Inter-VLAN Routing (Physical Mode)

## Descripción del Laboratorio
- **Curso:** Cisco Networking Academy (CCNA)
- **Actividad:** 4.4.9 Packet Tracer - Troubleshoot Inter-VLAN Routing - Physical Mode
- **Objetivo:** Diagnosticar y corregir errores de configuración que impiden el enrutamiento entre VLANs (Inter-VLAN Routing) y la conectividad de extremo a extremo.
- **Habilidades evaluadas:**
  - Configuración de Router-on-a-Stick (subinterfaces 802.1Q)
  - Configuración de VLANs en switches
  - Configuración de enlaces troncales (Trunks)
  - Configuración de VLAN nativa y VLANs permitidas
  - Verificación de conectividad usando `ping`

---

## Topología

*(Puedes insertar una imagen de la topología aquí si la tienes en tu repositorio)*

- **R1 (Router):** Realiza el enrutamiento entre VLANs mediante subinterfaces en G0/0/1.
- **S1 (Switch 1):** Conecta PC-A (VLAN 4) y enlaza con R1 y S2 mediante trunks.
- **S2 (Switch 2):** Conecta PC-B (VLAN 13) y enlaza con S1 mediante trunk.
- **PC-A:** IP 10.4.0.50 / Gateway 10.4.0.1 (R1)
- **PC-B:** IP 10.13.0.50 / Gateway 10.13.0.1 (R1)

---

## Tabla de Direccionamiento (Resumen)

| Dispositivo | Interfaz        | IP           | Máscara       | Gateway |
|-------------|-----------------|--------------|---------------|---------|
| R1          | G0/0/1.3        | 10.3.0.1     | 255.255.255.0 | N/A     |
| R1          | G0/0/1.4        | 10.4.0.1     | 255.255.255.0 | N/A     |
| R1          | G0/0/1.13       | 10.13.0.1    | 255.255.255.0 | N/A     |
| S1          | VLAN 3          | 10.3.0.11    | 255.255.255.0 | 10.3.0.1|
| S2          | VLAN 3          | 10.3.0.12    | 255.255.255.0 | 10.3.0.1|
| PC-A        | NIC             | 10.4.0.50    | 255.255.255.0 | 10.4.0.1|
| PC-B        | NIC             | 10.13.0.50   | 255.255.255.0 | 10.13.0.1|

---

## Análisis y Diagnóstico Inicial

### Pruebas de Conectividad (Antes de las correcciones)

| Origen | Destino          | Resultado | Observación |
|--------|------------------|-----------|-------------|
| R1     | S1 (10.3.0.11)   | ❌ Falla  | Sin respuesta |
| R1     | S2 (10.3.0.12)   | ❌ Falla  | Sin respuesta |
| R1     | PC-A (10.4.0.50) | ❌ Falla  | Sin respuesta |
| R1     | PC-B (10.13.0.50)| ❌ Falla  | Sin respuesta |
| S1     | S2 (10.3.0.12)   | ❌ Falla  | Sin respuesta |
| PC-A   | PC-B             | ❌ Falla  | Sin respuesta |

### Hipótesis Inicial
- Posible falta de configuración de subinterfaces en R1 (Router-on-a-Stick).
- Posible VLAN nativa incorrecta en los trunks.
- Posible falta de creación de VLANs en los switches.
- Posible configuración incorrecta de puertos de acceso.

---

##  Comandos de Configuración Aplicados

### 1. Configuración del Router R1 (Router-on-a-Stick)

```bash
enable
conf t
interface G0/0/1
 no shutdown
interface G0/0/1.3
 encapsulation dot1Q 3
 ip address 10.3.0.1 255.255.255.0
interface G0/0/1.4
 encapsulation dot1Q 4
 ip address 10.4.0.1 255.255.255.0
interface G0/0/1.13
 encapsulation dot1Q 13
 ip address 10.13.0.1 255.255.255.0
exit
