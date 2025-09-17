Passing DAS Drives to OMV VM in Proxmox and Setting Up SMB

This guide explains how to attach physical DAS drives to an OpenMediaVault (OMV) VM in Proxmox and configure them as SMB network shares.

Requirements

Proxmox VE (tested on 7.x/8.x/9.0)

OMV VM installed (any version)

DAS with 1 or more physical drives connected to the Proxmox host

Steps
1️⃣ Identify Disks on Proxmox

Log into your Proxmox host and list connected disks:

lsblk


Check /dev/disk/by-id/ for stable identifiers:

ls -l /dev/disk/by-id/


Example output:

wwn-0x5000c5005db84727 -> ../../sda
wwn-0x5000c500b2f7a696 -> ../../sdb


Use by-id names to prevent errors if disk order changes.

2️⃣ Stop OMV VM
qm shutdown <VMID>


Replace <VMID> with your OMV VM ID.

3️⃣ Attach DAS Drives to OMV VM

Important: Do not attach as scsi0 if that is your boot disk. Use scsi1/scsi2 or virtio1/virtio2.

Example using SCSI:
qm set <VMID> -scsi1 /dev/disk/by-id/wwn-0x5000c5005db84727
qm set <VMID> -scsi2 /dev/disk/by-id/wwn-0x5000c500b2f7a696

Or using VirtIO (recommended for speed):
qm set <VMID> -virtio1 /dev/disk/by-id/wwn-0x5000c5005db84727
qm set <VMID> -virtio2 /dev/disk/by-id/wwn-0x5000c500b2f7a696

4️⃣ Start the OMV VM
qm start <VMID>

5️⃣ Configure Disks in OMV

Log in to OMV Web GUI.

Go to Storage → Disks → verify new disks appear (sdb, sdc).

Wipe disks if new: Disks → Wipe.

Create a filesystem: Storage → File Systems → Create → choose EXT4 or XFS.

Mount the filesystem.

6️⃣ Create SMB Share

Go to Access Rights Management → Shared Folders → Add → select mounted filesystem.

Go to Services → SMB → Shares → Add → select the shared folder.

Enable SMB service: Services → SMB → Enable.

Your DAS drives are now accessible as SMB NAS shares on your network.

✅ Notes

Always use disk by-id names to avoid drive order issues after reboot.

Never attach the boot disk (scsi0/virtio0) to a raw passthrough.

Ext4 is stable, XFS can perform better for large NAS storage.
