# 📋 Bitácora de Laboratorio: Investigación de Operaciones NAT

**Módulo:** 6.2.7 – Packet Tracer: Investigate NAT Operations
**Área:** Infraestructura de Redes
**Enfoque:** Impacto de negocio y continuidad operativa
**Responsable:** [Tu nombre]
**Fecha:** [Fecha]

---

## 1. 🎯 Propósito del laboratorio

Evaluar cómo la implementación de **NAT (Network Address Translation)** permite a una organización **conectar sus redes internas con Internet** sin necesidad de asignar una dirección IP pública a cada dispositivo, resolviendo así problemas de **escasez de direcciones, costos operativos y exposición de la red interna**.

---

## 2. 🏢 Contexto de negocio

La empresa cuenta con dos sedes:

| Sede | Rol | Red interna |
|------|-----|-------------|
| **Central** | Oficina principal con servidores y estaciones de trabajo | `10.10.10.0/24` |
| **Home Office** | Oficina remota con empleados conectados | `192.168.0.0/24` |

Ambas sedes necesitan:

- Acceder a **Internet** para servicios en la nube, correo y videollamadas.
- Comunicarse entre sí para compartir **servidores internos** (ej. `branchserver.pka`, `centralserver.pka`).
- Hacerlo **sin comprar una IP pública por cada empleado o dispositivo**.
- Mantener la **red interna oculta** ante amenazas externas.

---

## 3. 🧩 Problema de negocio que NAT resuelve

| Problema | Impacto en el negocio | ¿Cómo lo resuelve NAT? |
|----------|----------------------|------------------------|
| **Escasez de IPs públicas IPv4** | Costos elevados por comprar bloques de IPs | Permite que cientos de dispositivos compartan una o pocas IPs públicas |
| **Exposición de la red interna** | Riesgo de ataques directos a servidores y PCs | Oculta las IPs privadas detrás de una IP pública |
| **Complejidad de configuración** | Cada dispositivo necesitaría IP pública y configuración individual | Centraliza la salida a Internet en el router de borde |
| **Falta de escalabilidad** | Agregar un empleado implicaría comprar otra IP pública | NAT escala sin necesidad de nuevas IPs públicas |
| **Costos de ISP** | Los ISPs cobran por IP pública adicional | NAT reduce el costo a una sola IP pública por sede |

---

## 4. 🔍 Desarrollo de la investigación

### Parte 1: Operación NAT en la intranet

**Escenario:** Un empleado de la sede Central accede a `http://branchserver.pka`.

**Observaciones:**

- El paquete sale de la PC con IP privada `10.10.10.x`.
- Al pasar por **R2**, el router traduce la IP privada a una IP pública del pool `R2Pool` (`64.100.100.3 – 64.100.100.31`).
- El servidor remoto ve únicamente la IP pública.
- La tabla NAT de R2 registra la traducción para el retorno.

**Valor de negocio:**

- La sede Central puede acceder a servidores remotos **sin exponer su red interna**.
- No se necesita una IP pública por cada empleado.

---

### Parte 2: Operación NAT a través de Internet

**Escenario:** Un empleado del Home Office accede a `http://centralserver.pka`.

**Observaciones:**

- El tráfico sale de la PC con IP privada `192.168.0.x`.
- **WRS** traduce a su IP pública `64.104.223.2` usando **PAT (overload)**.
- El paquete cruza Internet y llega a **R2**, que solo enruta (no traduce, porque el tráfico viene de outside hacia inside).
- El Central Server responde y el retorno se traduce de vuelta en WRS.

**Valor de negocio:**

- Los empleados remotos pueden **trabajar desde casa** sin VPN compleja ni IPs públicas dedicadas.
- La empresa ahorra en infraestructura y simplifica la gestión.

---

### Parte 3: Investigaciones adicionales

| Pregunta | Hallazgo | Implicación de negocio |
|----------|----------|------------------------|
| ¿Crecen las tablas NAT? | Sí, una entrada por conexión | El router debe tener recursos suficientes para tráfico concurrente |
| ¿WRS tiene pool? | No, usa PAT con 1 IP | Menor costo, pero limita conexiones simultáneas |
| ¿Así se conectan las aulas? | Sí, mismo concepto | Modelo replicable en toda la empresa |
| ¿Por qué 4 columnas? | Para traducir origen y destino | Permite auditoría y troubleshooting |
| ¿Dónde opera NAT? | En routers de borde (WRS, R2) | Punto único de control y seguridad |

---

## 5. 📊 Resultados de negocio

| Indicador | Antes de NAT | Después de NAT |
|-----------|-------------|----------------|
| IPs públicas necesarias | Una por dispositivo | Una por sede |
| Costo de IPs | Alto | Bajo |
| Exposición de red interna | Alta | Baja |
| Escalabilidad | Limitada | Alta |
| Complejidad de configuración | Alta | Centralizada |
| Capacidad de trabajo remoto | Limitada | Habilitada |

---

## 6. ✅ Conclusiones

1. **NAT no es solo una solución técnica, es una estrategia de negocio.** Permite operar con costos predecibles y escalables.
2. **La empresa puede conectar miles de dispositivos a Internet con una sola IP pública**, reduciendo costos de ISP y simplificando la administración.
3. **La red interna queda protegida de forma natural**, ya que los atacantes externos no pueden iniciar conexiones directas a los hosts internos.
4. **NAT es transparente para el usuario final**: los empleados navegan, acceden a servidores y trabajan remotamente sin saber que existe traducción.
5. **Es fundamental documentar las tablas NAT** para auditorías, troubleshooting y cumplimiento normativo.

---

## 7. 📌 Recomendaciones

- **Documentar las traducciones NAT** en caso de auditorías o incidentes.
- **Monitorear el tamaño de las tablas NAT** para evitar agotamiento en horas pico.
- **Complementar NAT con firewall** para protección avanzada (NAT no es seguridad por sí solo).
- **Evaluar IPv6** a largo plazo para eliminar la dependencia de NAT.
- **Capacitar al personal** en troubleshooting de NAT para reducir tiempos de inactividad.

---

## 8. 📎 Anexos

- Tabla de direccionamiento.
- Capturas de `show ip nat translations` en R2 y WRS.
- Diagrama de topología.
- Flujo de paquetes HTTP/HTTPS.
