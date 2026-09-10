# 🔐 Laboratorio: ACL Estándar con Nombre para IPv4
## Enfoque: Resolviendo Problemas de Negocio Reales

---

## 📋 Descripción del Laboratorio

Este laboratorio implementa una **ACL (Access Control List) estándar con nombre** en un router Cisco para proteger un **Servidor de Archivos** crítico. Aunque técnicamente es un ejercicio de configuración de red, el objetivo real es resolver **problemas de negocio** relacionados con seguridad, cumplimiento y control de acceso.

---

## 🎯 Problema de Negocio

> **"Necesitamos proteger la información confidencial de la empresa, pero sin bloquear el acceso a los recursos que los empleados necesitan para trabajar."**

Este es el equilibrio fundamental entre **seguridad** y **productividad** que toda empresa debe resolver.

---

## 🏢 Escenario Empresarial

| Dispositivo | Rol en el Negocio | Dirección IP |
|-------------|-------------------|--------------|
| **Servidor de Archivos** | Base de datos crítica (información confidencial) | 192.168.200.100 |
| **Servidor Web** | Portal de aplicaciones web (necesita acceso a la BD) | 192.168.100.100 |
| **PC1** | Estación de trabajo autorizada (ej. Gerente de TI) | 192.168.20.4 |
| **PC0** | Estación de trabajo no autorizada (ej. Recepción) | 192.168.10.3 |
| **PC2** | Estación de trabajo no autorizada (ej. Ventas) | 192.168.10.4 |

**Regla de negocio:**
- ✅ **PC1** y el **Servidor Web** pueden acceder al Servidor de Archivos.
- ❌ **PC0** y **PC2** NO pueden acceder al Servidor de Archivos.
- ✅ Todos pueden acceder al Servidor Web.

---

## 🛠️ Solución Técnica

### Topología
<img width="726" height="437" alt="image" src="https://github.com/user-attachments/assets/1eace251-2055-4d82-9ee2-8c24a63572aa" />


### Configuración de la ACL

```cisco
R1> enable
R1# configure terminal

! Crear la ACL estándar con nombre
R1(config)# ip access-list standard File_Server_Restrictions

! Reglas en orden específico (importante)
R1(config-std-nacl)# permit host 192.168.20.4
R1(config-std-nacl)# permit host 192.168.100.100
R1(config-std-nacl)# deny any

! Verificar antes de aplicar
R1(config-std-nacl)# end
R1# show access-lists

! Aplicar a la interfaz que conecta al Servidor de Archivos
R1# configure terminal
R1(config)# interface fastethernet 0/1
R1(config-if)# ip access-group File_Server_Restrictions out
R1(config-if)# end

! Guardar configuración
R1# copy running-config startup-config 

### Configuración de la ACL

R1# show access-lists

Standard IP access list File_Server_Restrictions
    10 permit host 192.168.20.4 (4 match(es))
    20 permit host 192.168.100.100 (4 match(es))
    30 deny any (8 match(es))

