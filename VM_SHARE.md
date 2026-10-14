# Windows VM folder share

`excel/` is shared into the `win11-excel` VM (libvirt/KVM) over virtiofs and shows up in Windows as `Z:` (label `riskpack`). Save the workbook there and it lands directly in this repo.

## Host (Fedora)

- The share is in the VM config: a `<filesystem>` with driver `virtiofs`, source `excel/` and target `riskpack`. You need a full VM shutdown and start for config changes to apply. A reboot from inside Windows isn't enough.
- `excel/` has the SELinux type `virt_image_t`. It also has a default ACL for `nor` because `virtiofsd` runs as root. To make the label survive a relabel:
  `sudo semanage fcontext -a -t virt_image_t '/home/nor/Projects/finance_things/riskpack/excel(/.*)?'`
- Docker sets the `FORWARD` policy to DROP, which kills VM internet. Fixed with `{ "ip-forward-no-drop": true }` in `/etc/docker/daemon.json` plus a one-time `sudo iptables -P FORWARD ACCEPT`.

## Guest (Windows 11)

Install WinFsp and `virtio-win-guest-tools.exe` (from the virtio-win ISO), then set **VirtIO-FS Service** to Automatic. `Z:` appears under *This PC*, not under Quick Access.

## References

- libvirt virtiofs guide: https://libvirt.org/kbase/virtiofs.html
- virtio-win virtiofs on Windows: https://github.com/virtio-win/kvm-guest-drivers-windows/wiki/Virtiofs:-Shared-file-system
- WinFsp: https://winfsp.dev/rel/
- Docker and firewall/forwarding: https://docs.docker.com/engine/network/packet-filtering-firewalls/
