#  Lab: Router-on-a-Stick (Inter-VLAN Routing)

## Descripción
Configuración de enrutamiento entre VLANs usando la técnica "Router-on-a-Stick" (trunk hacia el router + subinterfaces).

## Topología
![Topología](https://via.placeholder.com/800x300?text=Topología+Router-on-a-Stick)

## Tabla de Direcciones
| Dispositivo | Interfaz | VLAN | IP | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| R1 | G0/0/1.3 | 3 | 192.168.3.1 | N/A |
| R1 | G0/0/1.4 | 4 | 192.168.4.1 | N/A |
| R1 | G0/0/1.8 | 8 (Nativa) | Sin IP | N/A |
| S1 | VLAN 3 | 3 | 192.168.3.11 | 192.168.3.1 |
| S2 | VLAN 3 | 3 | 192.168.3.12 | 192.168.3.1 |
| PC-A | NIC | 3 | 192.168.3.3 | 192.168.3.1 |
| PC-B | NIC | 4 | 192.168.4.3 | 192.168.4.1 |

## Comandos Clave

### 1. VLANs y Puertos (Switches)
```bash
vlan 3
name Management
vlan 4
name Operations
vlan 7
name ParkingLot
vlan 8
name Native

interface f0/6
switchport mode access
switchport access vlan 3
