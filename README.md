# ALPHA CONCEPT WARNING - Heavy Update from original script on June 08, 2026

Original Script is here: https://github.com/offternet/linstaller/tree/main

This repository contains experimental installer design code. It is not production ready. Do not run this on daily-use systems, production systems, or machines containing important data.

Testing should be limited to virtual machines, disposable test drives, Raspberry Pi test systems, or other non-critical environments.

The purpose of this fork is to explore a browser-driven, modular Linux installation model using HTML/JavaScript frontend concepts and local Linux command execution.

======================================================================

# linstaller
A minimalistic, universal, easy-to-implement Linux installer

## Why?

Current Linux installers have absorbed out-of-scope functions, making them complex and difficult to debug. They also eventually adopted the logic of formatting, extracting the image, configuring, and only then installing the bootloader. While this allows for a reliable installation, it can result in an incomplete installation if certain configurations—such as generating locales or cleaning the package manager—fail. This can lead to an unbootable system. `linstaller` aims to maintain reliability while focusing on providing a bootable system rather than a pre-configured one. With the exception of the user and hostname, all configurations are handled by a post-installation script. This ensures that the system, in minimal conditions, can boot after installation.

## `linstaller` Setup

Configuring `linstaller` is incredibly simple:
- [`linstaller-backend`](https://raw.githubusercontent.com/natanael-b/linstaller/main/linstaller-backend) is the installer—place it in the image
  - These are its dependencies:
    - `grub-install`
    - `grub-pc`
    - `grub-efi`
- [`linstaller-fulldisk-setup`](https://raw.githubusercontent.com/natanael-b/linstaller/main/linstaller-fulldisk-setup)  is the formatter, providing the simplest way to configure a blank disk.
  - These are its dependencies:
    - `gdisk`
    - `mkfs.fat`

On a Ubuntu/Debian-based system, install everything you need with:

```bash
sudo apt install gdisk                  \
                 squashfs-tools         \
                 grub-common            \
                 grub-pc-bin grub-efi   \
                 grub-efi-amd64-signed  \
                 dosfstools             # For EFI partitions
```

## Disk setup

You will need to set up a disk. `linstaller` does not do this directly but provides a utility for quickly setting up a full-disk installation. Once linstaller is installed in /usr/bin, you should be able to run (as root):

```bash 
linstaller-fulldisk-setup --filesystem=ext4 /dev/sda
```

This will **erase all partitions and data** on `/dev/sda` create a 512M partition for EFI as FAT32 (even on MBR), and create a second partition with the remaining disk space, formatting it as `ext4`

> **Tip:** You can format the system’s target partition using any `mkfs`-supported format. List supported formats with: `ls /usr/sbin | grep "mkfs\." | cut -d\. -f2`

## Usage

The script supports various arguments to specify installation settings. Here are the supported arguments:



## Usage

The script supports the following arguments to specify installation settings.

### Required arguments

* `--target-partition=<target_partition>`
Specifies the target partition where the system will be installed.

* `--target-disk=<target_disk>`
Specifies the target disk where GRUB will be installed.

* `--squashfs=<squashfs_file>`
Specifies the SquashFS file containing the system image to be installed.

* `--user-name=<user_name>`
Specifies the name of the user to be created during installation.

* `--user-password=<user_password>`
Specifies the user's password.

* `--hostname=<host_name>`
Specifies the hostname for the installed system.

* `--timezone=<timezone>`
Specifies the system timezone.

* `--keyboard-map=<keyboard_layout>`
Specifies the keyboard layout.

* `--language=<language>`
Specifies the system language.

* `--i-understand-this-erases-target`
Required safety confirmation. The script will refuse to continue without this flag.

### Conditionally required arguments

* `--efi-partition=<efi_partition>`
Specifies the EFI partition. Required when the installer is running on a UEFI-booted system.

### Optional arguments

* `--home-partition=<home_partition>`
Specifies a separate user home partition.

* `--user-password-file=<password_file>`
Reads the user's password from a local file instead of passing the password directly on the command line. This is safer than `--user-password`.

* `--format-target`
Formats the target partition before installation.

* `--format-home`
Formats the home partition before installation. Only applies when `--home-partition` is used.

* `--target-fs=<filesystem>`
Specifies the filesystem to use when formatting the target partition. Default is `ext4`.

* `--home-fs=<filesystem>`
Specifies the filesystem to use when formatting the home partition. Default is `ext4`.

* `--config=<config_file>`
Loads installation settings from a local configuration file.

* `--skip-umount`
Prevents automatic cleanup unmounts at script exit. Useful only for debugging.

* `--dry-run`
Prints the actions that would be performed without actually running destructive commands.

* `--help`
Displays usage information.

## How does it work?
The script performs the following steps during installation:

1. Mounts the target partition at /target in read-write mode.
2. Mounts the home partition (if specified) at /target/home.
3. Extracts the system image from the specified SquashFS file to /target.
4. Generates the /target/etc/fstab file for automatic partition mounting.
5. Installs Grub on the destination disk (considering EFI or not).
6. Configures language, keyboard layout, time zone, hostname, and creates a user with the specified password.
7. Executes any custom post-installation script, if provided in the image.

## The post install

Since `linstaller` is designed to install the system in the most fail-safe way possible, all configurations should be handled through a post-installation script or software located in the SquashFS image at `/usr/bin/post-install`. Any arguments not supported by `linstaller` are passed to this script/software for further processing.
