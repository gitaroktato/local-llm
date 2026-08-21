# GPU Power Cap ("Undervolt")

On NVIDIA Linux there is no true mV undervolt — the closest equivalent is a **power-limit cap** via `nvidia-smi -pl`. This page documents why, how it was applied on this machine (2× RTX 5060 Ti, driver `610.57.04`), and how to keep it across reboots.

## Why not a real undervolt?

- NVIDIA has never exposed a voltage-offset API in NVML on Linux. Probing `libnvidia-ml.so` confirms: no voltage-related symbols exist.
- mV undervolting (MSI Afterburner / Rivatuner) is **Windows-only**.
- Clock locking (`nvidia-smi -lgc`) is deprecated for this Blackwell driver generation — P-state query APIs return `NOT_SUPPORTED`, and nvidia-smi marks Applications Clocks as "deprecated". Excluded from scope.

The power cap is a soft PMU constraint: fully reversible, zero hardware risk. For LLM inference (often memory-bandwidth-bound and rarely at TDP), dropping the limit typically costs only a few % of throughput while cutting heat, noise and wall power.

## Applying the cap

Allowed range per card on the RTX 5060 Ti: **150–180 W** (stock `180 W`).

```bash
# check current limits
nvidia-smi --query-gpu=index,power.limit,power.default_limit,power.min_limit,power.max_limit --format=csv

# apply 150 W to all GPUs (applies live, no restart needed; root required)
sudo nvidia-smi -pl 150

# verify
nvidia-smi --query-gpu=index,power.limit --format=csv
```

Note: plain `-pl <watts>` already targets all GPUs. Do not use `--gpu-all` (not recognized) or `-i ALL` ("No devices found") on driver `610.x` — both fail. Setting the limit requires root; querying does not.

This machine runs at **150 W** on both GPUs. The cap was chosen directly (no 180/165/150 W perf-loss table was recorded) — inference throughput was acceptable and heat/noise reduction was the goal.

## Persistence across reboots

The cap does not survive a driver reload or reboot by itself. Two pieces:

### 1. `nvidia-persistenced`

Keeps the driver loaded (and the daemon context alive) between sessions:

```bash
sudo systemctl enable --now nvidia-persistenced
```

### 2. Oneshot unit applying the cap at boot

`/etc/systemd/system/gpu-power-cap.service`:

```ini
[Unit]
Description=Apply 150 W power cap to all NVIDIA GPUs
After=sysinit.target nvidia-persistenced.service
ConditionPathExists=/usr/bin/nvidia-smi

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/sh -c 'for i in $(seq 1 30); do nvidia-smi -pl 150 >/dev/null && exit 0; sleep 2; done; echo "gpu-power-cap: failed to apply power limit after retries" >&2; exit 1'

[Install]
WantedBy=multi-user.target
```

The retry loop (30 × 2 s) covers the case where the nvidia module is not up yet when the unit runs. Install and enable:

```bash
sudo cp gpu-power-cap.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now gpu-power-cap.service
```

### Verify after reboot

```bash
nvidia-smi --query-gpu=index,power.limit --format=csv   # expect 150.00 W on both
systemctl status gpu-power-cap.service nvidia-persistenced.service
```

## Revert

```bash
sudo nvidia-smi -pl 180              # restore stock limit on all GPUs (live)
sudo systemctl disable --now gpu-power-cap.service   # optional: stop forcing the cap at boot
```

No clock locks, no register hacks — everything is reversible.
