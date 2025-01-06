ProxMox 8 PCIe Passthrough Guide


First, find out if Proxmox is boothing using GRUB (Legacy BIOS) or Systemd (EFI boot).
During booting look at your screen, if the screen has a Blue box then the system is booting with GRUB.
If it is the screen is Black, then the system boots using Systemd



1A. (Legacy System) Add IMMOU support for GRUB

Edit /etc/default/grub:
```	
nano /etc/default/grub
```
Content /etc/default/grub:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```
OR
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```
Save file and close.

Then update grub:
```
update-grub
```



1B. (EFI boot system) Add IMMOU support for Systemd (ZFS):

Edit /etc/kernel/cmdline:
```
nano /etc/kernel/cmdline
```
Content /etc/kernel/cmdline:
```	
intel_iommu=on
```
OR
```
amd_iommu=on
```
Save file and close.

Then update proxmox-bootl-tool:
```
proxmox-boot-tool refresh
```



2. Load VFIO modules at boot

edit /etc/modules:
```
nano /etc/modules
```
Content /etc/modules:
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
Save file and close



3.Blacklist graphic drivers (optional):

edit /etc/modprobe.d/iommu_unsafe_interrupts.conf & /etc/modprobe.d/kvm.conf:
```
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```
edit /etc/modprobe.d/blacklist.conf:
```
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
```

Configure GPU for PCIe Passthrough

Find your GPU / PCie device
```
lspci
```
Enter the PCI identifier
```
lspci -n -s 82:00 -v
```
Copy the HEX values from your GPU to /etc/modprobe.d/vfio.conf:
```
echo "options vfio-pci ids=####.####,####.#### disable_vga=1"> /etc/modprobe.d/vfio.conf
```



4. Apply all changes

Update-inirafs:
```
update-initramfs -u -k all
```



5. Verify PCIE PASSTHROUGH

Verify IOMMU is enabled
```
dmesg | grep -e DMAR -e IOMMU
```	
There should be a line that looks like "DMAR: IOMMU enabled". If there is no output, something is wrong.

Verify IOMMU interrupt remapping is enabled:
```
dmesg | grep 'remapping'	
```	

If you see one of the following lines: then remapping is supported.
```	
AMD-Vi: Interrupt remapping enabled
DMAR-IR: Enabled IRQ remapping in x2apic mode ('x2apic' can be different on old CPUs, but should still work)
```
5. Reboot & ready for device passthrough.



Guide:
```	
https://www.youtube.com/watch?v=_hOBAGKLQkI&t=434s
```
```
https://www.youtube.com/watch?v=BoMlfk397h0&t=1612s
```
```
https://pve.proxmox.com/wiki/PCI(e)_Passthrough
````

Verify PCIE Passthtrough documentation:
```
https://pve.proxmox.com/wiki/PCI_Passthrough#Verifying_IOMMU_parameters
```
```
https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/
```

