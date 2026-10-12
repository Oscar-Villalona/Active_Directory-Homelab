# Bitácora

## 2026-10-12: Construcción del dominio astra.lab

### Planificación
- **Objetivo:** definir red, inventario y estructura de documentación.
- **Qué hice:** diseñé la red 192.168.10.0/24 aislada y el inventario de VMs. Detalle en `01-planificacion/red.md`.

### Hipervisor y red virtual
- **Qué hice:** instalé VMware Workstation y creé la red Host-only LAB (VMnet2) sin DHCP de VMware.
- **Evidencia:** `capturas/02-virtual-network-editor.png`.

### Descarga y verificación de ISOs
- **Qué hice:** descargué Windows Server 2022 y Windows 11 Enterprise (evaluación) y registré sus hashes SHA-256.
- **Observación:** Microsoft publicó el hash de Windows 11 pero no el de Windows Server; este último queda como línea base propia.

### Creación de la VM DC01
- **Problema 1:** el asistente asigna la red VMnet1 por defecto. **Solución:** cambiar a VMnet2 en *Customize Hardware*.
- **Problema 2:** disco preasignado, 4 cores y tarjeta de sonido por defecto. **Solución:** disco dinámico en un archivo, 2 vCPU y sin sonido.
- **Decisión:** controlador LSI Logic SAS (Paravirtualized SCSI exigiría cargar drivers en la instalación).

### Primer arranque
- **Problema:** `EFI Network... Time out`; no arrancaba el instalador.
- **Causa:** aviso "Press any key" perdido / ISO no conectada al encender (UEFI).
- **Solución:** verificar *Connect at power on* y la ISO, y entrar al firmware para priorizar el CD-ROM.

### Instalación de Windows Server 2022
- **Decisión:** Standard Evaluation (Desktop Experience); descartados Server Core y Datacenter.
- **Contraseñas:** generadas en un gestor de contraseñas. Aprendizaje: una contraseña pegada en un chat o nota deja de ser secreta, por lo que se generó una nueva.
- **VMware Tools:** se instalaron montando `windows.iso` de VMware cuando la unidad seguía mostrando la ISO del instalador.
- **Resultado:** servidor renombrado a `DC01`. Snapshot `01-instalacion-limpia`.

### IP estática y promoción a DC
- **Qué hice:** IP 192.168.10.10/24, instalé AD DS y promoví el servidor con `Install-ADDSForest` creando el dominio `astra.lab`.
- **Hallazgo:** ping a 192.168.10.1 con 100 % de pérdida; causa probable, firewall del host bloqueando ICMP en VMnet2.
- **Advertencias de la promoción:** criptografía NT 4.0 y delegación DNS; ambas esperables.

### Verificación
- `Get-ADDomain` correcto; DNS, Netlogon y NTDS en ejecución.
- `dcdiag /q`: fallos en `DFSREvent` y `SystemLog` por eventos de arranque (`netprofm` y Printer Extensions). Se contrastó con `net share` (SYSVOL y NETLOGON).
- **Resultado:** snapshot `02-dominio-astra-lab`.

### Pendiente
- Crear OUs, grupos y usuarios de prueba.
- Carga masiva de usuarios desde CSV con PowerShell.
- Crear WS01 y unirlo al dominio.
