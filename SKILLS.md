# Free Docker Memory and Shrink VHDX

## Step 1 — Clean Docker internal space

Run in terminal (no admin required):

```bash
docker system prune -a --volumes
```

This removes: stopped containers, unused images, orphaned volumes, and build cache.

---

## Step 2 — Force shutdown of WSL and Docker

In PowerShell:

```powershell
wsl --shutdown
taskkill /F /IM "wslhost.exe" 2>$null
taskkill /F /IM "wsl.exe" 2>$null
taskkill /F /IM "Docker Desktop.exe" 2>$null
taskkill /F /IM "com.docker.backend.exe" 2>$null
```

Verify WSL is stopped:

```powershell
wsl -l --running
# Should show: "There are no running distributions."
```

---

## Step 3 — Compact the VHDX file with diskpart

> Requires **PowerShell as Administrator**.  
> `Optimize-VHD` does NOT work on Windows 11 Home — use `diskpart` instead.

First confirm the file path:

```powershell
Get-ChildItem "$env:LOCALAPPDATA\Docker" -Recurse -Filter "*.vhdx" | Select FullName
```

Usual path:
```
C:\Users\alejo\AppData\Local\Docker\wsl\disk\docker_data.vhdx
```

Open diskpart as Administrator and run one by one:

```
diskpart
select vdisk file="C:\Users\alejo\AppData\Local\Docker\wsl\disk\docker_data.vhdx"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

> The process can take several minutes. When done it shows the percentage reduced.

---

## Step 4 — Restart Docker Desktop

Open Docker Desktop normally.

---

## Notes

- If `attach vdisk readonly` fails with "being used by another process": restart Windows and run diskpart **before** opening Docker Desktop.
- `Optimize-VHD` requires the Hyper-V module (only available on Windows Pro/Enterprise).
- The VHDX does not shrink automatically after `docker prune` — step 3 is necessary to reclaim physical disk space.
