# PCIe Benchmarks

## Stress test: GPU 1 via riser, 2026-10-09 (issue #36)

GPU 1 (`00000000:02:00.0`) under sustained bidirectional memcpy load, with link state polled every 2 s during the run.

```bash
CUDA_VISIBLE_DEVICES=1 ~/bin/nvbandwidth -p host_to_device_bidirectional -b 128 -i 100
CUDA_VISIBLE_DEVICES=1 ~/bin/nvbandwidth -p device_to_host_bidirectional -b 128 -i 100

# link state poller (parallel)
watch -n2 'nvidia-smi -i 1 --query-gpu=pcie.link.gen.current,pcie.link.gen.max,pcie.link.width.current,pcie.link.width.max --format=csv'
```

Results:

| Test | CE | SM |
|---|---|---|
| host→device bidirectional | 17.88 GB/s | 22.20 GB/s |
| device→host bidirectional | 26.73 GB/s | 22.24 GB/s |

- Link state: held `Gen 5 x8` (max) throughout the entire run — no downshifts under load.
- AER check after stress (`sudo dmesg | grep -iE "pcie|aer"`): no errors.
- Bandwidth matches the single-GPU riser results below (~18 / ~22 / ~27 GB/s).

## Diagnostics

```bash
sudo lspci -vvv | grep -E "LnkCap:|LnkSta:" | grep 'Port #0'
```

LnkCap (Capability): Should show `Speed 32GT/s` (which denotes PCIe 5.0).

LnkSta (Current Status): Should also show `Speed 32GT/s`. If `LnkSta` shows `16GT/s` (PCIe 4.0) or `8GT/s` (PCIe 3.0) while the GPU is under load, the riser cable is degrading the signal.

```bash
sudo dmesg | grep -i -E "pcie|aer"
```

What to look for: If you see a stream of `PCIe Bus Error: severity=Corrected` or `Uncorrected` errors pointing to your GPU's PCI address, the riser cable is failing to provide adequate signal integrity for PCIe 5.0.

## Kolink 300mm PGW-RC-MRK-011 

```bash
nvbandwidth -p host_to_device_bidirectional -i 128
```

```text
nvbandwidth Version: v0.9
Built from Git version: v0.9

CUDA Runtime Version: 13030
CUDA Driver Version: 13030
Driver Version: 610.43.02

archlinux
Device 0: NVIDIA GeForce RTX 5060 Ti (00000000:01:00)

Running host_to_device_bidirectional_memcpy_ce.
memcpy CE CPU(row) <-> GPU(column) bandwidth (GB/s)
           0
 0     18.00

SUM host_to_device_bidirectional_memcpy_ce 18.00

Running host_to_device_bidirectional_memcpy_sm.
memcpy SM CPU(row) <-> GPU(column) bandwidth (GB/s)
           0
 0     21.99

SUM host_to_device_bidirectional_memcpy_sm 21.99

NOTE: The reported results may not reflect the full capabilities of the platform.
Performance can vary with software drivers, hardware clocks, and system topology.
```

```bash
nvbandwidth -p device_to_host_bidirectional -i 128
```

```text
nvbandwidth Version: v0.9
Built from Git version: v0.9

CUDA Runtime Version: 13030
CUDA Driver Version: 13030
Driver Version: 610.43.02

archlinux
Device 0: NVIDIA GeForce RTX 5060 Ti (00000000:01:00)

Running device_to_host_bidirectional_memcpy_ce.
memcpy CE CPU(row) <-> GPU(column) bandwidth (GB/s)
           0
 0     26.78

SUM device_to_host_bidirectional_memcpy_ce 26.78

Running device_to_host_bidirectional_memcpy_sm.
memcpy SM CPU(row) <-> GPU(column) bandwidth (GB/s)
           0
 0     22.02

SUM device_to_host_bidirectional_memcpy_sm 22.02

NOTE: The reported results may not reflect the full capabilities of the platform.
Performance can vary with software drivers, hardware clocks, and system topology.
```


## ADT-LINK K33UR-TL 30 cm

```bash
nvbandwidth -p host_to_device_bidirectional -i 128
```

```text
nvbandwidth Version: v0.9
Built from Git version: v0.9

CUDA Runtime Version: 13030
CUDA Driver Version: 13030
Driver Version: 610.43.02

archlinux
Device 0: NVIDIA GeForce RTX 5060 Ti (00000000:01:00)

Running host_to_device_bidirectional_memcpy_ce.
memcpy CE CPU(row) <-> GPU(column) bandwidth (GB/s)
           0
 0     18.04

SUM host_to_device_bidirectional_memcpy_ce 18.04

Running host_to_device_bidirectional_memcpy_sm.
memcpy SM CPU(row) <-> GPU(column) bandwidth (GB/s)
           0
 0     22.43

SUM host_to_device_bidirectional_memcpy_sm 22.43

NOTE: The reported results may not reflect the full capabilities of the platform.
Performance can vary with software drivers, hardware clocks, and system topology.
```


```bash
nvbandwidth -p device_to_host_bidirectional -i 128
```

```text
nvbandwidth Version: v0.9
Built from Git version: v0.9

CUDA Runtime Version: 13030
CUDA Driver Version: 13030
Driver Version: 610.43.02

archlinux
Device 0: NVIDIA GeForce RTX 5060 Ti (00000000:01:00)

Running device_to_host_bidirectional_memcpy_ce.
memcpy CE CPU(row) <-> GPU(column) bandwidth (GB/s)
0
0     26.84

SUM device_to_host_bidirectional_memcpy_ce 26.84

Running device_to_host_bidirectional_memcpy_sm.
memcpy SM CPU(row) <-> GPU(column) bandwidth (GB/s)
0
0     22.49

SUM device_to_host_bidirectional_memcpy_sm 22.49

NOTE: The reported results may not reflect the full capabilities of the platform.
Performance can vary with software drivers, hardware clocks, and system topology.
```


 
## ADT-LINK K33UR-TL 30 cm and ADT-LINK K33UF-TR 60 cm

```bash
nvbandwidth -p host_to_device_bidirectional -i 128
```

```text
nvbandwidth Version: v0.9
Built from Git version: v0.9

CUDA Runtime Version: 13030
CUDA Driver Version: 13030
Driver Version: 610.43.02

archlinux
Device 0: NVIDIA GeForce RTX 5060 Ti (00000000:01:00)
Device 1: NVIDIA GeForce RTX 5060 Ti (00000000:02:00)

Running host_to_device_bidirectional_memcpy_ce.
memcpy CE CPU(row) <-> GPU(column) bandwidth (GB/s)
           0         1
 0     18.02     17.98

SUM host_to_device_bidirectional_memcpy_ce 36.00
COEFFICIENT_OF_VARIATION host_to_device_bidirectional_memcpy_ce 0.00

Running host_to_device_bidirectional_memcpy_sm.
memcpy SM CPU(row) <-> GPU(column) bandwidth (GB/s)
           0         1
 0     22.37     22.48

SUM host_to_device_bidirectional_memcpy_sm 44.85
COEFFICIENT_OF_VARIATION host_to_device_bidirectional_memcpy_sm 0.00

NOTE: The reported results may not reflect the full capabilities of the platform.
Performance can vary with software drivers, hardware clocks, and system topology.
```

```bash
nvbandwidth -p device_to_host_bidirectional -i 128
```

```text
nvbandwidth Version: v0.9
Built from Git version: v0.9

CUDA Runtime Version: 13030
CUDA Driver Version: 13030
Driver Version: 610.43.02

archlinux
Device 0: NVIDIA GeForce RTX 5060 Ti (00000000:01:00)
Device 1: NVIDIA GeForce RTX 5060 Ti (00000000:02:00)

Running device_to_host_bidirectional_memcpy_ce.
memcpy CE CPU(row) <-> GPU(column) bandwidth (GB/s)
           0         1
 0     26.83     26.83

SUM device_to_host_bidirectional_memcpy_ce 53.67
COEFFICIENT_OF_VARIATION device_to_host_bidirectional_memcpy_ce 0.00

Running device_to_host_bidirectional_memcpy_sm.
memcpy SM CPU(row) <-> GPU(column) bandwidth (GB/s)
           0         1
 0     22.48     22.49

SUM device_to_host_bidirectional_memcpy_sm 44.97
COEFFICIENT_OF_VARIATION device_to_host_bidirectional_memcpy_sm 0.00

NOTE: The reported results may not reflect the full capabilities of the platform.
Performance can vary with software drivers, hardware clocks, and system topology.
```
