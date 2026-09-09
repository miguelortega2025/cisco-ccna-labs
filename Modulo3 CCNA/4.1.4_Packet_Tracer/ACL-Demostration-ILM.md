# Packet Tracer: Demostración de Listas de Control de Acceso (ACL)

## Descripción del laboratorio

En esta actividad se demuestra cómo una **Lista de Control de Acceso (ACL)** puede utilizarse para bloquear el tráfico de una red específica hacia redes remotas. Se utiliza una ACL estándar para filtrar el tráfico originado en la red `192.168.10.0/24`.

---

## Tabla de asignación de direcciones

| Dispositivo | Interfaz | Dirección IP / Prefijo |
|-------------|----------|------------------------|
| **R1** | G0/0 | 192.168.10.1/24 |
| | G0/1 | 192.168.11.1/24 |
| | S0/0/0 | 10.1.1.1/30 |
| **R2** | S0/0/0 | 10.10.1.2/30 |
| | S0/0/1 | 10.10.1.5/30 |
| | G0/0 | 192.168.30.1/24 |
| | G0/1 | 192.168.31.1/24 |
| | S0/0/1 | 10.10.1.6/24 |
| **PC1** | NIC | 192.168.10.10/24 |
| **PC2** | NIC | 192.168.10.11/24 |
| **PC3** | NIC | 192.168.11.10/24 |
| **PC4** | NIC | 192.168.30.12/24 |
| **Servidor DNS** | NIC | 192.168.31.12/24 |

---

## Parte 1: Verificar conectividad local y probar la ACL

### Paso 1: Ping a dispositivos en la red local

| Comando | Destino | Resultado |
|---------|---------|-----------|
| `ping 192.168.10.11` | PC2 | ✅ Éxito |
| `ping 192.168.11.10` | PC3 | ✅ Éxito |

**¿Por qué funcionan los pings?**
- PC1, PC2 y PC3 están en **redes locales** de R1 (conectadas directamente a G0/0 y G0/1).
- La ACL 11 está aplicada solo en **Serial0/0/0** en dirección **OUT**, por lo que **no afecta** el tráfico local.

---

### Paso 2: Ping a dispositivos en redes remotas

| Comando | Destino | Resultado |
|---------|---------|-----------|
| `ping 192.168.30.12` | PC4 | ❌ Falla |
| `ping 192.168.31.12` | Servidor DNS | ❌ Falla |

**¿Por qué fallan los pings?**
- La **ACL 11** está aplicada en la interfaz **Serial0/0/0** en dirección **OUT**.
- La ACL bloquea todo el tráfico que se origina en la red **192.168.10.0/24** (red de PC1).
- PC1 no puede enviar tráfico hacia redes remotas (PC4 y DNS).

---

## Verificación de la ACL con comandos show

### Comando: `show access-lists`

```bash
R1# show access-lists
Standard IP access list 11
    10 deny   192.168.10.0 0.0.0.255
    20 permit any
