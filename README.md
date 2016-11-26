# How to use

* Disk size is adjustable in the `packer-template.json`:
```
      "disk_size": 400000,
```

* Build box with packer:
```
$ packer build packer-template.json
```

* Add box to Vagrant
```
$ vagrant box add --name dcos-centos-virtualbox-400g dcos-centos-virtualbox.box
```

* Use box in the `dcos-vagrant` project. Edit the `VagrantConfig.yaml`
```
a1:
  ip: 192.168.65.111
  cpus: 4
  memory: 6144
  memory-reserved: 512
  type: agent-private
  box: dcos-centos-virtualbox-400g
  box-version: =0
```

* That's it. Now vagrant up!

# DC/OS Vagrant Box

Vagrant box builder for [dcos-vagrant](https://github.com/mesosphere/dcos-vagrant) using [Packer](https://www.packer.io/).

The box produced by this builder **does not include DC/OS**, just it's dependencies.

Pre-provisioning this box front-loads internet access requirements, flakiness, and slowness resulting from third party package managemers and installers. This makes DC/OS installation faster and more reliable at the cost of potentially having slightly outdated dependencies.

**Issue tracking is on the [DCOS JIRA](https://dcosjira.atlassian.net/projects/VAGRANT).**


## Provisioning Summary

- Ansible (used for provisioning)
  - Python 2.7+
  - Pip 8.1+
  - Ansible 2.1+
- CentOS
  - Latest CentOS 7+ kernel & packages
  - Enable Overlay file system
  - Disable kdump
- DC/OS Node
  - Docker 1.11 w/ OverlayFS
  - curl, bash, ping, tar, xz, unzip, ipset
  - Disable firewalld
  - Disable IPv6
  - Cache docker images: nginx, zookeeper, registry
- Debug
  - jq
  - probe
  - net-utils
- Vagrant
  - Default Vagrant SSH key
  - Configure SSH
- VirtualBox
  - Guest Additions
  - Reset network interface config
  - Remove packages to reduce image size
- Zero out free space to improve image compression


## Build

The DC/OS Vagrant Box is normally built in CI, on demand, but can be built manually as well.

Note that because the build process uses internet repositories with unversioned requirements, it's not **exactly** reproducible. Each built box may be slightly different than the last, but installations using the same box should be exactly the same.


### Build Requirements

- [Packer](https://www.packer.io/) - tool for OS image building
- [VirtualBox](https://www.virtualbox.org/) (>= 4.3)
- [Git](https://git-scm.com/) - for updating the Vagrant Box Catalog Metadata
- [Docker](https://www.docker.com/) - for running various other dependencies
- [OpenSSL](https://www.openssl.org/) - for generating box checksums


### Build Pipeline

1. Checkout - Clone or pull this repo
1. Clean - Delete any left over box files
1. Build - Use Packer to install the OS, runs all the build scripts, and export a compressed box file (~700MB)
1. Upload - Upload the new box to S3
1. Add - Add the new box version to the Vagrant Box Catalog Metadata (metadata.json)
1. Publish - Upload the updated Vagrant Box Catalog Metadata to S3

Note that because the build process uses internet repositories with unversioned requirements, it's not **exactly** reproducible. Each built box may be slightly different than the last, but installations using the same box should be exactly the same.


### Manual Build

Use the following commands to build a dcos-centos-virtualbox box:

```bash
cd <dcos-vagrant-box>
packer build packer-template.json
```


## Test

New boxes can be tested with either the Vagrantfile in this repo or (preferably) with [dcos-vagrant](https://github.com/mesosphere/dcos-vagrant).


### Test Requirements

- [Vagrant](https://www.vagrantup.com/) (>= 1.8.1)
- [VirtualBox](https://www.virtualbox.org/) (>= 4.3)
  - [VBGuest Plugin](https://github.com/dotless-de/vagrant-vbguest)

#### Install Vagrant VBGuest Plugin

The [VBGuest Plugin](https://github.com/dotless-de/vagrant-vbguest) manages automatically installing VirtualBox Guest Additions appropriate to your local Vagrant version on each new VirtualBox VM as it is created.

```bash
vagrant plugin install vagrant-vbguest
```

This allows the pre-built vagrant box image to work on multiple (and future) versions of VirtualBox.

### Test Local Box

To test a new local box (e.g. after `ci/build_release.sh`), add the box to vagrant and set the `DCOS_BOX_VERSION` environment variable:

```
cd <dcos-vagrant-box>
vagrant box add mesosphere/dcos-centos-virtualbox dcos-centos-virtualbox.box
export DCOS_BOX_VERSION=0
vagrant up
```

Or with [dcos-vagrant](https://github.com/mesosphere/dcos-vagrant):

```
cd <dcos-vagrant>
vagrant box add mesosphere/dcos-centos-virtualbox dcos-centos-virtualbox.box
export DCOS_BOX_VERSION=0
vagrant up [vms...]
```

To revert back to the remote Vagrant Box Catalog:

```
unset DCOS_BOX_VERSION
vagrant box remove mesosphere/dcos-centos-virtualbox --box-version=0
```

### Test Local Catalog

To test a local Vagrant Box Catalog (e.g. after `ci/update_catalog.sh`), set the `DCOS_BOX_URL` environment variable:

```
cd <dcos-vagrant-box>
export DCOS_BOX_URL=file://~/workspace/dcos-vagrant-box/metadata.json
vagrant up
```

Or with [dcos-vagrant](https://github.com/mesosphere/dcos-vagrant):

```
cd <dcos-vagrant>
export DCOS_BOX_URL=file://~/workspace/dcos-vagrant-box/metadata.json
vagrant up [vms...]
```

To revert back to the remote Vagrant Box Catalog:

```
unset DCOS_BOX_URL
echo -n "https://downloads.dcos.io/dcos-vagrant/metadata.json" > ~/.vagrant.d/boxes/mesosphere-VAGRANTSLASH-dcos-centos-virtualbox/metadata_url
```


# License

Copyright 2016 Mesosphere, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this repository except in compliance with the License.

The contents of this repository are solely licensed under the terms described in the [LICENSE file](./LICENSE) included in this repository.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
