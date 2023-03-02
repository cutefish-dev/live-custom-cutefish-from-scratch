# How to create a custom Cutefish live from scratch

<p align="center">
   <img src="images/live-boot.png">
</p>

This procedure shows how to create a **bootable** and **installable** Cutefish Live (along with the automatic hardware detection and configuration) from scratch.  The steps described below are also available in this repo in the `/scripts` directory.

## Authors

* **Marcos Vallim** - *Founder, Author, Development, Test, Documentation* - [mvallim](https://github.com/mvallim)
* **Ken Gilmer** -  *Commiter, Development, Test, Documentation* - [kgilmer](https://github.com/kgilmer)

See also the list of [contributors](CONTRIBUTORS.txt) who participated in this project.

## Ways of Using this Tutorial

* (Recommended) follow the directions step by step below to understand how to build an Cutefish ISO.
* Run the `build.sh` script in the `scripts` directory after checking this repo out locally.
* Fork this repo and run the github action `build`.  This will generate an ISO in your github account.

[![build-bionic](https://github.com/mvallim/live-custom-ubuntu-from-scratch/actions/workflows/build-bionic.yml/badge.svg)](https://github.com/mvallim/live-custom-ubuntu-from-scratch/actions/workflows/build-bionic.yml)
[![build-focal](https://github.com/mvallim/live-custom-ubuntu-from-scratch/actions/workflows/build-focal.yml/badge.svg)](https://github.com/mvallim/live-custom-ubuntu-from-scratch/actions/workflows/build-focal.yml)

## Terms

* `build system` - the computer environment running the build scripts that generate the ISO.
* `live system` - the computer environment that runs from the live OS, generated by a `build system`.  This may also be referred to as the `chroot environment`.
* `target system` - the computer environment that runs after installation has completed from a `live system`.

## Prerequisites (GNU/Linux Debian/Ubuntu)

Install packages we need in the `build system` required by our scripts.

```shell
sudo apt-get install \
    binutils \
    debootstrap \
    squashfs-tools \
    xorriso \
    grub-pc-bin \
    grub-efi-amd64-bin \
    mtools
```

```shell
mkdir $HOME/live-cutefish-from-scratch
```

## Bootstrap and Configure Cutefish

`debootstrap` is a program for generating OS images.  We install it into our `build system` to begin generating our ISO.

* Checkout bootstrap

  ```shell
  sudo debootstrap \
     --arch=amd64 \
     --variant=minbase \
     focal \
     $HOME/live-cutefish-from-scratch/chroot \
     http://us.archive.ubuntu.com/ubuntu/
  ```
  
  > **debootstrap** is used to create a Debian base system from scratch, without requiring the availability of **dpkg** or **apt**. It does this by downloading .deb files from a mirror site, and carefully unpacking them into a directory which can eventually be **chrooted** into.

* Configure external mount points
  
  ```shell
  sudo mount --bind /dev $HOME/live-cutefish-from-scratch/chroot/dev
  
  sudo mount --bind /run $HOME/live-cutefish-from-scratch/chroot/run
  ```

  As we will be updating and installing packages (grub among them), these mount points are necessary inside the chroot environment, so we are able to finish the installation without errors.

## Define chroot environment

*A chroot on Unix operating systems is an operation that changes the apparent root directory for the current running process and its children. A program that is run in such a modified environment cannot name (and therefore normally cannot access) files outside the designated directory tree. The term "chroot" may refer to the chroot system call or the chroot wrapper program. The modified environment is called a chroot jail.*

> Reference: https://en.wikipedia.org/wiki/Chroot

From this point we will be configuring the `live system`.

1. **Access chroot environment**

   ```shell
   sudo chroot $HOME/live-cutefish-from-scratch/chroot
   ```

2. **Configure mount points, home and locale**

   ```shell
   mount none -t proc /proc

   mount none -t sysfs /sys

   mount none -t devpts /dev/pts

   export HOME=/root

   export LC_ALL=C
   ```

   These mount points are necessary inside the chroot environment, so we are able to finish the installation without errors.

3. **Set a custom hostname**

   ```shell
   echo "cutefish-fs-live" > /etc/hostname
   ```

4. **Configure apt sources.list**

   ```shell
   cat <<EOF > /etc/apt/sources.list
   deb http://us.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
   deb-src http://us.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse

   deb http://us.archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
   deb-src http://us.archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse

   deb http://us.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
   deb-src http://us.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
   EOF
   ```

5. **Update indexes packages**

   ```shell
   apt-get update
   ```

6. **Install systemd**

   ```shell
   apt-get install -y libterm-readline-gnu-perl systemd-sysv
   ```

   > **systemd** is a system and service manager for Linux. It provides aggressive parallelization capabilities, uses socket and D-Bus activation for starting services, offers on-demand starting of daemons, keeps track of processes using Linux control groups, maintains mount and automount points and implements an elaborate transactional dependency-based service control logic.

7. **Configure machine-id and divert**

   ```shell
   dbus-uuidgen > /etc/machine-id

   ln -fs /etc/machine-id /var/lib/dbus/machine-id
   ```

   > The `/etc/machine-id` file contains the unique machine ID of the local system that is set during installation or boot. The machine ID is a single newline-terminated, hexadecimal, 32-character, lowercase ID. When decoded from hexadecimal, this corresponds to a 16-byte/128-bit value. This ID may not be all zeros.

   ```shell
   dpkg-divert --local --rename --add /sbin/initctl

   ln -s /bin/true /sbin/initctl
   ```

   > **dpkg-divert** is the utility used to set up and update the list of diversions.

8. **Upgrade packages**

   ```shell
   apt-get -y upgrade
   ```

9. **Install packages needed for Live System**

   ```shell
   apt-get install -y \
       sudo \
       ubuntu-standard \
       casper \
       lupin-casper \
       discover \
       laptop-detect \
       os-prober \
       network-manager \
       resolvconf \
       net-tools \
       wireless-tools \
       wpagui \
       locales \
       grub-common \
       grub-gfxpayload-lists \
       grub-pc \
       grub-pc-bin \
       grub2-common
   ```

   ```shell
   apt-get install -y --no-install-recommends linux-generic
   ```

10. **Graphical installer**

    ```shell
    apt-get install -y \
       ubiquity \
       ubiquity-casper \
       ubiquity-frontend-gtk \
       ubiquity-slideshow-ubuntu \
       ubiquity-ubuntu-artwork
    ```

    The next steps will appear, as a result of the packages that will be installed from the previous step, this will happen without anything having  to be informed or executed.

    1. Configure keyboard

      <p align="center">
      <img src="images/keyboard-configure-01.png">
      </p>

      <p align="center">
      <img src="images/keyboard-configure-02.png">
      </p>

    2. Console setup

      <p align="center">
      <img src="images/console-configure-01.png">
      </p>

11. **Install window manager**

    ```shell
    apt-get install -y \
        plymouth-theme-ubuntu-logo \
        ubuntu-gnome-desktop \
        ubuntu-gnome-wallpapers
    ```

12. **Install useful applications**

    ```shell
    apt-get install -y \
        clamav-daemon \
        terminator \
        apt-transport-https \
        curl \
        vim \
        nano \
        less
    ```

13. **Install Visual Studio Code (optional)**

   1. Download and install the key

      ```shell
      curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg

      install -o root -g root -m 644 microsoft.gpg /etc/apt/trusted.gpg.d/

      echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list

      rm microsoft.gpg
      ```

   2. Then update the package cache and install the package using

      ```shell
      apt-get update

      apt-get install -y code
      ```

14. **Install Google Chrome (optional)**

   1. Download and install the key

      ```shell
      wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

      echo "deb http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
      ```

   2. Then update the package cache and install the package using

      ```shell
      apt-get update

      apt-get install google-chrome-stable
      ```

15. **Install Java JDK 8 (optional)**

    ```shell
    apt-get install -y \
        openjdk-8-jdk \
        openjdk-8-jre
    ```

16. **Remove unused applications (optional)**

    ```shell
    apt-get purge -y \
        transmission-gtk \
        transmission-common \
        gnome-mahjongg \
        gnome-mines \
        gnome-sudoku \
        aisleriot \
        hitori
    ```

17. **Remove unused packages**

    ```shell
    apt-get autoremove -y
    ```

18. **Reconfigure packages**

   1. Generate locales

      ```shell
      dpkg-reconfigure locales
      ```

      1. *Select locales*
         <p align="center">
         <img src="images/locales-select.png">
         </p>

      2. *Select default locale*
         <p align="center">
         <img src="images/locales-default.png">
         </p>   

   2. Reconfigure resolvconf

      ```shell
      dpkg-reconfigure resolvconf
      ```

      1. *Confirm changes*
         <p align="center">
         <img src="images/resolvconf-confirm-01.png">
         </p>

         <p align="center">
         <img src="images/resolvconf-confirm-02.png">
         </p>

         <p align="center">
         <img src="images/resolvconf-confirm-03.png">
         </p>

   3. Configure network-manager

      ```shell
      cat <<EOF > /etc/NetworkManager/NetworkManager.conf
      [main]
      rc-manager=resolvconf
      plugins=ifupdown,keyfile
      dns=dnsmasq

      [ifupdown]
      managed=false
      EOF
      ```

   4. Reconfigure network-manager

      ```shell
      dpkg-reconfigure network-manager
      ```

19. **Cleanup the chroot environment**

   1. If you installed software, be sure to run

      ```shell
      truncate -s 0 /etc/machine-id
      ```

   2. Remove the diversion

      ```shell
      rm /sbin/initctl

      dpkg-divert --rename --remove /sbin/initctl
      ```

   3. Clean up

      ```shell
      apt-get clean

      rm -rf /tmp/* ~/.bash_history

      umount /proc

      umount /sys

      umount /dev/pts

      export HISTSIZE=0

      exit
      ```

## Unbind mount points

```shell
sudo umount $HOME/live-cutefish-from-scratch/chroot/dev

sudo umount $HOME/live-cutefish-from-scratch/chroot/run
```

## Create the CD image directory and populate it

We are now back in our `build environment` after setting up our `live system` and will continue creating files necessary to generate the ISO.

1. Access build directory

   ```shell
   cd $HOME/live-cutefish-from-scratch
   ```

2. Create directories

   ```shell
   mkdir -p image/{casper,isolinux,install}
   ```

3. Copy kernel images

   ```shell
   sudo cp chroot/boot/vmlinuz-**-**-generic image/casper/vmlinuz

   sudo cp chroot/boot/initrd.img-**-**-generic image/casper/initrd
   ```

4. Copy memtest86+ binary (BIOS)

   ```shell
   sudo cp chroot/boot/memtest86+.bin image/install/memtest86+
   ```

5. Download and extract memtest86 binary (UEFI)

   ```shell
   wget --progress=dot https://www.memtest86.com/downloads/memtest86-usb.zip -O image/install/memtest86-usb.zip

   unzip -p image/install/memtest86-usb.zip memtest86-usb.img > image/install/memtest86

   rm -f image/install/memtest86-usb.zip
   ```

## GRUB menu configuration

   1. Access build directory

      ```shell
      cd $HOME/live-cutefish-from-scratch
      ```

   2. Create base point access file for grub

      ```shell
      touch image/cutefish
      ```

   3. Create image/isolinux/grub.cfg

      ```shell
      cat <<EOF > image/isolinux/grub.cfg

      search --set=root --file /cutefish

      insmod all_video

      set default="0"
      set timeout=30

      menuentry "Try Cutefish FS without installing" {
         linux /casper/vmlinuz boot=casper nopersistent toram quiet splash ---
         initrd /casper/initrd
      }

      menuentry "Install Cutefish FS" {
         linux /casper/vmlinuz boot=casper only-ubiquity quiet splash ---
         initrd /casper/initrd
      }

      menuentry "Check disc for defects" {
         linux /casper/vmlinuz boot=casper integrity-check quiet splash ---
         initrd /casper/initrd
      }

      menuentry "Test memory Memtest86+ (BIOS)" {
         linux16 /install/memtest86+
      }

      menuentry "Test memory Memtest86 (UEFI, long load time)" {
         insmod part_gpt
         insmod search_fs_uuid
         insmod chain
         loopback loop /install/memtest86
         chainloader (loop,gpt1)/efi/boot/BOOTX64.efi
      }
      EOF
      ```

## Create manifest

Next we create a file `filesystem.manifest` to specify each package and it's version that is installed on the `live system`.  We create another file `filesystem.manifest-desktop` which specifies which files will be installed on the `target system`.  Once the Ubiquity installer completes, it will
remove packages specified in `filesystem.manifest` that are *not* listed in `filesystem.manifest-desktop`.

1. Access build directory

   ```shell
   cd $HOME/live-cutefish-from-scratch
   ```

2. Generate manifest

   ```shell
   sudo chroot chroot dpkg-query -W --showformat='${Package} ${Version}\n' | sudo tee image/casper/filesystem.manifest

   sudo cp -v image/casper/filesystem.manifest image/casper/filesystem.manifest-desktop

   sudo sed -i '/ubiquity/d' image/casper/filesystem.manifest-desktop

   sudo sed -i '/casper/d' image/casper/filesystem.manifest-desktop

   sudo sed -i '/discover/d' image/casper/filesystem.manifest-desktop

   sudo sed -i '/laptop-detect/d' image/casper/filesystem.manifest-desktop

   sudo sed -i '/os-prober/d' image/casper/filesystem.manifest-desktop
   ```

## Compress the chroot

After everything has been installed and preconfigured in the **chrooted** environment, we need to generate an image of everything that was done by following the next steps in the `build environment`.

1. Access build directory

   ```shell
   cd $HOME/live-cutefish-from-scratch
   ```

2. Create squashfs

   ```shell
   sudo mksquashfs chroot image/casper/filesystem.squashfs
   ```

   > **Squashfs** is a highly compressed read-only filesystem for Linux. It uses zlib compression to compress both files, inodes and directories. Inodes in the system are very small and all blocks are packed to minimize data overhead. Block sizes greater than 4K are supported up to a maximum of 64K.
   > **Squashfs** is intended for general read-only filesystem use, for archival use (i.e. in cases where a .tar.gz file may be used), and in constrained block device/memory systems (e.g. **embedded systems**) where low overhead is needed.

3. Write the filesystem.size

   ```shell
   printf $(sudo du -sx --block-size=1 chroot | cut -f1) > image/casper/filesystem.size
   ```

## Create diskdefines

**README** file often found on Linux LiveCD installer discs, such as an Ubuntu Linux installation CD; typically named “**README.diskdefines**” and may be referenced during installation.

1. Access build directory

   ```shell
   cd $HOME/live-cutefish-from-scratch
   ```

2. Create file image/README.diskdefines

   ```shell
   cat <<EOF > image/README.diskdefines
   #define DISKNAME  Cutefish from scratch
   #define TYPE  binary
   #define TYPEbinary  1
   #define ARCH  amd64
   #define ARCHamd64  1
   #define DISKNUM  1
   #define DISKNUM1  1
   #define TOTALNUM  0
   #define TOTALNUM0  1
   EOF
   ```

## Create ISO Image for a LiveCD (BIOS + UEFI)

1. Access image directory

   ```shell
   cd $HOME/live-cutefish-from-scratch/image
   ```

2. Create a grub UEFI image

   ```shell
   grub-mkstandalone \
      --format=x86_64-efi \
      --output=isolinux/bootx64.efi \
      --locales="" \
      --fonts="" \
      "boot/grub/grub.cfg=isolinux/grub.cfg"
   ```

3. Create a FAT16 UEFI boot disk image containing the EFI bootloader

   ```shell
   (
      cd isolinux && \
      dd if=/dev/zero of=efiboot.img bs=1M count=10 && \
      sudo mkfs.vfat efiboot.img && \
      LC_CTYPE=C mmd -i efiboot.img efi efi/boot && \
      LC_CTYPE=C mcopy -i efiboot.img ./bootx64.efi ::efi/boot/
   )
   ```

4. Create a grub BIOS image

   ```shell
   grub-mkstandalone \
      --format=i386-pc \
      --output=isolinux/core.img \
      --install-modules="linux16 linux normal iso9660 biosdisk memdisk search tar ls" \
      --modules="linux16 linux normal iso9660 biosdisk search" \
      --locales="" \
      --fonts="" \
      "boot/grub/grub.cfg=isolinux/grub.cfg"
   ```

5. Combine a bootable Grub cdboot.img

   ```shell
   cat /usr/lib/grub/i386-pc/cdboot.img isolinux/core.img > isolinux/bios.img
   ```

6. Generate md5sum.txt

   ```shell
   sudo /bin/bash -c "(find . -type f -print0 | xargs -0 md5sum | grep -v -e 'md5sum.txt' -e 'bios.img' -e 'efiboot.img' > md5sum.txt)"
   ```

7. Create iso from the image directory using the command-line

   ```shell
   sudo xorriso \
      -as mkisofs \
      -iso-level 3 \
      -full-iso9660-filenames \
      -volid "Cutefish from scratch" \
      -output "../cutefish-from-scratch.iso" \
      -eltorito-boot boot/grub/bios.img \
         -no-emul-boot \
         -boot-load-size 4 \
         -boot-info-table \
         --eltorito-catalog boot/grub/boot.cat \
         --grub2-boot-info \
         --grub2-mbr /usr/lib/grub/i386-pc/boot_hybrid.img \
      -eltorito-alt-boot \
         -e EFI/efiboot.img \
         -no-emul-boot \
      -append_partition 2 0xef isolinux/efiboot.img \
      -m "isolinux/efiboot.img" \
      -m "isolinux/bios.img" \
      -graft-points \
         "/EFI/efiboot.img=isolinux/efiboot.img" \
         "/boot/grub/bios.img=isolinux/bios.img" \
         "."
   ```

## Alternative way, if previous one fails, create an Hybrid ISO

1. Create a ISOLINUX (syslinux) boot menu

   ```shell
   cat <<EOF> isolinux/isolinux.cfg
   UI vesamenu.c32

   MENU TITLE Boot Menu
   DEFAULT linux
   TIMEOUT 600
   MENU RESOLUTION 640 480
   MENU COLOR border       30;44   #40ffffff #a0000000 std
   MENU COLOR title        1;36;44 #9033ccff #a0000000 std
   MENU COLOR sel          7;37;40 #e0ffffff #20ffffff all
   MENU COLOR unsel        37;44   #50ffffff #a0000000 std
   MENU COLOR help         37;40   #c0ffffff #a0000000 std
   MENU COLOR timeout_msg  37;40   #80ffffff #00000000 std
   MENU COLOR timeout      1;37;40 #c0ffffff #00000000 std
   MENU COLOR msg07        37;40   #90ffffff #a0000000 std
   MENU COLOR tabmsg       31;40   #30ffffff #00000000 std

   LABEL linux
    MENU LABEL Try Cutefish FS
    MENU DEFAULT
    KERNEL /casper/vmlinuz
    APPEND initrd=/casper/initrd boot=casper

   LABEL linux
    MENU LABEL Try Cutefish FS (nomodeset)
    MENU DEFAULT
    KERNEL /casper/vmlinuz
    APPEND initrd=/casper/initrd boot=casper nomodeset
   EOF
   ```

2. Include syslinux bios modules

   ```shell
   apt install -y syslinux-common && \
   cp /usr/lib/ISOLINUX/isolinux.bin isolinux/ && \
   cp /usr/lib/syslinux/modules/bios/* isolinux/
   ```

3. Create iso from the image directory

   ```shell
   sudo xorriso \
      -as mkisofs \
      -iso-level 3 \
      -full-iso9660-filenames \
      -volid "Cutefish from scratch" \
      -output "../cutefish-from-scratch.iso" \
    -isohybrid-mbr /usr/lib/ISOLINUX/isohdpfx.bin \
    -eltorito-boot \
        isolinux/isolinux.bin \
        -no-emul-boot \
        -boot-load-size 4 \
        -boot-info-table \
        --eltorito-catalog isolinux/isolinux.cat \
    -eltorito-alt-boot \
        -e /EFI/boot/efiboot.img \
        -no-emul-boot \
        -isohybrid-gpt-basdat \
    -append_partition 2 0xef EFI/boot/efiboot.img \
      "$HOME/live-cutefish-from-scratch/image"
   ```

## Make a bootable USB image

It is simple and easy, using "dd"

```shell
sudo dd if=cutefish-from-scratch.iso of=<device> status=progress oflag=sync
```

## Summary

This completes the process of creating a live Ubuntu installer from scratch.  The generated ISO may be tested in a virtual machine such as `VirtualBox` or written to media and booted from a standard PC.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [GitHub](https://github.com/cutefish-dev/live-custom-cutefish-from-scratch) for versioning. For the versions available, see the [tags on this repository](https://github.com/cutefish-dev/live-custom-cutefish-from-scratch/tags).

## License

This project is licensed under the GNU GENERAL PUBLIC LICENSE - see the [LICENSE](LICENSE) file for details
