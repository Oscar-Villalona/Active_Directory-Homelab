# ISOs y verificación de integridad

Todas las imágenes provienen del **Microsoft Evaluation Center** (licencia de evaluación de 180 días). Fecha de descarga: 2026-10-12.

## Cómo se verifica

```powershell
Get-FileHash "C:\ISOs\nombre-del-archivo.iso" -Algorithm SHA256
```

Para comparar contra un hash publicado:

```powershell
$calculado = (Get-FileHash "C:\ISOs\archivo.iso" -Algorithm SHA256).Hash
$publicado = "<hash publicado por Microsoft>"
$calculado -eq $publicado   # debe devolver True
```

## Registro

| ISO | SHA-256 | Hash oficial | ¿Coincide? |
|---|---|---|---|
| Windows Server 2022 Evaluation | `3E4FA6D8507B554856FC9CA6079CC402DF11A8B79344871669F0251535255325` | No publicado en la página de descarga | N/A: queda como línea base propia |
| Windows 11 Enterprise Evaluation 26H2 (x64, EN-US) | Pendiente de anotar el hash calculado localmente | `BC3F24086EBADC94489066B5AD78089E2CF5C3491E90E790BB81A2B199C10E38` | Pendiente de confirmar con el script de arriba |

> Nota: el hash de Windows Server es el que figura en mis notas; el de Windows 11 es el publicado por Microsoft en su página de evaluación. Antes de publicar, ejecutar la comparación en PowerShell y actualizar la última columna con el resultado real.

Evidencia: `capturas/03-hashes-iso.png`.
