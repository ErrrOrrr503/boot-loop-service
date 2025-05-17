# Boot-Loop systemd serevice

The service let you combine ventoy and full linux installation in one usb stick. No Ventoy plugins needed.

[!TIP]
Ventoy can boot vhd and vdi. Maybe they are better way to achieve the goal.

## Principle

Linux's /boot and /boot/efi is placed into an .img, which ventoy boots. / is stored as a separate partition (ventoy can leave empty space in the end of the drive during installation). Of cause, linux is fully encrypted, both / and /boot.

## Layout

* VentoyPartition (exfat/ntfs)
    * IMG/
        * booter.img
            * BIOS partition (for MBR boot)
            * EFI (boot-efi) partition
            * Encrypted boot partition
                * kernel
                * initramfs with root crypyto keyfile
                * grub theme
    * ISO/
        * some installers, etc.
* VentoyEFI
* Encrypted root partition

## Boot process:

* Ventoy boots booter.img either in EFI or in BIOS mode.
* GRUB asks password and decrypts boot, loads kernel and initramfs.
* Kernel recognizes /, as it is a separate partition (not hidden in booter.img), decrypts and mounts it.
* fstab-based systemd service mounts VentoyPartition.
* boot-loop service initiates *multy-partitioned* loop from booter.img and decrypts boot.
[!NOTE]
Multi-partitioned loop requirement is in fact the cause of the service. Fstab just supports only single-partition loops.
* fstab-based service mounts (decrypted) boot and boot-efi from the loop.
* System correctly booted!

## Usage

Somewhat repeat the advices layout, edit .in templates accordingly (UUIDS, paths, etc).
