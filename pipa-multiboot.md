# EFI Multiboot for Pipa

![Usage](https://github.com/TheMojoMan/xiaomi-pipa/blob/main/pipa-efi-multiboot.mov)

## Installation

1. **Extract the archive**  
Change into your Downloads folder and decompress 'pipa-efi-multiboot.tar':

```bash
   cd Downloads && tar -xf pipa-efi-multiboot.tar
```
2. **Optional: Clean up unused files**  

If you have a small ESP partition, remove unneeded folders/files in EFI/. For example, if you don't use Ubuntu:

```bash
   rm -r EFI/ubuntu
```
3. **Find your ESP partition**  

Check for partition number of your ESP partition:

```bash
   blkid | grep esp
```

4. **Mount your ESP partition**  

```bash
   mkdir tmp && sudo mount /dev/sda35 tmp
```

*(Replace `/dev/sda35` with your actual ESP partition)*

5. **Copy EFI folder**  

```bash
   sudo cp -r EFI tmp/
```

6. **Flash the boot image**  

Flash `pipa_dualrole.img` to boot_a or boot_b. Choose the one that does **NOT** contain your Android boot image.

### Flashing via USB (Fastboot)

Restart tablet and hold down **Vol-down** key to enter fastboot mode:

```bash
# For boot_a
fastboot flash boot_a pipa_dualrole.img

# For boot_b
fastboot flash boot_b pipa_dualrole.img
```
### Flashing via Linux (Advanced)

If you know what you're doing, you can also flash using Linux:

```bash
# Backup boot_a and boot_b, transfer to PC before flashing
sudo dd if=/dev/sde12 of=boot_a
sudo dd if=/dev/sde37 of=boot_b

# For boot_a
sudo dd if=pipa_dualrole.img of=/dev/sde12

# For boot_b
sudo dd if=pipa_dualrole.img of=/dev/sde37
```

**⚠️ Important:** Double check that these are the correct partition numbers:

```bash
sudo parted /dev/sde
# Enter 'print' and look for boot_a and boot_b
```

## Creating Your Own EFI Files (UKIs)

Use `systemd-ukify` to create Universal Kernel Images. The `--initrd` parameter is optional:

```bash
ukify build --linux <your_kernel> --devicetree=<your_dtb> --initrd=<your_initrd.img> --cmdline="<cmds_to_boot_your_linux_image>" -o <name_of_efi_file_that_will_be_created>
```

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

## Distribution-Specific Notes

- **Ubuntu users**: Change your partition name from 'linux' to 'ubuntu'
- **Fedora users**: Change your partition name from 'linux' to 'fedora'  
- **Custom setup**: Navigate to your `/boot` folder where you'll find kernel and dtb files. Use `root=PARTLABEL=linux` in cmdline to create an EFI file that boots from a partition named 'linux'
