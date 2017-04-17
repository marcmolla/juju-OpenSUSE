# OpenSUSE Leap image for Juju and LXD

## Introduction

This instructions are for creating a LXD OpenSUSE image for Juju, based on
the image from https://images.linuxcontainers.org .

## Image creation

### Image copy from public repository

You can copy a LXD ready OpenSUSE image using `lxc` client:

```bash
lxc image copy images:opensuse/42.2 local: --alias juju/opensuseleap/amd64
```
 
