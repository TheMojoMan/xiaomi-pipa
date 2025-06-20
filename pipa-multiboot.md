# EFI Multiboot for Pipa

<img src="https://github.com/TheMojoMan/xiaomi-pipa/blob/main/pipa-efi-multiboot.jpg" width=300 />

## Installation (using Linux or Termux)

Installation can be done from your tablet running **Linux** or from your rooted Android running **Termux**. Run `termux-setup-storage` first in Termux for easy file access.

1. **Download and extract the [archive](https://github.com/TheMojoMan/xiaomi-pipa/releases/download/Pipa-efi-multiboot/pipa-efi-multiboot.tar)**
   
Change into your Downloads folder and decompress 'pipa-efi-multiboot.tar':

```bash
   # Linux
   cd Downloads && tar -xf pipa-efi-multiboot.tar

   # Termux
   cd storage/downloads && tar -xf pipa-efi-multiboot.tar
```
2. **Optional: Clean up unused files**  

If you have a small ESP partition, remove unneeded folders/files in EFI/. For example, if you don't use Ubuntu:

```bash
   rm -r EFI/ubuntu
```

3. **Mount your ESP partition**  

```bash
   # Linux
   mkdir tmp && sudo mount /dev/disk/by-partlabel/esp tmp

   # Termux
   mkdir tmp && sudo mount /dev/block/by-name/esp tmp
```

4. **Copy EFI folder and unmount esp**  

```bash
   sudo cp -r EFI tmp/
   sudo umount tmp
```

5. **Flash the boot image**  

Flash `pipa_dualrole.img` to boot_a or boot_b. Choose the one that does **NOT** contain your Android boot image.

### Flashing via Linux

If you know what you're doing, you can also flash using Linux:

```bash
# Backup boot_a and boot_b, transfer to PC before flashing
sudo dd if=/dev/disk/by-partlabel/boot_a of=boot_a_backup.img
sudo dd if=/dev/disk/by-partlabel/boot_b of=boot_b_backup.img

# For boot_a
sudo dd if=pipa_dualrole.img of=/dev/disk/by-partlabel/boot_a

# For boot_b
sudo dd if=pipa_dualrole.img of=/dev/disk/by-partlabel/boot_b
```

### Flashing via Termux

```bash
# Backup boot_a and boot_b, transfer to PC before flashing
sudo dd if=/dev/block/by-name/boot_a of=boot_a_backup.img
sudo dd if=/dev/block/by-name/boot_b of=boot_b_backup.img

# For boot_a
sudo dd if=pipa_dualrole.img of=/dev/block/by-name/boot_a

# For boot_b
sudo dd if=pipa_dualrole.img of=/dev/block/by-name/boot_b
```

### Flashing via USB

Restart tablet and hold down **Vol-down** key to enter fastboot mode:

```bash
# For boot_a
fastboot flash boot_a pipa_dualrole.img

# For boot_b
fastboot flash boot_b pipa_dualrole.img
```

## Usage
 - Use `Vol-up` and `Vol-down` buttons to change between options.
 - Use `Power` button to boot distro / reboot / shutdown.

> **Return to Android**: Press `Vol-down` button 2-3 x __right after__ 'Mu-Qcom' image appears.  
> <img src="https://github.com/TheMojoMan/xiaomi-pipa/blob/main/mu-qcom.jpg" width=100 />

## Creating Your Own EFI Files

Use `systemd-ukify` to create Universal Kernel Images (UKIs). The `--initrd` parameter is optional:

```bash
ukify build --linux <your_kernel> --devicetree=<your_dtb> --initrd=<your_initrd.img> --cmdline="<cmds_to_boot_your_linux_image>" -o <name_of_efi_file_that_will_be_created>
```
> Depending on distribution you might find kernel and dtb files in `/boot` folder.

### Examples

#### PostmarketOS

```bash
ukify build --linux zImage --devicetree=dtb --initrd=initrd.img --cmdline="quiet pmos_boot_uuid=054bf566-ce53-4e59-bfe1-732bdbb9f12f pmos_root_uuid=615c6c38-6b97-46fa-826b-39a482799856 pmos_rootfsopts=defaults fbcon=rotate:1" -o pmos_6.14.2.efi
```

#### Arch Linux
```bash
ukify build --linux zImage --devicetree=dtb --initrd=initrd.img --cmdline="noquiet loglevel=0 fbcon=rotate:1 root=LABEL=arch_rootfs rw" -o arch_6.14.2.efi

```

#### Ubuntu

```bash
ukify build --linux vmlinuz-6.12.0-sm8250-domin746826+ --devicetree=dtb-6.12.0-sm8250-domin746826+ --cmdline="noquiet loglevel=0 fbcon=rotate:1 root=PARTLABEL=ubuntu rw" -o ubuntu_6.12.0.efi
```

## Troubleshooting
 - Linux distro does not boot / boots but is not functioning properly:  
   This can be caused when your installed distro did a kernel update and kernel modules are changed. Please ask your distro provider for an updated .efi file which contains the matching kernel. Mount your esp and copy the new .efi file to tmp/EFI/<your_distro_name> e.g. ˋsudo cp <updated.efi> tmp/EFI/archˋ or ˋsudo cp <updated.efi> tmp/EFI/ubuntuˋ.

## Thanks
 - The author of the Mu-Qcom efi boot loader.
 - [Timofey](https://github.com/timoxa0) for initial refind files/config for nabu which I adapted for pipa.
 - rmux and domin746826 for improving readability and for helpful improvements.
