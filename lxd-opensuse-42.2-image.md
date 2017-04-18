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

When the copy finishes, you can launch a new container
```bash
lxc launch juju/opensuseleap/amd64 imagesuse
```

and access to the bash

```bash
lxc exec imagesuse bash
```
 
### Basic software

We need to install `cloud-init` as Juju uses it to install the Juju agent (`jujud`) in the machines. We also need an editor:

```bash
imagesuse:~ # zypper install cloud-init nano
```

In the `cloud-init` config file we have to comment the setting and updating of the hostname and we also have to add the nocloud data source:

```bash

imagesuse:~ # nano /etc/cloud/cloud.cfg

#Comment set and update hostname
#Add at the bottom of the file

datasource_list: [ NoCloud, ConfigDrive, OpenNebula, DigitalOcean, Azure, AltCloud, OVF, MAAS, GCE, OpenStack, CloudSigma, SmartOS, Ec2, CloudStack, None ]
```

Next step is to clean zypper and ssh keys:

```bash
imagesuse:~ # zypper clean --all

rm -f /etc/ssh/*key*
```
By default, sshd service is disabled and we need to start it:

```bash
imagesuse:~ # systemctl unmask sshd.service
Removed symlink /etc/systemd/system/sshd.service.
imagesuse:~ # systemctl enable sshd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/sshd.service to /usr/lib/systemd/system/sshd.service.
imagesuse:~ # systemctl start sshd.service
```
and we do the same for `cloud-init`, `cloud-config` and `cloud-final` services.

Finally, we exit from our OpenSUSE container

###

lxc publish imagesuse --alias opensuse/tmp --force
lxc delete imagesuse --force

lxc image export local:opensuse/tmp

sudo tar zxvf <fingerprint>.tar.gz

sudo vi metadata.yaml
Add in templates section:
        "/var/lib/cloud/seed/nocloud-net/meta-data": {
            "template": "cloud-init-meta.tpl",
            "when": [
                "create",
                "copy"
            ]
        },
        "/var/lib/cloud/seed/nocloud-net/network-data": {
            "template": "cloud-init-network.tpl",
            "when": [
                "create",
                "copy"
            ]
        },
        "/var/lib/cloud/seed/nocloud-net/user-data": {
            "template": "cloud-init-user.tpl",
            "when": [
                "create",
                "copy"
            ]
        },
        "/var/lib/cloud/seed/nocloud-net/vendor-data": {
            "template": "cloud-init-vendor.tpl",
            "when": [
                "create",
                "copy"
            ]
        }




sudo vi templates/cloud-init-meta.tpl

#cloud-config
instance-id: {{ container.name }}
local-hostname: {{ container.name }}
{{ config_get("user.meta-data", "") }}

sudo vi templates/cloud-init-network.tpl

{% if config_get("user.network-config", "") == "" %}version: 1
config:
    - type: physical
      name: eth0
      subnets:
          - type: {% if config_get("user.network_mode", "") == "link-local" %}manual{% else %}dhcp{% endif %}
            control: auto{% else %}{{ config_get("user.network-config", "") }}{% endif %}

sudo vi templates/cloud-init-user.tpl

{{ config_get("user.user-data", properties.default) }}

sudo vi templates/cloud-init-vendor.tpl

{{ config_get("user.vendor-data", properties.default) }}


lxc image import opensuse_lxd.tar.gz

lxc image delete juju/opensuseleap/amd64
ubuntu@canyoles:~/images/opensuse$ lxc image delete opensuse/tmp

lxc image alias create juju/opensuseleap/amd64 <new_fingerprint>

