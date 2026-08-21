##  ¿Qué conceptos clave entendí?

### 1. Las VLANs son como "edificios" dentro de un mismo terreno
- **VLAN 10 (Management):** Donde "vive" el switch para ser administrado.
- **VLAN 20 (Sales):** Donde está conectado PC-A.
- **VLAN 30 (Operations):** Donde está conectado PC-B.
- **VLAN 999 (ParkingLot):** Una "zona muerta" para puertos que no uso, así nadie se conecta ahí por error.
- **VLAN 1000 (Native):** La VLAN que viaja "sin etiqueta" por el trunk (la cambié de la default 1 a la 1000 por seguridad).

### 2. El Trunk es como una "autopista etiquetada"
- Cuando conecté S1 y S2 por el puerto F0/1, configuré ese puerto como **trunk**.
- Esto significa que por ese cable viajan **todas** las VLANs que permití (10, 20, 30, 1000), pero cada paquete lleva una "etiqueta" (tag 802.1Q) que indica a qué VLAN pertenece.
- Si no fuera trunk, solo podría pasar una VLAN por ese cable.

### 3. Los switches de capa 2 NO enrutan entre VLANs
- Aquí viene el "¡ajá!" del laboratorio.
- **PC-A** (192.168.20.13) y **PC-B** (192.168.30.13) están en VLANs diferentes.
- Aunque el trunk les permite viajar por el mismo cable, **no pueden hablarse** porque cada uno está en una subred diferente.
- Para que se comunicaran, necesitaría un **router** o un **switch de capa 3** con *Inter-VLAN Routing*.

---

##  ¿Qué hice paso a paso? (Sin comandos crudos)

### Parte 1: Preparación
- Asigné nombre a los switches (`S1` y `S2`).
- Configuré contraseñas cifradas para acceso seguro.
- Desactivé la búsqueda de DNS (para que no se ralentice al escribir mal un comando).
- Asigné las IPs a los PCs según la tabla.

### Parte 2: Creación de VLANs y asignación
- Creé las VLANs en ambos switches con sus nombres.
- Asigné el puerto F0/6 de S1 a la **VLAN 20** (modo acceso).
- Asigné el puerto F0/18 de S2 a la **VLAN 30** (modo acceso).
- Todos los puertos no usados los mandé a la **VLAN 999** y los apagué administrativamente (`shutdown`).

### Parte 3: Trunking
- Configuré el puerto F0/1 en ambos switches como **trunk**.
- Definí la VLAN nativa como **1000**.
- Restringí el trunk para que solo permita las VLANs 10, 20, 30 y 1000.

---

##  Resultados y reflexión

### Lo que SÍ funcionó
- Los puertos quedaron asignados correctamente a sus VLANs (`show vlan brief`).
- El trunk quedó activo y pasando tráfico etiquetado (`show interfaces trunk`).

### Lo que NO funcionó (y por qué está bien)
- **PC-A → S1 (192.168.10.11):**  Falló.
  - *¿Por qué?* PC-A está en VLAN 20 (subred 20) y S1 tiene su IP de administración en VLAN 10 (subred 10). Son mundos diferentes.
- **PC-B → S2 (192.168.10.12):**  Falló.
  - *¿Por qué?* PC-B está en VLAN 30 y S2 en VLAN 10. Mismo caso.

###  La gran lección
> **Un switch de capa 2 no es un router.**  
> Puede transportar VLANs por un trunk, pero **no puede comunicar** dispositivos de VLANs distintas. Para eso necesito un equipo de capa 3.

---

##  ¿Qué me llevo para el futuro?

- **Siempre verificar** que los puertos de acceso estén en modo `access` y no en modo dinámico (para evitar problemas).
- **La VLAN nativa** debe ser una VLAN dedicada y no usada (como la 1000), por seguridad.
- **El trunk** debe tener una lista explícita de VLANs permitidas (principio de mínimo privilegio).
- **Las SVIs** (interfaces virtuales del switch) solo son accesibles desde su propia VLAN.

---

##  Evidencias (Opcional)

> *Si quieres, agrega capturas de:*
> - `show vlan brief` (S1 y S2)
> - `show interfaces trunk` (S1)
> - El ping fallido (para mostrar que entendiste el comportamiento esperado)

---

##  Siguiente paso

Ahora que entiendo VLANs y trunks, el siguiente reto es **Inter-VLAN Routing** usando un router o un switch de capa 3. Ahí es donde PC-A y PC-B podrán comunicarse entre sí.

---

**¿Te gusta más esta versión?** Es mucho más narrativa, explica el *porqué* de las cosas y evita el formato de "lista de comandos". Si quieres, puedo ajustarla aún más (por ejemplo, si prefieres que hable en primera persona o si quieres agregar más detalles de tus errores).

