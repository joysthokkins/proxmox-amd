# proxmox-amd
## Asrock x570 itx/tb3 + 5950x + Vega 64 + eGPU Vega 56

## UEFI:
Enable VT-d
Disable CSM
ACS Enable
Enable 4G Decoding
Disable Resizable BAR
Enable IOMMU

## MERGING PROXMOX PARTITIONS
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root

## GPU PASSTHROUGH WITH eGPU
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt pcie_ports=native pci=assign-busses,nocrs,realloc"
update-grub

## FIX WEBUI (When PCIe ADDed):
ip a

nano /etc/network/interfaces
replace the iface and bridge-ports line

## ADD MODULES
nano /etc/modules
vendor-reset
vfio
vfio_iommu_type1
vfio_pci

## VFIO
nano /etc/modprobe.d/vfio-pci.conf
softdep amdgpu pre: vfio-pci
softdep radeon pre: vfio-pci
softdep snd_hda_intel pre: vfio-pci
softdep xhci_pci pre: vfio-pci
options vfio-pci ids=1002:687f disable_vga=1

## KVM
nano /etc/modprobe.d/kvm.conf
options kvm ignore_msrs=1 report_ignored_msrs=0

#BLACKLISTING
#nano /etc/modprobe.d/pve-blacklist.conf

update-initramfs -u -k all

VENDOR-RESET INSTALL
apt install pve-headers
apt install git dkms build-essential
git clone https://github.com/gnif/vendor-reset.git
cd vendor-reset
sudo dkms install .
echo "vendor-reset" >> /etc/modules

VENDOR-RESET BOOT FIX HOOKSCRIPT
Main GPU

## eGPU
#!/bin/bash
GPU='0000:0b:00.0'
[ "$2" == "pre-start" ] && echo device_specific >"/sys/bus/pci/devices/${GPU}/reset_method"

Windows 11

code 43 error: machine: pc-q35-6.2

## POWER SAVING MODE

cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
performance

cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
conservative ondemand userspace powersave performance schedutil

watch -n1 "grep \"^[c]pu MHz\" /proc/cpuinfo"

https://community.home-assistant.io/t/psa-how-to-configure-proxmox-for-lower-power-usage/323731

nano /etc/systemd/system/power-saving.service

[Unit]
Description=CPU Power Saving
After=multi-user.target

[Service]
ExecStart=/usr/bin/bash -c 'echo "powersave" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor'

[Install]
WantedBy=multi-user.target

systemctl enable power-saving


## FIX BLUETOOTH

Bluetooolfixup v2.6.8
