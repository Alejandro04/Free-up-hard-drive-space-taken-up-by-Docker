# Free Disk Space Used by Docker on Windows 11 Home

## The problem

Docker Desktop on Windows uses WSL2, which stores all its data in a `.vhdx` file. This file **grows automatically** but **never shrinks on its own**, even after deleting containers, images, or volumes. The result: your physical disk fills up even though Docker "has nothing in it".

## The solution

4 steps in order:

1. **Clean Docker** — remove unused containers, images, and volumes with `docker system prune`.
2. **Shut down WSL and Docker** — the VHDX must be completely idle before it can be compacted.
3. **Compact the VHDX with diskpart** — reclaims actual physical disk space (`Optimize-VHD` does not work on Windows Home, only `diskpart` does).
4. **Restart Docker Desktop**.

See the exact commands in [`SKILLS.md`](./SKILLS.md).
