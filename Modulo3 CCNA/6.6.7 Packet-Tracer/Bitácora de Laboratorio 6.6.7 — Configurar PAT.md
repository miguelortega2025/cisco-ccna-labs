#  Bitácora de Laboratorio 6.6.7 — Configurar PAT

## Enfoque de Negocio

---

##  1. Contexto del negocio

**Empresa**: "TecnoRed S.A."
**Sector**: Servicios informáticos y conectividad
**Situación**:

- La empresa tiene **dos sucursales** (representadas por R1 y R2).
- Cada sucursal tiene **empleados con PCs y laptops** que necesitan acceder a Internet y a un **servidor web corporativo** (Server1).
- La empresa **no cuenta con muchas direcciones IP públicas**; su ISP solo le ha asignado:
  - Un **rango pequeño** para la sucursal 1 (R1): `209.165.200.232/30`
  - Una **sola IP pública** para la sucursal 2 (R2): la de la interfaz `s0/1/1`

---

##  2. Problema de negocio

La empresa enfrenta **tres problemas críticos**:

###  Problema 1: Escasez de direcciones IPv4 públicas

- Hay **más dispositivos internos que direcciones públicas disponibles**.
- No se puede asignar una IP pública a cada PC, laptop o servidor.

###  Problema 2: Costos de contratar más IPs públicas

- El ISP cobra **por cada IP pública adicional**.
- La empresa quiere **minimizar costos** sin sacrificar conectividad.

###  Problema 3: Necesidad de acceso simultáneo a Internet

- Todos los empleados deben poder navegar, acceder al servidor web y usar aplicaciones en la nube **al mismo tiempo**.
- Sin una solución adecuada, solo unos pocos podrían conectarse.

---

##  3. Solución implementada: PAT (NAT con sobrecarga)

La empresa implementa **PAT** en sus dos routers para **permitir que múltiples dispositivos internos compartan una o pocas IPs públicas**, usando **puertos lógicos** para diferenciar cada conexión.

###  Sucursal 1 (R1) — NAT dinámico con pool + PAT

- Se configura un **pool con 2 IPs públicas** (`209.165.200.233` y `.234`).
- Se usa **PAT con overload** para que todos los dispositivos compartan esas IPs.
- **Beneficio**: mayor capacidad si se agotan los puertos de la primera IP.

###  Sucursal 2 (R2) — PAT mediante interfaz

- Se usa **directamente la IP pública de la interfaz WAN** (`s0/1/1`).
- No se necesita pool.
- **Beneficio**: simplicidad y ahorro, ideal cuando solo hay una IP pública.

---

##  4. Beneficios de negocio

| Beneficio | Impacto en el negocio |
|-----------|------------------------|
| **Ahorro de costos** | No se contratan IPs públicas adicionales |
| **Conectividad simultánea** | Todos los empleados navegan al mismo tiempo |
| **Escalabilidad** | Se pueden agregar más dispositivos sin cambiar la configuración |
| **Seguridad** | Las IPs privadas internas no se exponen directamente a Internet |
| **Continuidad operativa** | El acceso al servidor web corporativo está garantizado |
| **Simplicidad operativa** | La configuración con interfaz es fácil de mantener |

---

##  5. Verificación y resultados

###  En R1 (Sucursal 1)

```bash
R1# show ip nat translations
```

-  Los 4 dispositivos acceden a Server1.
-  Se usa una sola IP del pool.
-  PAT reutiliza puertos para diferenciar conexiones.

###  En R2 (Sucursal 2)

```bash
R2# show ip nat translations
R2# show ip nat statistics
```

- ✅ Los 4 dispositivos acceden a Server1.
- ✅ Se usa la misma IP pública (la de `s0/1/1`).
- ✅ No hay "dynamic mappings" porque no hay pool.

---

##  6. Lecciones aprendidas (enfoque de negocio)

1. **PAT es una solución rentable** para empresas con pocas IPs públicas.
2. **No todas las sucursales necesitan la misma configuración**: unas pueden usar pool, otras interfaz.
3. **La elección depende de los recursos del ISP** y del presupuesto.
4. **La verificación constante** garantiza que el servicio no se interrumpa.

---

##  7. Conclusión ejecutiva

> La implementación de **PAT (NAT con sobrecarga)** permitió a TecnoRed S.A. **conectar a todos sus empleados a Internet y al servidor corporativo** usando **muy pocas direcciones IP públicas**, reduciendo costos, mejorando la escalabilidad y garantizando la continuidad del negocio.

---

##  Anexo: Comandos clave usados

### R1 (Pool NAT + PAT)

```bash
access-list 1 permit 172.16.0.0 0.0.255.255
ip nat pool ANY_POOL_NAME 209.165.200.233 209.165.200.234 netmask 255.255.255.252
ip nat inside source list 1 pool ANY_POOL_NAME overload
interface s0/1/0
 ip nat outside
interface g0/0/0
 ip nat inside
interface g0/0/1
 ip nat inside
```

### R2 (PAT con interfaz)

```bash
access-list 2 permit 172.16.0.0 0.0.255.255
ip nat inside source list 2 interface s0/1/1 overload
interface s0/1/1
 ip nat outside
interface g0/0/0
 ip nat inside
interface g0/0/1
 ip nat inside
```
