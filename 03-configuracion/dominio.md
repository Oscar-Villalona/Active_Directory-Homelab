# Configuración del dominio astra.lab

## 1. IP estática

```powershell
Get-NetAdapter
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.10.10 -PrefixLength 24
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 127.0.0.1
Get-NetIPConfiguration
```

- Sin gateway: la red es aislada.
- El DNS apunta a sí mismo porque DC01 será el servidor DNS del dominio.

### Hallazgo: ping sin respuesta hacia el host

El ping a 192.168.10.1 (adaptador VMnet2 del host) resultó en 100 % de pérdida.
- **Causa probable:** el firewall del host bloquea ICMP en la nueva red, que trata como pública.
- **Impacto:** ninguno para el dominio. Se probará la conectividad por otras vías cuando haya clientes.

## 2. Instalación del rol AD DS

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

Resultado: `Success: True`, sin reinicio necesario.

## 3. Promoción a Domain Controller

```powershell
Install-ADDSForest -DomainName "astra.lab" -DomainNetbiosName "LAB" -InstallDns
```

- Se solicitó la contraseña DSRM (*Safe Mode Administrator Password*). Es **distinta** de la de Administrator y se guarda en el gestor de contraseñas.
- El servidor se reinició solo; el inicio de sesión posterior es como `LAB\Administrator`.

### Advertencias durante la promoción (esperables)

| Advertencia | Explicación |
|---|---|
| Algoritmos de criptografía compatibles con Windows NT 4.0 | Windows Server 2022 los bloquea por defecto en sesiones de canal seguro. Es una mejora de seguridad, no un problema. |
| No se puede crear la delegación DNS | `astra.lab` no tiene zona padre. En una red aislada no hay que hacer nada. |

Las advertencias aparecen duplicadas porque la validación se imprime dos veces.

Evidencia: `capturas/08-promocion-dc.png`.

## 4. Verificación

```powershell
Get-ADDomain
Get-Service NTDS, DNS, Netlogon
dcdiag /q
net share
```

### Resultados

| Comprobación | Resultado |
|---|---|
| `Get-ADDomain` | `DNSRoot: astra.lab`, `NetBIOSName: LAB`, `DomainMode: Windows2016Domain` |
| Roles FSMO | PDC Emulator, RID Master e Infrastructure Master en `DC01.astra.lab` (normal con un único DC) |
| Servicios | DNS, Netlogon y NTDS en estado `Running` |
| `dcdiag /q` | Dos pruebas fallidas: `DFSREvent` y `SystemLog` (ver abajo) |

Evidencia: `capturas/09-get-addomain.png`, `capturas/10-dcdiag.png`.

### Hallazgo: fallos de dcdiag (DFSREvent y SystemLog)

Ambas pruebas revisan el registro de eventos de las últimas 24 horas y encontraron dos errores generados durante la promoción:

| Evento | Descripción | Evaluación |
|---|---|---|
| `0xC000186F` | El servicio `netprofm` (Network List Service) terminó con error | Frecuente en el primer reinicio de un DC recién promovido |
| `0xC0001B76` | Printer Extensions and Notifications está marcado como servicio interactivo y el sistema no permite servicios interactivos | Ruido conocido, sin relación con AD |

**Cómo se verifica que SYSVOL funciona:** `net share` debe listar los recursos `SYSVOL` y `NETLOGON`. Los dos fallos desaparecen de `dcdiag` pasadas 24 horas, cuando los eventos salen de la ventana de análisis.

> Punto de método: antes de concluir que una alerta es un problema real o un falso positivo, se revisa el evento concreto, su hora y si hay síntomas funcionales que lo respalden.

## 5. Snapshot

`02-dominio-astra-lab`: DC01 promovido, servicios verificados, sin objetos de laboratorio creados todavía.

## 6. Siguiente paso: estructura de OUs, grupos y usuarios

Diseño previsto:

```
astra.lab
└── AstraLab
    ├── Usuarios  (IT, Ventas, RRHH, Finanzas)
    ├── Equipos
    ├── Grupos    (GG-IT, GG-Ventas, GG-RRHH, GG-Finanzas)
    └── Servidores
```

Principios: una OU raíz propia para separar mis objetos de los contenedores por defecto, OUs por tipo de objeto y departamento, y permisos asignados a **grupos**, no a usuarios individuales.
