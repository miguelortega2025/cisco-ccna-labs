# Bitácora de Laboratorio: Configuración de ACLs Extendidas IPv4 — Escenario 2

**Título:** Implementación de políticas de control de acceso para segmentación de servicios en red empresarial  
**Módulo:** Networking Cisco Academy — Packet Tracer  
**Práctica:** 5.4.13 — Configure Extended IPv4 ACLs — Scenario 2  
**Fecha:** [Fecha de realización]  
**Responsable:** [Tu nombre]  
**Herramienta:** Cisco Packet Tracer  

---

## 1. Contexto Empresarial

En un entorno corporativo, no todos los usuarios deben tener acceso a todos los servicios ni a todos los servidores. La organización ficticia de este escenario cuenta con:

- **3 estaciones de trabajo (PC1, PC2, PC3)** en una LAN corporativa `172.31.1.96/27`
- **2 servidores remotos (Server1 y Server2)** en la nube/internet, que ofrecen servicios web (HTTP/HTTPS) y transferencia de archivos (FTP)
- **1 router perimetral (RT1)** que conecta la LAN con el exterior

Sin controles de acceso, cualquier empleado podría acceder a cualquier servicio de cualquier servidor, lo que representa riesgos de seguridad, uso indebido de recursos y posibles incidentes de confidencialidad.

---

## 2. Problema de Negocio Identificado

La empresa necesita **aplicar el principio de mínimo privilegio**: cada usuario debe tener acceso únicamente a los servicios que su rol requiere, y nada más.

Específicamente, se identificaron los siguientes riesgos:

| Riesgo | Impacto en el negocio |
|---|---|
| PC1 (estación de un área específica) navega a sitios web de servidores críticos | Exposición a contenido no autorizado, consumo de ancho de banda, posibles descargas maliciosas |
| PC2 (estación de un área operativa) accede por FTP a servidores remotos | Fuga de información, transferencia no controlada de archivos, incumplimiento normativo |
| PC3 (estación de monitoreo) puede hacer ping a servidores externos | Reconocimiento de red no autorizado, posible mapeo de infraestructura por parte de un atacante interno |

**En resumen:** La empresa necesita **segmentar el acceso a servicios** según el rol de cada equipo, sin afectar la conectividad general ni otros servicios legítimos.

---

## 3. Solución Implementada

Se implementó una **ACL extendida nombrada** llamada `LimitedAccess` en el router perimetral **RT1**, aplicada en la interfaz **G0/0** (la más cercana al origen del tráfico) en **dirección entrante (`in`)**.

### 3.1 Política de seguridad definida

| Usuario/Equipo | Servicio restringido | Destino | Justificación de negocio |
|---|---|---|---|
| PC1 | HTTP (80) y HTTPS (443) | Server1 y Server2 | Evitar navegación web no autorizada desde esa estación |
| PC2 | FTP (21) | Server1 y Server2 | Impedir transferencia de archivos no controlada |
| PC3 | ICMP (ping) | Server1 y Server2 | Prevenir reconocimiento de red externa |

### 3.2 Reglas configuradas (orden crítico)

```cisco
ip access-list extended LimitedAccess
 deny tcp host 172.31.1.101 host 64.101.255.254 eq 80
 deny tcp host 172.31.1.101 host 64.101.255.254 eq 443
 deny tcp host 172.31.1.101 host 64.103.255.254 eq 80
 deny tcp host 172.31.1.101 host 64.103.255.254 eq 443
 deny tcp host 172.31.1.102 host 64.101.255.254 eq 21
 deny tcp host 172.31.1.102 host 64.103.255.254 eq 21
 deny icmp host 172.31.1.103 host 64.101.255.254
 deny icmp host 172.31.1.103 host 64.103.255.254
 permit ip any any
```

**Nota empresarial:** La regla `permit ip any any` final es esencial para no bloquear el resto del tráfico legítimo (correo, DNS, actualizaciones, etc.). Sin ella, el "deny implícito" de toda ACL cortaría toda comunicación no listada, afectando la operación del negocio.

### 3.3 Aplicación en el router

```cisco
interface g0/0
 ip access-group LimitedAccess in
```

**Justificación técnica-empresarial:** Aplicar la ACL en la interfaz **más cercana al origen** evita que el tráfico no deseado cruce la red corporativa, ahorrando ancho de banda y recursos del router.

---

## 4. Resultados Obtenidos

### 4.1 Matriz de acceso resultante

| Origen | HTTP/HTTPS | FTP | ICMP (ping) |
|---|---|---|---|
| PC1 | ❌ Bloqueado | ✅ Permitido | ✅ Permitido |
| PC2 | ✅ Permitido | ❌ Bloqueado | ✅ Permitido |
| PC3 | ✅ Permitido | ✅ Permitido | ❌ Bloqueado |

### 4.2 Verificación con contadores

```cisco
RT1# show ip access-lists
```

Los contadores de coincidencias confirmaron que las reglas se activaron correctamente al generar tráfico de prueba desde cada PC.

---

## 5. Beneficios Empresariales Obtenidos

| Beneficio | Descripción |
|---|---|
| **Seguridad perimetral** | Se redujo la superficie de ataque limitando servicios por usuario |
| **Cumplimiento normativo** | Se alineó el acceso con políticas de mínimo privilegio (ISO 27001, NIST) |
| **Optimización de recursos** | El tráfico no deseado se bloquea antes de cruzar la red |
| **Trazabilidad** | Los contadores de la ACL permiten auditar intentos de acceso |
| **Escalabilidad** | La ACL nombrada permite agregar/modificar reglas sin rehacer la configuración |
| **Continuidad operativa** | El `permit ip any any` garantiza que los servicios legítimos no se vean afectados |

---

## 6. Lecciones Aprendidas

1. **El orden de las reglas es crítico:** una ACL se evalúa de arriba hacia abajo y la primera coincidencia gana.
2. **El deny implícito existe:** toda ACL bloquea todo lo no permitido explícitamente.
3. **La ubicación importa:** las ACL extendidas van cerca del origen, no del destino.
4. **La dirección importa:** `in` filtra antes de que el router procese el paquete; `out` lo hace después.
5. **Probar antes de aplicar:** en producción, una ACL mal configurada puede tumbar servicios críticos.
6. **Documentar con `remark`:** facilita el mantenimiento y la auditoría por otros administradores.
7. **Las ACLs son una herramienta de negocio, no solo técnica:** traducen políticas de seguridad en reglas ejecutables.

---

## 7. Recomendaciones para Producción

- **Agregar comentarios (`remark`)** a cada regla para documentar su propósito de negocio.
- **Revisar periódicamente** los contadores de coincidencias para detectar intentos de acceso no autorizados.
- **Combinar con otras capas de seguridad:** firewall, IDS/IPS, autenticación 802.1X.
- **Versionar la configuración** de las ACLs en un repositorio de cambios.
- **Probar en un entorno de laboratorio** antes de aplicar en la red productiva.
- **Considerar ACLs dinámicas o basadas en tiempo** si las políticas cambian según horario o rol.

---

## 8. Conclusión

Este laboratorio demostró cómo una **ACL extendida nombrada** puede traducir una **política de seguridad empresarial** en reglas concretas de filtrado en el router perimetral. Más allá de la sintaxis Cisco, el valor real está en:

- **Proteger activos de información** limitando el acceso a servicios según el rol del usuario.
- **Reducir riesgos** de fuga de datos, reconocimiento de red y uso indebido de recursos.
- **Mantener la operatividad** del negocio permitiendo el tráfico legítimo no restringido.

La configuración de ACLs no es un fin en sí mismo, sino un **control técnico que materializa decisiones de negocio** sobre quién puede acceder a qué, desde dónde y hacia dónde.

---


---

