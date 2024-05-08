ProxMox Setup for PCIe Passthrough
https://www.youtube.com/watch?v=_hOBAGKLQkI&t=434s
https://www.youtube.com/watch?v=BoMlfk397h0&t=1612s
https://pve.proxmox.com/wiki/PCI(e)_Passthrough

1A. -Legacy Systems; Add IOMMU Support- (GRUB)

nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
	- OR -
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"

Save file and close

update-grub


1B. -EFI Boot Systems; Add IOMMU Support- (Systemd (ZFS))

nano /etc/kernel/cmdline
%> intel_iommu=on
    - OR -
%> amd_iommu=on

Save file and close

proxmox-boot-tool refresh


2. -Load VFIO modules at boot-

nano /etc/modules

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

Save file and close


3.-Blacklist graphic drivers (optional)-

echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf

echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf

-Configure GPU for PCIe Passthrough-

	- Find your GPU
lspci

	- Enter the PCI identifier
lspci -n -s 82:00 -v

	- Copy the HEX values from your GPU here:
echo "options vfio-pci ids=####.####,####.#### disable_vga=1"> /etc/modprobe.d/vfio.conf


4. -Apply all changes-

update-initramfs -u -k all

5. Verify PCIE PASSTHROUGH
https://pve.proxmox.com/wiki/PCI_Passthrough#Verifying_IOMMU_parameters

Verify IOMMU is enabled
dmesg | grep -e DMAR -e IOMMU
	There should be a line that looks like "DMAR: IOMMU enabled". If there is no output, something is wrong.

Verify IOMMU interrupt remapping is enabled
dmesg | grep 'remapping'	
	If you see one of the following lines: then remapping is supported.
	AMD-Vi: Interrupt remapping enabled
	DMAR-IR: Enabled IRQ remapping in x2apic mode ('x2apic' can be different on old CPUs, but should still work)

5. --REBOOT--
