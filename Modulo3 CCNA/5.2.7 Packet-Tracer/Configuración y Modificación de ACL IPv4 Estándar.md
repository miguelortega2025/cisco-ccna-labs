# 🔐 Configuración y Modificación de ACL IPv4 Estándar

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-blue?logo=cisco)
![Networking](https://img.shields.io/badge/Networking-ACL-green)
![Security](https://img.shields.io/badge/Security-Filtering-red)
![Status](https://img.shields.io/badge/Status-Completado-brightgreen)

---

## 📑 Tabla de Contenidos

- [Problema Empresarial](#-problema-empresarial-que-resuelve)
- [Objetivos del Laboratorio](#-objetivos-del-laboratorio)
- [Conceptos Clave](#-conceptos-clave-aprendidos)
- [Topología](#-topología)
- [Configuración Aplicada](#️-configuración-aplicada)
- [Verificación](#-verificación)
- [Pruebas Realizadas](#-pruebas-realizadas)
- [Lecciones Aprendidas](#-lecciones-aprendidas)
- [Impacto Empresarial](#-impacto-empresarial)
- [Referencias](#-referencias)

---

## 🏢 Problema Empresarial que Resuelve

En una empresa con **múltiples sucursales** (representadas por R1 y R3), la administración necesita **controlar qué empleados pueden acceder a qué recursos** dentro de la red corporativa. Sin este control:

- ❌ Cualquier usuario podría acceder a **información confidencial** de otras áreas.
- ❌ El tráfico no autorizado podría **saturar enlaces** entre sucursales.
- ❌ No habría **trazabilidad** de quién accede a qué.
- ❌ Se violarían **políticas de seguridad** y normativas como ISO 27001, PCI-DSS o GDPR.

**Solución implementada:** ACL IPv4 estándar para **filtrar tráfico por dirección IP de origen**, permitiendo solo a ciertos hosts o redes acceder a recursos específicos, y denegando explícitamente todo lo demás.

---

## 🎯 Objetivos del Laboratorio

1. ✅ Verificar conectividad antes de aplicar ACL.
2. ✅ Configurar y verificar ACL estándar numeradas y nombradas.
3. ✅ Modificar una ACL estándar sin eliminarla.

---

## 🧠 Conceptos Clave Aprendidos

| Concepto | Descripción |
|----------|-------------|
| **ACL estándar** | Filtra solo por **IP de origen**. Números: 1–99, 1300–1999. |
| **ACL extendida** | Filtra por origen, destino, protocolo y puerto. Números: 100–199, 2000–2699. |
| **Ubicación recomendada** | ACL estándar → **cerca del destino**. ACL extendida → **cerca del origen**. |
| **Deny any implícito** | Toda ACL tiene un `deny any` al final (no visible). |
| **Deny any explícito** | Buena práctica: agregarlo para documentar y contar coincidencias. |
| **IP de origen no cambia** | Los routers no modifican la IP de origen (sin NAT). |
| **Flujo de retorno** | Las ACL también filtran el tráfico que **regresa** hacia la red protegida. |

---

## 🗺️ Topología
