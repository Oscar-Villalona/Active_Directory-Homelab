# Homelab de Active Directory para práctica SOC

Laboratorio virtualizado de Active Directory construido desde cero para practicar administración de Windows Server, telemetría y, más adelante, detección de ataques desde la perspectiva de un analista SOC.

**Autora:** Astra · **Inicio del proyecto:** 2026-10-12 · **Estado:** Fase 1 completada (dominio funcionando)

---

## Objetivo

1. Construir y administrar un dominio de Active Directory real (DC, DNS, OUs, usuarios, grupos, GPOs).
2. Generar y centralizar telemetría de Windows en un SIEM.
3. Simular ataques comunes contra AD (password spraying, Kerberoasting, enumeración) y escribir reglas de detección.
4. Documentar todo el proceso, incluyendo errores y soluciones, como evidencia de práctica.

## Arquitectura actual

| Componente | Detalle |
|---|---|
| Host | PC física con VMware Workstation Pro (versión 25H2) |
| Red del lab | VMnet2 "LAB", Host-only, 192.168.10.0/24, **sin DHCP de VMware** |
| Controlador de dominio | `DC01`, Windows Server 2022 Standard Evaluation (Desktop Experience) |
| IP de DC01 | 192.168.10.10/24, sin gateway, DNS = 127.0.0.1 |
| Dominio / bosque | `astra.lab` (NetBIOS: `LAB`) |
| Nivel funcional | Windows Server 2016 (el máximo disponible) |

```
 ┌──────────────── PC host (Windows) ────────────────┐
 │  VMware Workstation                                │
 │                                                    │
 │   VMnet2 "LAB" (Host-only, 192.168.10.0/24)        │
 │   ├─ Adaptador del host ......... 192.168.10.1     │
 │   └─ DC01 (AD DS + DNS) ......... 192.168.10.10    │
 │                                                    │
 │   (próximamente: WS01 cliente, KALI, SIEM)         │
 └────────────────────────────────────────────────────┘
```

## Hoja de ruta

- [x] **Fase 0:** estructura de documentación y plan de red
- [x] **Fase 1:** hipervisor, red aislada, instalación de DC01, promoción a Domain Controller
- [ ] **Fase 2:** OUs, grupos y usuarios de prueba; carga masiva desde CSV con PowerShell
- [ ] **Fase 3:** cliente Windows unido al dominio (WS01)
- [ ] **Fase 4:** GPOs (política de contraseñas, bloqueo de USB, unidades de red) y hardening
- [ ] **Fase 5:** Sysmon + reenvío de eventos al SIEM (Wazuh / Splunk / Security Onion)
- [ ] **Fase 6:** ataques con Kali y reglas de detección

## Índice de la documentación

| Archivo | Contenido |
|---|---|
| [01-planificacion/red.md](01-planificacion/red.md) | Plan de red, inventario de VMs y decisiones de diseño |
| [02-instalacion/isos.md](02-instalacion/isos.md) | ISOs usadas y verificación de integridad (SHA-256) |
| [02-instalacion/dc01.md](02-instalacion/dc01.md) | Creación de la VM e instalación de Windows Server 2022 |
| [03-configuracion/dominio.md](03-configuracion/dominio.md) | IP estática, promoción a DC, verificación y hallazgos |
| [bitacora.md](bitacora.md) | Diario cronológico con problemas y soluciones |
| [capturas/](capturas/) | Evidencia visual de cada paso |

## Lecciones aprendidas (resumen)

- Verificar el hash de las ISOs antes de instalar.
- Los asistentes de VMware proponen valores por defecto que hay que revisar (red, disco, CPU, hardware innecesario).
- En UEFI, el aviso "Press any key to boot from CD or DVD" dura pocos segundos.
- Las advertencias de `dcdiag` justo después de promover un DC suelen ser ruido de arranque; hay que comprobar con evidencia antes de concluir que algo falla.
- Ninguna credencial se guarda en este repositorio: las contraseñas viven en un gestor de contraseñas.

## Seguridad del repositorio

Este repositorio **no contiene contraseñas, claves ni datos personales**. Los valores sensibles se omiten o se muestran enmascarados. Las ISOs de evaluación de Microsoft no se incluyen; solo sus hashes.
