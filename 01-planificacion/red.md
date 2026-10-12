# Plan de red e inventario

## Red del laboratorio

| Parámetro | Valor |
|---|---|
| Nombre en VMware | LAB (VMnet2) |
| Tipo | Host-only (aislada) |
| Subred | 192.168.10.0/24 |
| DHCP de VMware | **Desactivado** (el DHCP lo dará DC01 más adelante) |
| Adaptador virtual del host | Habilitado (VMware Network Adapter VMnet2, normalmente 192.168.10.1) |
| Dominio | `astra.lab` |
| NetBIOS | `LAB` |

## Inventario de máquinas

| Máquina | Rol | IP | RAM | vCPU | Disco | Estado |
|---|---|---|---|---|---|---|
| DC01 | Domain Controller + DNS | 192.168.10.10 (estática) | 4 GB | 2 | 60 GB | Creada, promovida |
| WS01 | Cliente Windows 10/11 Enterprise Eval | DHCP | 4 GB | 2 | 60 GB | Pendiente |
| KALI | Atacante | 192.168.10.50 | 4 GB | 2 | 40 GB | Pendiente |
| SIEM | Recolección de logs | por definir | 8 GB | 2 | 60 GB | Pendiente |

## Decisiones de diseño y su razón

| Decisión | Razón |
|---|---|
| Red Host-only aislada | Los ataques de práctica no deben salir a mi red real. |
| Mantener activo el adaptador virtual del host | Permite administrar las VMs desde mi PC (RDP, copia de archivos). Contrapartida: mi PC real está en esa red. |
| Desactivar el DHCP de VMware | DC01 será el servidor DHCP del dominio; dos servidores DHCP en la misma red causarían conflictos. |
| Dominio `astra.lab` | Nombre propio y fácil de distinguir. `.lab` evita colisionar con dominios reales. En producción se usaría un subdominio de un dominio real controlado. |
| Windows Server Desktop Experience | El objetivo es aprender con las herramientas gráficas de AD (ADUC, GPMC, Server Manager). Server Core es lo habitual en producción, pero tiene mayor curva de aprendizaje. |
| Edición Standard (no Datacenter) | Las funciones adicionales de Datacenter no se necesitan en este lab. |
| Disco de crecimiento dinámico en un solo archivo | Ahorra espacio en el host con varias VMs. |
| Controlador de disco LSI Logic SAS | Windows Server lo reconoce sin drivers adicionales; Paravirtualized SCSI exigiría cargar drivers durante la instalación. |
| 2 vCPU por VM | Con varias VMs encendidas a la vez, 4 vCPU cada una saturarían el procesador del host. |
