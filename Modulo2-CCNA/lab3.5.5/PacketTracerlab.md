# 📓 Bitácora de Aprendizaje
## Laboratorio: Configurar DTP (3.5.5)
---

##  ¿Qué estaba intentando lograr?

Configurar troncales (trunks) entre 3 switches usando DTP, asignar VLANs y lograr conectividad entre PCs en diferentes switches.

---

##  Mi dificultad principal

### El problema con el puerto G0/2 (S1 ↔ S3)

Cuando llegué a la Parte 4, configuré S1 así:

```bash
S1(config)# interface g0/2
S1(config-if)# switchport mode trunk
S1(config-if)# switchport nonegotiate
