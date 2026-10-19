# Hardware & Storage

## Specifications
| Component | Specification |
|-----------|---------------|
| Platform | Raspberry Pi 4 Model B Rev 1.5 |
| CPU | Broadcom BCM2711 (4×1.8 GHz) |
| RAM | 4 GB LPDDR4 |
| Network | Gigabit Ethernet |
| OS | Debian GNU/Linux 13 (Trixie) |

## Storage Layout
Both storage devices are connected to the Pi's USB 3.0 ports to avoid throughput bottlenecks.

- **System (`/dev/sda`):** 128 GB NVMe SSD. Hosts Debian, OpenMediaVault, Docker Engine, and OMV's overlay write-cache for system directories (protecting the SSD from excessive writes).
- **Data (`/dev/sdb`):** 1 TB External HDD formatted as ext4. Mounted at `/srv/dev-disk-by-uuid-[UUID]`.
- **MergerFS:** Unifies the data disk at `/srv/mergerfs/storage`. With a single disk today, it provides no pooling benefit but future-proofs the setup for adding more disks without changing user paths.

## Power Management
To balance performance and disk longevity, the data HDD is configured with:
- **Advanced Power Management (APM):** 127
- **Spindown timeout:** 20 minutes

This reduces unnecessary wear while avoiding excessive spin-up/spin-down cycles during idle periods.