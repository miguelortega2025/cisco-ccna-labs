#  Lab: NAT Dinámico en Cisco Packet Tracer

##  ¿Qué hice en este lab?

Configuré **NAT dinámico** en el router R2 para que los dispositivos de la red interna (L1, PC1 y PC2) pudieran salir a Internet usando un pool pequeño de direcciones públicas.

La topología era así:

- Red interna: subredes 172.16.10.0/24, 172.16.11.0/24 y el enlace 172.16.1.0/30 entre R1 y R2.
- Router NAT: R2.
- Red externa: 209.165.200.224/27 (hacia Internet).
- Servidor: 209.165.201.5 (Server1).

La idea era que los dispositivos internos salieran a Internet con una IP pública del pool, sin exponer sus IPs privadas.

---

##  ¿Qué configuré?

### 1. ACL para seleccionar el tráfico a traducir

```bash
access-list 1 permit 172.16.0.0 0.0.255.255
```

Aunque en la topología no aparece escrita la red 172.16.0.0/16, todas las subredes internas (172.16.10.0/24, 172.16.11.0/24, 172.16.1.0/30) pertenecen a ese bloque. Por eso usé la wildcard `0.0.255.255`, que cubre todo ese rango.

### 2. Pool NAT con dos direcciones públicas

```bash
ip nat pool POOL-NAT 209.165.200.229 209.165.200.230 netmask 255.255.255.252
```

El bloque 209.165.200.228/30 tiene 4 direcciones:

- .228 → red
- .229 → usable
- .230 → usable
- .231 → broadcast

Usé las dos usables.

### 3. Asociar la ACL con el pool

```bash
ip nat inside source list 1 pool POOL-NAT
```

Este comando es el que realmente activa la traducción: le dice al router que use la ACL 1 para decidir **qué** traducir y el pool POOL-NAT para decidir **a qué** traducir.

### 4. Interfaces inside y outside

```bash
interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat outside
```

Aquí marqué la interfaz que va hacia R1 como `inside` (lado privado) y la que va hacia Internet como `outside` (lado público).

---

##  Lo que aprendí

### La diferencia entre ACL y NAT

Al principio me confundía mucho esto. Pensaba que `inside` y `outside` de NAT tenían algo que ver con `in` y `out` de las ACLs. Pero no:

- **ACL con `in`/`out`** → filtra paquetes en una interfaz (permit/deny).
- **NAT con `inside`/`outside`** → etiqueta la interfaz como lado privado o público.

Son cosas independientes. La ACL dentro del comando `ip nat inside source list 1 pool POOL-NAT` **no filtra**, solo **selecciona** qué tráfico se traduce.

### NAT dinámico = 4 piezas

| Pieza | Función |
|---|---|
| ACL | Qué se traduce |
| Pool | A qué se traduce |
| `ip nat inside source` | Une ACL + pool |
| `ip nat inside/outside` | Roles de las interfaces |

### Limitación del pool

El pool tiene solo 2 direcciones y hay 3 dispositivos internos. Eso significa que **solo 2 pueden navegar al mismo tiempo**. El tercero tendrá que esperar a que una traducción se libere.

Esto me hizo pensar en el mundo real: si una empresa tiene 100 empleados y solo 10 IPs públicas, NAT dinámico no alcanza. Ahí es donde entra **PAT (NAT overload)**, que permite que muchos dispositivos compartan una sola IP pública usando puertos diferentes.

---

##  Verificación

### Ver traducciones activas

```bash
show ip nat translations
```

Aquí se ve la IP privada del dispositivo (inside local) y la IP pública que se le asignó (inside global).

### Probar acceso web

Desde L1, PC1 o PC2 abrí el navegador y accedí a `209.165.201.5`. La página del Server1 cargó correctamente.

### Probar el límite del pool

Abrí el navegador en PC1 y PC2 al mismo tiempo. Luego intenté desde L1 y no cargó, porque las 2 direcciones del pool estaban ocupadas. Eso confirma la limitación.

---

##  ¿Qué problema de negocio resuelve esto?

NAT dinámico le permite a una empresa:

- Usar **IPs privadas** en su red interna (gratis, sin agotamiento).
- Salir a Internet con un **pool pequeño de IPs públicas** (caras y escasas).
- **Ahorrar dinero** al no comprar una IP pública por cada dispositivo.
- **Ocultar** las IPs internas de Internet (seguridad por oscurecimiento).
- **Escalar** agregando dispositivos sin pedir más IPs al ISP.

Pero tiene límites:

- Si el pool es más pequeño que la demanda, hay cuellos de botella.
- No hay multiplexación de puertos (eso es PAT).
- Sin logs, no hay trazabilidad de quién usó qué IP.

Por eso, en empresas grandes, **PAT es la evolución natural** de NAT dinámico.

---

##  Conclusión

Este lab me ayudó a entender que NAT dinámico no es solo un tema técnico, sino una **decisión de negocio**: cómo conectar muchos dispositivos a Internet con pocos recursos públicos. Aprendí a configurarlo, a verificar que funcione y a reconocer sus límites.

Y sobre todo, entendí la diferencia entre **filtrar** (ACL) y **traducir** (NAT), que era mi mayor confusión al principio.
