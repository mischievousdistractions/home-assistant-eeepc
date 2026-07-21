# Home Assistant on older hardware (Asus Seashell EeePC 1005PE)
This repo aims at remembering how I installed Home Assistant on my old Asus EeePC 1005PE laptop from 2010.  
I followed the steps from the following issue and aside from the resizing of the latest partition, it just worked: [Installing Home Assistant OS on Legacy BIOS Hardware: ThinkPad X61s Success Story](https://github.com/home-assistant/home-assistant.io/issues/39584)  

In case the original content disappears, here are the steps taken directly from it (I only changed the steps to resize because the original ones didn't work for me, rephrased some snippets and added some comments):

## Hardware and Issues
- Device: Asus Seashell EeePC 1005PE
- CPU: Intel Atom N450 @1.66GHz (x86-64 architecture)
- RAM: 1GB (upgraded to 2GB)
- BIOS: Legacy only (no UEFI)

The main issue to overcome was the lack of BIOS UEFI support that is now mandatory for HAOS.  


## Installation

### 1. Boot the GNU-Linux live environment of your choice
I used [Artix Linux](https://artixlinux.org/) (with XFCE)

### 2. Open a terminal and switch to root prompt

```
sudo -l
```

### 3. Check your disks and identify where you want to install HAOS

```
lsblk
fdisk -l /dev/sda
```
(replace `/dev/sda` by the name of your chosen drive in the next steps)

### 4. Proceed with the installation of HAOS on your drive, using the Method 1 of the [official documentation](https://www.home-assistant.io/installation/generic-x86-64/)
- Extract the downloaded HAOS image if compressed (link is in the step 5 of method 1)

```
cd /location/of/your/downloaded/image/
xz -d haos-generic-x86-64-XX.Y.img.xz
```

- Copy the entire HAOS image to your selected drive (including all partitions)

```
dd if=haos-generic-x86-64-XX.Y.img of=/dev/sda bs=4M status=progress
sync
```

### 6. Expand the last partition to use the full drive and create a small bridge partition for GRUB Legacy  

Use your partition editor of choice (I used gparted)
- Select the correct drive and expand the latest partition (`/dev/sda8` in my case) up to the end of the available space, leaving at least 10MB of free space.  
- Then create a ext4 partition in the remaining space (in my case, this created the `/dev/sda9` partition and I will use this name for the next steps).

### 7. Install GRUB Legacy Bridge

*All the following operations are to be done with root privileges*  

- Mount bridge partition

```
mount /dev/sda9 /mnt
```

- Install GRUB Legacy

```
grub-install --root-directory=/mnt /dev/sda --force
```

- Create the bridge configuration file

```
nano /mnt/boot/grub/grub.cfg
```
- Add this to the file (It didn't exist in my case, the previous command created it)

```
set root=(hd0,1)
configfile /efi/boot/grub.cfg
```
`Ctrl + O` to save, `Ctrl + X` to exit the nano editor

### 8. BIOS Settings

Ensure your BIOS has Legacy boot mode enabled and no Secure boot enabled.

## How Does it Work
*Directly from the initial link explanations:*  

> HAOS Partition Structure (GPT/UEFI, does not work on older hardware):
> - sda1: `hassos-boot` (bootloader files)
> - sda2: `hassos-kernel0` (active kernel)
> - sda3: `hassos-system0` (root filesystem, SquashFS)
> - sda4: `hassos-kernel1` (backup kernel)
> - sda8: `hassos-data` (user data)
>
> GRUB Legacy Limitations:
> - No GPT partition table support
> - Cannot read SquashFS kernel partitions
> - Missing PARTUUID support
>
> Boot Sequence with workaround:
> 1. Legacy BIOS loads GRUB Legacy from bridge partition (/dev/sda9)
> 2. GRUB Legacy chainloads the second GRUB from HAOS boot partition (/dev/sda1)
> 3. Second GRUB loads modern GPT/SquashFS boot process
> 4. HAOS starts normally
> 
> The bridge partition method creates compatibility between legacy BIOS limitations and modern HAOS requirements without modifying the core HAOS image. This preserves update functionality while enabling installation on hardware that would otherwise be incompatible.
> 
> This solution works for any legacy x86-64 system lacking UEFI support, providing a path forward as Home Assistant phases out Core and Supervised installation methods.

## Additional Information
HAOS version installed: haos_generic-x86-64-18.1  

## Last Words
Many thanks to [motaviegas](https://github.com/motaviegas) for this great workaround and the explanations.
