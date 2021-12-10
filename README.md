# dployd parser

This repository contains the parser that turns a configuration file, inspired by `Docker`, into a
replicable, installable and deployable operating system for different architectures.

# Why?

The testing or deployment of software often requires the creation of multiple instances of deployments 
on bare metal devices. With this parser it is possible to easily create bootable images of 
operating systems with their configuration done ahead of the actual deployment.

# How?

This parser is inspired by the simple syntax of `Docker` and converts that syntax into 
a configuration file used by [HashiCorp Packer](https://packer.io), with the help of the builder
[VirtualBox](https://www.packer.io/docs/builders/virtualbox) and [this] awesome (https://github.com/mkaczanowski/packer-builder-arm) 
plugin we are able to create bootable images for different architectures. Since Packer requires a
verbose configuration we created our own syntax to reduce the time needed to create bootable images.

## Imagefile

The Imagefile contains the instructions that are necessary to create a bootable image. The following
example creates an image of Raspberry Pi OS, enables `ssh` and installs `nginx`.
```
FROM https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/2021-05-07-raspios-buster-armhf-lite.zip
ARCH ARM64
ON-DEVICE

FS vfat /boot boot 256M 8192 c
FS ext4 / root 0 532480 83

RUN touch /boot/ssh
RUN mkdir /root/.ssh
RUN sed -i 's|#PermitRootLogin prohibit-password|PermitRootLogin yes|g' /etc/ssh/sshd_config
RUN echo 'localhost' > /etc/hostname
RUN sudo apt-get update
RUN sudo apt-get -y install nginx
```
this file translates into:
```
source "arm" "imagefile" {
  image_build_method   = "reuse"
  image_path           = "generated.img"
  image_size           = "2G"
  image_type           = "dos"
  image_chroot_env     = ["PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin"]
  file_checksum_type   = "sha256"
  file_checksum_url    = "https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/2021-05-07-raspios-buster-armhf-lite.zip.sha256"
  file_target_extension = "zip"
  file_urls            = ["https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/2021-05-07-raspios-buster-armhf-lite.zip"]
  image_partitions {
    filesystem           = "vfat"
    mountpoint           = "/boot"
    name                 = "boot"
    size                 = "256M"
    start_sector         = "8192"
    type                 = "c"
  }
  image_partitions {
    filesystem           = "ext4"
    mountpoint           = "/"
    name                 = "root"
    size                 = "0"
    start_sector         = "532480"
    type                 = "83"
  }

}
build {
  sources = ["source.arm.imagefile"]

  provisioner "shell" {
    inline  = ["touch /boot/ssh", "mkdir /root/.ssh", "sed -i 's|#PermitRootLogin prohibit-password|PermitRootLogin yes|g' /etc/ssh/sshd_config", "echo 'localhost' > /etc/hostname", "sudo apt-get update", "sudo apt-get -y install nginx"]
  }
}
```