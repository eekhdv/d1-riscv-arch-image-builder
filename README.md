# Image Builder for Archlinux on an Allwinner D1 / Sipeed Lichee RV
These scripts compile, copy, bake, unpack and flash a [ready](https://wiki.archlinux.org/title/installation_guide#Configure_the_system) to use RISC-V Archlinux.  
*With "ready to use" i mean that it boots, you still need to configure everything!*

Special thanks to **smaeul** for all their work!


## Components
- mainline OpenSBI
- U-Boot based on https://github.com/eekhdv/u-boot_sunxi_licheerv
- mainline kernel
- WiFi driver (rtl8723ds) based on https://github.com/lwfinger/rtl8723ds
- RootFS based on https://archriscv.felixc.at

## How to build
### 1. Install requirements: 

#### For Archlinux
```sh
pacman -Sy riscv64-linux-gnu-gcc swig cpio python3 python-setuptools base-devel bc
```
* If you want to `chroot` into the RISC-V image, you also need `arch-install-scripts qemu-user-static qemu-user-static-binfmt`

#### For Debian
```sh
apt install -y gcc-riscv64-linux-gnu bison flex python3-dev libssl-dev swig cpio python3-setuptools build-essential bc
```
* If you want to `chroot` into the RISC-V image, you also need `arch-install-scripts qemu-user-static`

### 2. Compile

1. Edit `consts.sh` to your needs.
2. Run `1_compile.sh` which compiles everything into the `output` folder.

### 3. Burn Image
Run `2_create_sd.sh /dev/<device>` to flash everything on the SD card.

**(root password is `archriscv`)**

## Configure your Archlinux

There are 2 ways to configure Archlinux:

### Setup directly on the board

#### Prerequisite
1. Mount the system from the SD card with Archlinux (it will be on the 2nd partition)
    ```sh 
    sudo mount /def/<device>[p]2 ./mnt
    ```
2. Copy the `networkmanager` folder to the system
    ```sh 
    sudo cp -r ./networkmanager ./mnt/root
    ```
3. Umount the mounted system
    ```sh 
    sudo umount ./mnt
    ```
4. Insert the SD card into the board. Run ArchLinux
5. Install NetworkManager
    ```sh
    cd /root/netwokmanager
    pacman -U ./*.zst
    ```
6. Connect to the internet using `nmtui`
7. Set up time 
    ```sh
    timedatectl set-ntp true
    ```
8. Synchronize packages
    ```sh
    pacman -Sy --disable-sandbox
    ```

### Setup using `arch-chroot`
The second script requires `arch-install-scripts`, `qemu-user-static-bin` (AUR) and `binfmt-qemu-static` (AUR) for an architectural chroot.
If you don't want to use/do this, change `USE_CHROOT` to `0` in `consts.sh`.  

## Using loop image file instead of a SD card
Simply loop it using `sudo losetup -f -P <file>` and then use `/dev/loopX` as the target device.

## Notes
* *Keep in mind, that this is just a extracted rootfs with **no** configuration. You probably want to update the system, install an editor and take care of network access/ssh*
* Some commits are pinned, this means that in the future this script might stop working since often a git HEAD is checked out. This is intentional.
* The second script uses `sudo` for root access. Like any random script from a random stranger from the internet, have a look at the code first and use at own risk!
* Things are rebuild whenever the corresponding `output/<file>` is missing. For example, the kernel is rebuilt when there is no `Image` file.

# Status

## (eekhdv) 31.03.2025
- update rootfs version
- add instruction for setting up without chroot
- change u-boot base

## 25.07.2023
- forgot to keep this up to date...
- mainline kernel fully supports the D1
- kernel updated to 6.4 (mainline)
- OpenSBI updated to 1.3

## 4.11.2022
- WiFi is working again

## 3.11.2022
- updated U-Boot and remove boot0 (handled by [U-Boot](https://github.com/smaeul/u-boot/releases/tag/d1-2022-10-31))
- updated kernel to 6.1.0-rc3
    - **THIS BRAKES BUILT-IN WIFI**

## 13.09.2022
- kernel is back at 5.18-rc1 due to being more reliable

## 14.06.2022
- kernel updated to 5.19-rc1
- added initramfs support (untested)
- added extlinux support

## 22.04.2022
- kernel includes modules for USB LAN adapter
- swap is enabled
## 09.04.2022
- HDMI, tested with LXDE/LXDM
## 07.04.2022
- WiFi \o/
## 06.04.2022
- Kernel is based on 5.18-rc1
- WiFi driver fails to build (linking error :feelsbadman:)
- Both kernel and U-Boot are using `nezha_defconfig` since i found them more reliable
    - With the [licheerv_linux_defconfig](https://andreas.welcomes-you.com/media/files/licheerv_linux_defconfig) from [here](https://andreas.welcomes-you.com/boot-sw-debian-risc-v-lichee-rv/) the kernel fails to find the sd card (or its partitions? not sure what exactly goes wrong)
- HDMI is not working (at least on the one screen i've tested it)

