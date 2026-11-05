# 🌐 Cisco Packet Tracer - Troubleshooting Challenge (12.6.2)

## 📋 Descripción del Laboratorio

Este repositorio contiene la documentación y solución del laboratorio **"Troubleshooting Challenge - Use Documentation to Solve Issues"** de Cisco Networking Academy.

El objetivo del laboratorio es diagnosticar y reparar problemas de conectividad en una red corporativa preconfigurada que ha sufrido fallas. Se debe garantizar que todos los hosts (PC1 a PC7) puedan comunicarse entre sí y con el Servidor de Internet (`203.0.113.27`).

## 🛠️ Herramientas y Tecnologías Utilizadas

- **Software:** Cisco Packet Tracer
- **Protocolos:** ICMP, CDP, OSPF, NAT, ARP
- **Conceptos:** Enrutamiento estático, Subinterfaces, VLSM (máscaras de longitud variable /25, /27, /30), Troubleshooting

## 🔍 Diagnóstico y Problemas Encontrados

El laboratorio presenta múltiples fallas ocultas en diferentes dispositivos de la topología. Aquí está el desglose de los errores encontrados y cómo se solucionaron:

### 1. Router ISP (Proveedor de Internet)

- **Problema:** El tráfico hacia Internet se perdía. El router ISP no tenía configurada una ruta por defecto para devolver el tráfico a las redes internas (`192.168.x.x`). El mensaje en consola era `Gateway of last resort is not set`.
- **Solución:** Se configuró una ruta estática por defecto apuntando al router central (Hub).
- **Comando:**

```text
ip route 0.0.0.0 0.0.0.0 192.0.2.1
```

### 2. Router Hub (Núcleo de la red)

- **Problema A:** La interfaz `GigabitEthernet0/0/1` (conexión hacia las PCs 4 y 5) estaba administrativamente apagada (`administratively down`).
  - **Solución:** Se encendió la interfaz con el comando `no shutdown`.
- **Problema B:** Faltaban rutas estáticas para llegar a las subredes de la PC6 y PC7 (`192.168.2.32/27` y `192.168.2.64/27`).
  - **Solución:** Se agregaron rutas estáticas apuntando a la interfaz Serial del router Branch-2.
  - **Comando:**

```text
ip route 192.168.2.32 255.255.255.224 Serial0/1/1 192.168.0.17
ip route 192.168.2.64 255.255.255.224 Serial0/1/1 192.168.0.17
```

### 3. Router Factory

- **Problema:** La interfaz `GigabitEthernet0/0/1` (red de la PC3) estaba apagada. Por esto, la PC3 no podía salir de su red.
- **Solución:** Se encendió la interfaz con `no shutdown`.

### 4. Router Branch-2 (Red de PC6 y PC7)

- **Problema:** Al igual que el ISP, no tenía una ruta por defecto para enviar el tráfico hacia el resto de la red y a Internet.
- **Solución:** Se configuró la ruta por defecto apuntando al router Hub.
- **Comando:**

```text
ip route 0.0.0.0 0.0.0.0 192.168.0.5
```

### 5. Configuraciones de Hosts (PC5)

- **Problema:** La PC5 no podía salir de su red local porque no tenía configurado un Default Gateway (`0.0.0.0`).
- **Solución:** Utilizando la técnica de descubrimiento de vecinos con el comando `show cdp neighbors detail` desde un router, se descubrió que la subinterfaz del router HQ para la red de la PC5 era la `192.168.5.129`. Se procedió a configurar esta IP como Gateway en la PC5.

## 📚 Comandos de Diagnóstico Utilizados

Durante la resolución del laboratorio, se utilizaron las siguientes herramientas de Cisco IOS para el troubleshooting:

- **Verificar conectividad:** `ping [IP]` y `tracert [IP]`
- **Descubrir vecinos y topología:** `show cdp neighbors detail` (fundamental para encontrar la IP del Gateway perdido)
- **Ver estado de las interfaces:** `show ip interface brief` (permite detectar interfaces `administratively down`)
- **Ver tabla de enrutamiento:** `show ip route` (permite verificar la presencia de rutas `C`, `S`, `O` y la ruta por defecto `S*`)

## ✅ Resultado Final

Tras aplicar las soluciones, se verificó que:

1. Todas las PCs (`192.168.1.x` a `192.168.6.x`) pueden hacer `ping` exitosamente entre sí.
2. Todas las PCs pueden hacer `ping` exitosamente al Servidor de Internet (`203.0.113.27`).

> **Nota:** Debido a un conocido bug de simulación en Packet Tracer con las subinterfaces, el comando `tracert` hacia la PC5 puede mostrar un asterisco de "Request timed out" en el último salto; sin embargo, el comando `ping` confirma la conectividad total extremo a extremo.
