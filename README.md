# linux-grub-dual-boot-repair
Step-by-step guide to troubleshooting and repairing GRUB rescue errors in a Kali Linux and Windows dual-boot system using Linux commands and UEFI bootloader recovery.


# Kali Linux + Windows Dual-Boot GRUB Rescue Recovery

## Overview

This repository documents the troubleshooting and recovery of a
Kali Linux + Windows dual-boot system that entered GRUB rescue mode
after a Windows update.

The system previously booted Kali Linux and Windows normally.

After restarting following a Windows update, the system displayed:

grub rescue>

error: ../../grub-core/kern/fs.c:122:unknown filesystem

Instead of reinstalling either operating system, I diagnosed the
existing partitions and repaired the UEFI GRUB bootloader.

---

# Problem

After a Windows update and restart:

    error: ../../grub-core/kern/fs.c:122:unknown filesystem
    grub rescue>

The normal GRUB boot menu was no longer loading.

---

# Step 1 — Inspect available disks and partitions

From the GRUB rescue prompt:

    ls

GRUB detected:

    (hd0)
    (hd0,gpt1)
    (hd0,gpt2)
    (hd0,gpt3)
    (hd0,gpt4)
    (hd0,gpt5)
    (hd0,gpt6)
    (hd0,gpt7)
    (hd0,gpt8)
    (hd0,gpt9)
    (hd0,gpt10)

---

# Step 2 — Identify the Kali Linux partition

Each GPT partition was checked individually:

    ls (hd0,gpt1)
    ls (hd0,gpt2)
    ls (hd0,gpt3)
    ls (hd0,gpt4)
    ls (hd0,gpt5)
    ls (hd0,gpt6)
    ls (hd0,gpt7)
    ls (hd0,gpt8)
    ls (hd0,gpt9)
    ls (hd0,gpt10)

The important result was:

    (hd0,gpt7): Filesystem is ext2.

Although GRUB reported the filesystem as ext2, it was the Linux
filesystem containing the Kali installation.

---

# Step 3 — Verify the Kali installation

The contents of the partition were inspected:

    ls (hd0,gpt7)/

The partition contained Linux directories including:

    boot/
    etc/
    home/
    usr/
    var/
    root/
    dev/
    proc/
    sys/

It also contained:

    vmlinuz
    initrd.img

This confirmed that the Kali installation was still present.

---

# Step 4 — Verify the GRUB installation

The GRUB directory was checked:

    ls (hd0,gpt7)/boot/grub/

The output included:

    x86_64-efi/
    locale/
    fonts/
    grubenv
    grub.cfg

This confirmed that the GRUB configuration files were still present.

---

# Step 5 — Temporarily boot Kali

The GRUB root and prefix were manually configured:

    set root=(hd0,gpt7)

    set prefix=(hd0,gpt7)/boot/grub

Then the normal GRUB module was loaded:

    insmod normal

And the normal GRUB menu was started:

    normal

This allowed the existing Kali installation to boot.

---

# Step 6 — Inspect the Linux partition layout

After booting into Kali, the disk layout was inspected with:

    lsblk -f

The important partitions were identified:

    /dev/nvme0n1p6
        FAT32
        mounted at /boot/efi

    /dev/nvme0n1p7
        ext4
        Kali Linux root filesystem

A separate EFI partition was also present for Windows.

---

# Step 7 — Check UEFI boot entries

The existing UEFI boot entries were inspected:

    sudo efibootmgr -v

The system contained entries for:

    Windows Boot Manager
    kali
    UEFI Firmware Settings
    UEFI removable devices
    UEFI network device

This confirmed that the system was using UEFI boot mode.

---

# Step 8 — Reinstall GRUB to the Kali EFI partition

The GRUB EFI bootloader was reinstalled using:

    sudo grub-install \
      --target=x86_64-efi \
      --efi-directory=/boot/efi \
      --bootloader-id=kali \
      --recheck

The command completed with:

    Installation finished. No error reported.

---

# Step 9 — Regenerate the GRUB configuration

The GRUB configuration was regenerated:

    sudo update-grub

The output detected the installed Kali kernels and Windows Boot Manager.

For example:

    Found linux image: /boot/vmlinuz...
    Found initrd image: /boot/initrd...
    Found Windows Boot Manager ...
    Adding boot menu entry for UEFI Firmware Settings

    done

This recreated the GRUB boot menu.

---

# Step 10 — Verify the UEFI boot order

The boot configuration was checked again:

    sudo efibootmgr

The system showed:

    Boot0002* kali
    Boot0000* Windows Boot Manager

with Kali placed in the boot order.

---

# Result

The GRUB bootloader was successfully repaired without reinstalling
Kali Linux or Windows.

The Kali installation, user files and Windows installation remained
intact.

The system could boot through GRUB and provide access to:

    Kali GNU/Linux

and

    Windows Boot Manager

---

# Commands Summary

## GRUB rescue

    ls

    ls (hd0,gpt1)
    ls (hd0,gpt2)
    ls (hd0,gpt3)
    ls (hd0,gpt4)
    ls (hd0,gpt5)
    ls (hd0,gpt6)
    ls (hd0,gpt7)
    ls (hd0,gpt8)
    ls (hd0,gpt9)
    ls (hd0,gpt10)

    ls (hd0,gpt7)/
    ls (hd0,gpt7)/boot/grub/

## Temporary GRUB boot

    set root=(hd0,gpt7)
    set prefix=(hd0,gpt7)/boot/grub
    insmod normal
    normal

## Kali Linux

    lsblk -f

    sudo efibootmgr -v

    sudo grub-install \
      --target=x86_64-efi \
      --efi-directory=/boot/efi \
      --bootloader-id=kali \
      --recheck

    sudo update-grub

    sudo efibootmgr

---

# Skills Demonstrated

- Linux troubleshooting
- GRUB troubleshooting
- UEFI bootloader recovery
- GPT partition identification
- Linux filesystem identification
- EFI partition understanding
- Dual-boot troubleshooting
- Command-line troubleshooting
- Bootloader configuration
- Windows/Linux interoperability

---

# Important Note

This was performed on my own computer for troubleshooting and
learning purposes.

The commands above should not be blindly copied to another system.
Partition names such as `(hd0,gpt7)` and `/dev/nvme0n1p7` are specific
to this system and can be different on another computer.
