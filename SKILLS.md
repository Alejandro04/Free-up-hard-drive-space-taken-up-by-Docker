# Liberar memoria Docker y reducir VHDX

## Paso 1 — Limpiar espacio interno de Docker

Ejecutar en terminal (no requiere admin):

```bash
docker system prune -a --volumes
```

Esto elimina: contenedores detenidos, imágenes sin usar, volúmenes huérfanos y caché de build.

---

## Paso 2 — Forzar cierre de WSL y Docker

En PowerShell:

```powershell
wsl --shutdown
taskkill /F /IM "wslhost.exe" 2>$null
taskkill /F /IM "wsl.exe" 2>$null
taskkill /F /IM "Docker Desktop.exe" 2>$null
taskkill /F /IM "com.docker.backend.exe" 2>$null
```

Verificar que WSL está detenido:

```powershell
wsl -l --running
# Debe mostrar: "There are no running distributions."
```

---

## Paso 3 — Compactar el archivo VHDX con diskpart

> Requiere **PowerShell como Administrador**.  
> `Optimize-VHD` NO funciona en Windows 11 Home — usar `diskpart` en su lugar.

Primero confirmar la ruta del archivo:

```powershell
Get-ChildItem "$env:LOCALAPPDATA\Docker" -Recurse -Filter "*.vhdx" | Select FullName
```

Ruta habitual:
```
C:\Users\alejo\AppData\Local\Docker\wsl\disk\docker_data.vhdx
```

Abrir diskpart como Administrador y ejecutar uno por uno:

```
diskpart
select vdisk file="C:\Users\alejo\AppData\Local\Docker\wsl\disk\docker_data.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

> El proceso puede tardar varios minutos. Al terminar muestra el porcentaje reducido.

---

## Paso 4 — Reiniciar Docker Desktop

Abrir Docker Desktop normalmente.

---

## Notas

- Si `attach vdisk readonly` falla con "being used by another process": reiniciar Windows y ejecutar diskpart **antes** de abrir Docker Desktop.
- `Optimize-VHD` requiere el módulo Hyper-V (solo disponible en Windows Pro/Enterprise).
- El VHDX no se achica automáticamente tras `docker prune` — el paso 3 es necesario para recuperar espacio físico en disco.
