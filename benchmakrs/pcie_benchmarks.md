# PCIe Benchmarks

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
