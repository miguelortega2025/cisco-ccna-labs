 PROBLEMA PENDIENTE - LAB 4.2.4.4
🎯 CONTEXTO
Laboratorio: Connecting a Wired and Wireless LAN

Fecha: [Fecha actual]

Estado: Parcialmente completado (90%)

✅ LO QUE SÍ FUNCIONA
✅ Conexiones físicas correctas (todos los cables verdes)

✅ Configuración IP básica en todos los dispositivos

✅ Ping local dentro de cada red:

Family PC → Wireless Router (192.168.1.1) ✅

netacad.pka → Router0 (10.0.0.1) ✅

✅ Ping a través de routers simples:

netacad.pka → Switch (172.16.0.2) ✅

Router0 → Router1 (172.31.0.2) ✅

✅ Rutas estáticas configuradas en Router0 y Router1

❌ PROBLEMA ESPECÍFICO
Descripción: Ping de netacad.pka (10.0.0.254) a Family PC (192.168.1.102) FALLA

Síntoma: Request timed out o Destination host unreachable

Paradoja: Ping en sentido contrario (Family PC → Router0) tampoco funciona

🔍 DIAGNÓSTICO PRELIMINAR
Causas probables:
Firewall/NAT en Wireless Router (más probable)

Router doméstico bloquea pings entrantes por defecto

SPI Firewall activado

No responde a ping en interfaz WAN

Falta gateway WAN en Wireless Router

Aunque configurado como Static IP (192.168.2.2), puede no tener gateway

Problema de rutas asimétricas

Paquete llega vía Router0 → Wireless Router

Respuesta intenta otra ruta (si hay múltiples caminos)

Evidencia:
Wireless Router no tiene CLI en Packet Tracer

Pestaña Config limitada (no muestra opciones de routing avanzado)

Opciones de Firewall pueden estar en pestaña GUI o Security

🛠️ INTENTOS REALIZADOS
✅ Verificado gateway en Family PC (192.168.1.1)

✅ Rutas estáticas en Router0:

bash
ip route 192.168.1.0 255.255.255.0 192.168.2.2
ip route 172.16.0.0 255.255.255.0 172.31.0.2
✅ Rutas estáticas en Router1:

bash
ip route 10.0.0.0 255.255.255.0 172.31.0.1
✅ Interfaces activas (no shutdown) en todos los routers

❌ Wireless Router no permite configurar rutas estáticas (interfaz limitada)

❌ No se encontró opción "Respond to Ping on WAN" en Config

🎯 PARA INVESTIGAR CUANDO VUELVAS
Temas a repasar:
NAT (Network Address Translation) en routers domésticos

Stateful vs Stateless Firewalls

Port Forwarding / DMZ para pruebas

Router doméstico vs Router empresarial (diferencias de funcionalidad)

Preguntas clave:
¿Cómo configurar un Wireless Router en PT para permitir ping entrante?

¿Dónde está la opción de firewall en la GUI simulada?

¿Es necesario NAT para esta topología o se puede usar routing puro?

📁 ARCHIVOS GUARDADOS
Packet Tracer file: 4.2.4.4-mi-configuracion.pkt

Capturas de:

show ip route de Router0 y Router1

Configuración Wireless Router (Interfaces)

Tabla ARP de Family PC

Resultados de ping fallidos

🔗 RELACIÓN CON TEMAS CCNA
Módulo 5: Seguridad de dispositivos

Módulo 6: NAT

Módulo 7: Routing estático avanzado

Módulo 8: Troubleshooting metodológico

💡 POSIBLES SOLUCIONES A INTENTAR DESPUÉS
Reemplazar Wireless Router por un router Cisco 1841 (tiene CLI)

Configurar NAT explícito si el Wireless Router lo permite

Usar Port Forwarding del puerto ICMP (poco común)

Crear DMZ con Family PC

Desactivar completamente el firewall (encontrar la opción en GUI)

📝 NOTAS ADICIONALES
Este problema parece más de simulación de Packet Tracer que de concepto

En vida real, se configuraría el firewall del router doméstico

Puede ser un bug conocido de PT con routers domésticos simulados

Prioridad baja para CCNA: conceptos más importantes ya dominados
