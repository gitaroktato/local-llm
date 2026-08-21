# Hardware BOM

Bill of materials for the local LLM build machine (identified in issue #8).

## Bill of Materials

| Component | Part / Model | Qty | Notes | Reference |
|-----------|--------------|-----|-------|-----------|
| CPU | Intel Core i5-14400F | 1 | 10C/16T, up to 4.7 GHz | [Intel ARK](https://ark.intel.com/content/www/us/en/ark/products/236181/intel-core-i5-1440f-processor-20m-cache.html) |
| Motherboard | ASUS ROG MAXIMUS Z690 HERO | 1 | Intel Z690, ATX | [ASUS product page](https://rog.asus.com/us/motherboards/rog-maximus/rog-maximus-z690-hero-model/spec/) |
| GPU | NVIDIA GeForce RTX 5060 Ti 16GB (GB206) | 2 | Driver 610.57.04 | [NVIDIA](https://www.nvidia.com/en-us/geforce/graphics-cards/50-series/rtx-5060-ti/) · [TechPowerUp specs](https://www.techpowerup.com/gpu-specs/rtx-5060-ti-16gb.c4273) |
| RAM | Kingston FURY Beast KF552C36BBEK2-32 (2×16GB DDR5-5200 CL36) | 1 kit (2 DIMMs) | ~32GB total | [Kingston datasheet](https://www.kingston.com/datasheets/KF552C36BWEAK2-32.pdf) |
| PSU | ASUS ROG STRIX 1000G (1000W) | 1 | | [ASUS product page](https://rog.asus.com/power-supply-units/rog-strix/rog-strix-1000g-model/) |
| Case | Corsair 5000T Black (CC-9011300-ww) | 1 | Mid tower, high performance; see `img/case-change.jpg` | [Corsair product page](https://www.corsair.com/us/en/p/pc-cases/cc-9011300-ww/5000t-mid-tower-high-performance-pc-case-cc-9011300-ww) |
| OS | Arch Linux | — | See [arch-linux-unsloth.md](./arch-linux-unsloth.md) | |

## PCIe Riser Cables

Both GPUs are mounted off-slot via ADT-LINK (Ares) PCIe 5.0 riser cables:

| Riser | Length | Notes |
|-------|--------|-------|
| ADT-LINK K33UR-TL | 30 cm | PCIe 5.0 x16 → x16, full speed (512 GB/s max) |
| ADT-LINK K33UF-TR | 60 cm | PCIe 5.0 x16 → x16, full speed (512 GB/s max) |

Reference: [ADT-LINK K33Ux-T product page](https://www.adtlink.com/product/K33Ux-T.html) · [adt.link x16 overview](https://www.adt.link/x16.html)

## PCIe Topology

- **GPU0** — Gen5 x16, wired to the CPU root port (`00:01.0`). Full width from the socket.
- **GPU1** — Gen5 **x1**, via the riser path on root port `00:01.1`. Bandwidth is capped at x1 (~4 GB/s); NVLink-style P2P across the pair is not possible, so multi-GPU inference should rely on tensor/pipe parallelism over PCIe or fit the model on one GPU when practical.

## 3D-Printed Parts

Custom parts for mounting the second GPU and holding cabling; CAD sources (FreeCAD) included:

| Part | STL | Source | Preview |
|------|-----|--------|---------|
| GPU holder base | `designs/GPU-holder-base.stl` | `designs/GPU-holder.FCStd` | [img/gpu-holder-screw.png](img/gpu-holder-screw.png) |
| GPU holder screw body | `designs/GPU-holder-screw-body.stl` | `designs/GPU-holder.FCStd` | [img/gpu-screw.jpg](img/gpu-screw.jpg) |
| GPU mount | `designs/GPU_mount-gpumount.stl` | `designs/GPU_mount.FCStd` | [img/gpu-mount-design.png](img/gpu-mount-design.png) |
| GPU mount lock | `designs/GPU_mount-lock.stl` | `designs/GPU_mount.FCStd` | [img/gpu-mount.jpg](img/gpu-mount.jpg) |

## Photos

| Photo | Description |
|-------|-------------|
| [final-machine.jpg](img/final-machine.jpg) | Finished build |
| [dual-gpu-no-thermal.jpg](img/dual-gpu-no-thermal.jpg) | Dual GPU before thermal paste |
| [gpu-mount-vertical.jpg](img/gpu-mount-vertical.jpg) | Vertical GPU mount |
| [motherboard-part.jpg](img/motherboard-part.jpg) | Motherboard / riser wiring |
| [pcie5-flex.jpg](img/pcie5-flex.jpg) · [pcie-flex2.jpg](img/pcie-flex2.jpg) | PCIe 5.0 flex risers |
| [gpu-mount-short-1.jpg](img/gpu-mount-short-1.jpg) · [gpu-mount-short-2.jpg](img/gpu-mount-short-2.jpg) · [gpu-mount-screw.jpg](img/gpu-mount-screw.jpg) | Mount assembly variants |
| [hardware-offline.jpg](img/hardware-offline.jpg) · [hardware-offline-2.jpg](img/hardware-offline-2.jpg) | Offline hardware shots |
