#  Bitácora de Aprendizaje: IPv4 ACL Implementation Challenge
## Enfoque de Negocio + Técnico

---

##  Información General

| Campo | Detalle |
|---|---|
| **Laboratorio** | Packet Tracer - IPv4 ACL Implementation Challenge |
| **Tema** | Listas de Control de Acceso (ACLs) IPv4 |
| **Duración estimada** | 2–3 horas |
| **Nivel** | Intermedio |
| **Enfoque** | Resolución de problemas de negocio mediante políticas de red |

---

##  Objetivo de Negocio

> Diseñar e implementar **políticas de seguridad de red** que protejan los activos de información de una empresa con dos sitios (HQ y Branch), conexión a Internet y múltiples departamentos, usando ACLs de Cisco como mecanismo de cumplimiento.

**Pregunta central:**
> ¿Cómo traduzco las **reglas del negocio** (quién puede ver qué, quién puede administrar qué) en **configuración de red** que se cumpla automáticamente?

---

##  Contexto de Negocio (Escenario)

### La empresa

**"TechCorp"** es una empresa con:

- **Oficina Central (HQ):** 3 departamentos + servidor web corporativo.
- **Sucursal (Branch):** 2 departamentos + servidor de archivos financieros.
- **Presencia en Internet:** usuarios externos y servidores públicos.

### Activos críticos a proteger

| Activo | Ubicación | Valor para el negocio |
|---|---|---|
| Enterprise Web Server | HQ LAN 2 | Imagen pública, clientes |
| Branch Server | Branch LAN 2 | Datos financieros |
| Router HQ | HQ | Corazón de la red |
| HQ LAN 1 | HQ | 100+ empleados generales |
| Branch LANs | Branch | Empleados de sucursal |

### Riesgos identificados por el negocio

| # | Riesgo | Impacto potencial |
|---|---|---|
| R1 | Ataque externo a servidores | Fuga de datos, multas |
| R2 | Ransomware desde departamento general | Paralización, rescate |
| R3 | Acceso no autorizado al router | Caída total de red |
| R4 | Fuga de datos entre sitios | Incumplimiento normativo |
| R5 | FTP de sucursal expuesto | Descarga no autorizada |

---

##  Topología y Direccionamiento

### Tabla de direcciones

| Dispositivo | Interfaz | IP | Red |
|---|---|---|---|
| **HQ** | G0/0/0 | 192.168.1.1/26 | HQ LAN 1 |
| HQ | G0/0/1 | 192.168.1.65/29 | HQ LAN 2 |
| HQ | S0/1/0 | 192.0.2.1/30 | WAN Internet |
| HQ | S0/1/1 | 192.168.3.1/30 | WAN Branch |
| **Branch** | G0/0/0 | 192.168.2.1/27 | Branch LAN 1 |
| Branch | G0/0/1 | 192.168.2.33/28 | Branch LAN 2 |
| Branch | S0/1/1 | 192.168.3.2/30 | WAN HQ |
| PC-1/2/3 | NIC | 192.168.1.10/20/30 | HQ LAN 1 |
| Admin | NIC | 192.168.1.67/29 | HQ LAN 2 |
| Enterprise Web Server | NIC | 192.168.1.70/29 | HQ LAN 2 |
| Branch PC | NIC | 192.168.2.17/27 | Branch LAN 1 |
| Branch Server | NIC | 192.168.2.45/28 | Branch LAN 2 |
| Internet User | NIC | 198.51.100.218/24 | Internet |
| External Web Server | NIC | 203.0.113.73/24 | Internet |

### Cálculo de wildcards (herramienta de negocio)

| Red | Máscara | Wildcard | Uso |
|---|---|---|---|
| HQ LAN 1 | /26 | 0.0.0.63 | Departamento general |
| HQ LAN 2 | /29 | 0.0.0.7 | TI + servidores |
| Branch LAN 1 | /27 | 0.0.0.31 | Empleados sucursal |
| Branch LAN 2 | /28 | 0.0.0.15 | Servidor financiero |

**Fórmula:** Wildcard = 255.255.255.255 − Máscara

---

##  Fase 1: Verificación de Conectividad Inicial

###  Propósito de negocio
> Establecer una **línea base** antes de aplicar políticas. Si algo falla después, sabremos que fue la ACL y no un problema preexistente.

### Estrategia eficiente (no probar 500 hosts)

| Segmento | Host de prueba | Destino | Propósito |
|---|---|---|---|
| HQ LAN 1 | PC-1 | Enterprise Web Server | Verificar intra-HQ |
| HQ LAN 2 | Admin | PC-1 | Verificar bidireccional |
| Branch LAN 1 | Branch PC | PC-1 | Verificar WAN Branch-HQ |
| Branch LAN 2 | Branch Server | Admin | Verificar servidores |
| Internet | Internet User | External Web Server | Verificar salida a Internet |

### Comandos de verificación

```cisco
show ip route
show ip interface brief
ping <IP_destino>
traceroute <IP_destino>
```

###  Resultado esperado
Todos los pings deben ser exitosos. Si alguno falla, **detenerse y corregir enrutamiento** antes de continuar.

---

##  Fase 2: Implementación de Políticas de Negocio

---

###  POLÍTICA 1: Proteger servidores de ataques externos

#### Problema de negocio
> **R1:** Internet puede atacar servidores y mapear la red interna.

#### Requisito traducido a política
> "Desde Internet: bloquear FTP al servidor web corporativo, bloquear ICMP hacia HQ LAN 1, permitir todo lo demás."

#### Decisión de diseño

| Pregunta | Respuesta | Justificación de negocio |
|---|---|---|
| ¿Qué tipo de ACL? | Extendida (101) | Necesito filtrar por protocolo y puerto |
| ¿Dónde aplicarla? | S0/1/0 del HQ `in` | Cerca del origen (Internet) para ahorrar recursos |
| ¿Por qué `in`? | Filtrar antes de procesar | Eficiencia y seguridad temprana |

#### Comandos

```cisco
HQ(config)# access-list 101 deny tcp any host 192.168.1.70 eq 21
HQ(config)# access-list 101 deny icmp any 192.168.1.0 0.0.0.63
HQ(config)# access-list 101 permit ip any any

HQ(config)# interface S0/1/0
HQ(config-if)# ip access-group 101 in
```

#### Explicación de cada línea

| Línea | Traducción de negocio |
|---|---|
| `deny tcp any host 192.168.1.70 eq 21` | "Nadie en Internet puede descargar archivos por FTP del servidor web" |
| `deny icmp any 192.168.1.0 0.0.0.63` | "Nadie en Internet puede hacer ping a los empleados de HQ LAN 1" |
| `permit ip any any` | "Todo lo demás (HTTP, HTTPS, correo) sigue funcionando" |

#### Impacto de negocio
-  Clientes siguen viendo la web.
-  Atacantes no pueden descargar archivos internos.
-  Atacantes no pueden mapear la red con ping.
-  Cumplimiento de normativas de seguridad.

---

###  POLÍTICA 2: Aislar departamento general de datos financieros

#### Problema de negocio
> **R2:** Ransomware desde departamento general puede contagiar al servidor financiero.

#### Requisito traducido a política
> "Ningún host de HQ LAN 1 puede acceder al Branch Server. Todo lo demás permitido."

#### Decisión de diseño

| Pregunta | Respuesta | Justificación de negocio |
|---|---|---|
| ¿Qué tipo de ACL? | Extendida (111) | Necesito filtrar por destino específico |
| ¿Dónde aplicarla? | G0/0/0 del HQ `in` | Cerca del origen (HQ LAN 1) |
| ¿Por qué ahí? | Bloquear justo al salir | No cruza la WAN innecesariamente |

#### Comandos

```cisco
HQ(config)# access-list 111 deny ip 192.168.1.0 0.0.0.63 host 192.168.2.45
HQ(config)# access-list 111 permit ip any any

HQ(config)# interface G0/0/0
HQ(config-if)# ip access-group 111 in
```

#### Explicación de cada línea

| Línea | Traducción de negocio |
|---|---|
| `deny ip 192.168.1.0 0.0.0.63 host 192.168.2.45` | "Los empleados generales no pueden tocar el servidor financiero" |
| `permit ip any any` | "Pueden seguir usando Internet, correo, impresoras, etc." |

#### Impacto de negocio
-  Segmentación efectiva.
-  Contención de ransomware.
-  Principio de mínimo privilegio.
-  Si HQ LAN 1 se compromete, el servidor financiero está a salvo.

---

###  POLÍTICA 3: Proteger la administración del router

#### Problema de negocio
> **R3:** Cualquiera puede administrar el router y tumbar la red.

#### Requisito traducido a política
> "Solo la red de TI (HQ LAN 2) puede hacer Telnet/SSH al router HQ."

#### Decisión de diseño

| Pregunta | Respuesta | Justificación de negocio |
|---|---|---|
| ¿Qué tipo de ACL? | Estándar nombrada `vty_block` | Solo necesito filtrar por origen |
| ¿Dónde aplicarla? | Líneas VTY del HQ | Cerca del destino (el propio router) |
| ¿Por qué estándar? | Solo importa quién se conecta | No hay destino variable |

#### Comandos

```cisco
HQ(config)# ip access-list standard vty_block
HQ(config-std-nacl)# permit 192.168.1.64 0.0.0.7

HQ(config)# line vty 0 4
HQ(config-line)# access-class vty_block in
```

#### Explicación de cada línea

| Línea | Traducción de negocio |
|---|---|
| `ip access-list standard vty_block` | "Creo la política de acceso administrativo" |
| `permit 192.168.1.64 0.0.0.7` | "Solo el personal de TI (HQ LAN 2) puede administrar" |
| `line vty 0 4` | "Aplico a las 5 líneas de acceso remoto" |
| `access-class vty_block in` | "Filtro quién puede conectarse" |

#### Diferencia clave: `access-class` vs `access-group`

| Comando | ¿Dónde? | ¿Qué filtra? |
|---|---|---|
| `ip access-group` | Interfaces | Tráfico que **atraviesa** el router |
| `access-class` | Líneas VTY | Tráfico **destinado al propio router** |

#### Impacto de negocio
-  Solo TI puede administrar el router.
-  Empleados generales no pueden ni intentar.
-  Atacantes externos sin puerta de entrada.
-  Auditoría limpia.

---

###  POLÍTICA 4: Aislar datos entre sitios

#### Problema de negocio
> **R4:** La sucursal no debería ver los datos de la oficina central.

#### Requisito traducido a política
> "Ningún host de las dos Branch LANs puede acceder a HQ LAN 1. Una sentencia por cada Branch LAN. Todo lo demás permitido."

#### Decisión de diseño

| Pregunta | Respuesta | Justificación de negocio |
|---|---|---|
| ¿Qué tipo de ACL? | Extendida nombrada `branch_to_hq` | Filtra origen y destino |
| ¿Dónde aplicarla? | S0/1/1 del HQ `in` | Cerca del origen (Branch) |
| ¿Por qué nombrada? | El lab lo pide | Claridad semántica |

#### Comandos

```cisco
HQ(config)# ip access-list extended branch_to_hq
HQ(config-ext-nacl)# deny ip 192.168.2.0 0.0.0.31 192.168.1.0 0.0.0.63
HQ(config-ext-nacl)# deny ip 192.168.2.32 0.0.0.15 192.168.1.0 0.0.0.63
HQ(config-ext-nacl)# permit ip any any

HQ(config)# interface S0/1/1
HQ(config-if)# ip access-group branch_to_hq in
```

#### Explicación de cada línea

| Línea | Traducción de negocio |
|---|---|
| Primera `deny` | "Empleados de Branch LAN 1 no ven HQ LAN 1" |
| Segunda `deny` | "Servidor financiero de Branch LAN 2 no ve HQ LAN 1" |
| `permit ip any any` | "Pueden seguir accediendo a Internet y a HQ LAN 2" |

#### Impacto de negocio
-  Aislamiento entre sitios.
-  Datos sensibles contenidos en HQ.
-  Cumplimiento normativo.
-  Reducción de superficie de ataque.

---

##  Fase 3: Verificación de Políticas

###  Propósito de negocio
> Confirmar que las **políticas de negocio** se cumplen y que no hay **brechas** no detectadas.

### Herramientas de verificación

```cisco
show ip access-lists
show ip interface S0/1/0
show ip interface G0/0/0
show ip interface S0/1/1
show running-config | section line vty
clear access-list counters
```

### Pruebas de política

| # | Prueba | Resultado esperado | Política validada |
|---|---|---|---|
| 1 | Branch PC → ping → Enterprise Web Server | ✅ Exitoso | Branch puede ver HQ LAN 2 |
| 2 | PC-1 → ping → Branch Server | ❌ Fallido | ACL 111 bloquea |
| 3 | External Server → HTTP → Enterprise Web Server | ✅ Exitoso | ACL 101 permite HTTP |
| 4 | Internet User → FTP → Branch Server | ❌ Fallido (tras corrección) | ACL 101 bloquea FTP |

---

##  Fase 4: Descubrimiento de Brecha de Seguridad

###  Problema de negocio no documentado
> **R5:** El FTP del Branch Server está expuesto a Internet.

### Situación
Al probar **FTP desde Internet User al Branch Server**, la conexión **fue exitosa**. Esto significa que:

- No había ninguna ACL que lo bloqueara.
- **Cualquiera en Internet** podía descargar archivos financieros.
- **Brecha de seguridad crítica** no documentada.

### Análisis de negocio

| Pregunta | Respuesta |
|---|---|
| ¿Qué ACL debe modificarse? | ACL 101 (filtra desde Internet) |
| ¿Qué sentencia agregar? | `deny tcp any host 192.168.2.45 eq 21` |
| ¿Dónde insertarla? | Antes del `permit ip any any` |

### Corrección

Como las ACLs numeradas no permiten insertar en medio, se reescribe:

```cisco
HQ(config)# no access-list 101
HQ(config)# access-list 101 deny tcp any host 192.168.1.70 eq 21
HQ(config)# access-list 101 deny icmp any 192.168.1.0 0.0.0.63
HQ(config)# access-list 101 deny tcp any host 192.168.2.45 eq 21
HQ(config)# access-list 101 permit ip any any
```

### Impacto de negocio
-  Brecha cerrada.
-  Datos financieros protegidos.
-  Cumplimiento restaurado.

---

##  Resumen: Problema → Política → Comando

| # | Problema de negocio | Política | ACL | Ubicación |
|---|---|---|---|---|
| R1 | Ataque externo | Bloquear FTP e ICMP desde Internet | 101 | HQ S0/1/0 in |
| R2 | Ransomware | Aislar HQ LAN 1 de Branch Server | 111 | HQ G0/0/0 in |
| R3 | Sabotaje | Solo TI administra router | vty_block | HQ line vty in |
| R4 | Fuga entre sitios | Aislar Branch de HQ LAN 1 | branch_to_hq | HQ S0/1/1 in |
| R5 | FTP expuesto | Bloquear FTP a Branch Server | 101 (mod.) | HQ S0/1/0 in |

---

##  Lecciones Aprendidas

### Técnicas

1. **ACL estándar vs extendida:** estándar filtra solo origen, extendida filtra origen, destino, protocolo y puerto.
2. **Ubicación:** estándar cerca del destino, extendida cerca del origen.
3. **Dirección:** `in` es más eficiente que `out`.
4. **Wildcards:** se calculan como 255.255.255.255 − máscara.
5. **`deny any` implícito:** toda ACL lo tiene al final.
6. **`access-class` vs `access-group`:** uno para VTY, otro para interfaces.
7. **Orden secuencial:** la primera coincidencia gana.
8. **Contadores:** `show ip access-lists` muestra coincidencias.

### De negocio

1. **Las ACLs son políticas de negocio escritas en la red.**
2. **Cada línea responde a una pregunta de negocio.**
3. **La seguridad no es un proceso lineal:** siempre hay brechas nuevas (R5).
4. **Segmentación = contención:** si un segmento se compromete, los demás están a salvo.
5. **Mínimo privilegio:** cada quien accede solo a lo que necesita.
6. **Cumplimiento normativo:** las ACLs son evidencia de controles.
7. **El costo de no hacerlo:** multas, rescates, caídas, pérdida de confianza.

---

##  Traducción a Dinero

| Riesgo | Costo potencial sin ACL | Costo con ACL |
|---|---|---|
| Fuga de datos de clientes | Millones en multas | $0 |
| Ransomware en servidor financiero | $500K+ rescate | $0 |
| Caída de red por sabotaje | $10K–$100K/hora | $0 |
| FTP expuesto | Fuga silenciosa | $0 |
| Auditoría fallida | Pérdida de certificaciones | Cumplimiento OK |

---

##  Checklist de Cumplimiento

- [ ] Conectividad inicial verificada
- [ ] ACL 101 configurada y aplicada
- [ ] ACL 111 configurada y aplicada
- [ ] ACL `vty_block` configurada y aplicada
- [ ] ACL `branch_to_hq` configurada y aplicada
- [ ] Contadores verificados
- [ ] Brecha de FTP descubierta y cerrada
- [ ] Todas las políticas de negocio cumplidas

---

##  Anexos

### A. Comandos de verificación rápida

```cisco
show ip access-lists
show ip interface <interfaz>
show running-config | include access
clear access-list counters
```

### B. Tabla de puertos comunes

| Puerto | Protocolo | Uso |
|---|---|---|
| 21 | TCP | FTP |
| 22 | TCP | SSH |
| 23 | TCP | Telnet |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |

### C. Fórmula de wildcard

```
Wildcard = 255.255.255.255 − Máscara de subred
```

| Máscara | Wildcard |
|---|---|
| /24 | 0.0.0.255 |
| /26 | 0.0.0.63 |
| /27 | 0.0.0.31 |
| /28 | 0.0.0.15 |
| /29 | 0.0.0.7 |
| /30 | 0.0.0.3 |

---

##  Reflexión Final

> Este laboratorio no se trata de memorizar comandos. Se trata de **traducir reglas de negocio a configuración de red** que se cumpla automáticamente, proteja los activos de la empresa y permita demostrar cumplimiento ante auditorías.

**La pregunta que todo administrador de red debe hacerse:**
> "¿Qué regla de negocio estoy implementando con esta ACL, y qué pasaría si no estuviera?"
