# Cloud Init files

These are a bunch of cloud-init files I've been using to quickly set up Ubuntu LXD containers/VMs and Multipass VM computers in my home lab. 

For info about how to set up LXD and multipass, see: 

LXD docs: [linuxcontainers.org](https://linuxcontainers.org/) and [lxd.readthedocs.io](https://lxd.readthedocs.io/)

multipass docs: [multipass.run](https://multipass.run/)

Mostly used with Ubuntu 20.04 cloud images from the `ubuntu` image source. 
The `xrdp`cloud init files create desktop containers/VMs that are accessible with RDP after the installation. These are only known to work on Ubuntu 20.04.

The minimal cloud-init file should work with pretty much any cloud-init enabled image.

Remove the Landscape registration stuff from the files if there is no [Landscape](https://landscape.canonical.com/) server in your network. 

## Usage examples

Launch LXD container
```
$ lxc launch ubuntu:20.04 u2004vm1 \
  --config=user.user-data="$( cat lxd-profile-ubuntu.yaml )" \
  [--config=limits.memory=4GB] [--vm]
```

Set CPU limit to 2 cores worth of CPU usage on a container (over all cores)
```
$ lxc config set <instance> limits.cpu.allowance=100ms/50ms
```

Instance sizes can also be specified as described here: [instance-types](https://lxd.readthedocs.io/en/latest/instances/#instance-types)

Set a default size for new created containers and VMs
```
$ lxc storage set <pool> volume.size=25GB 
```

For the following settings first create an override setting for the default profile, after that `set` can be used.

Disk limits
```
$ lxc config device override <instance> root size=10GB
$ lxc config device set <instance> root size=15GB
```


Select network bridge for an instance
```
$ lxc config device override <instance> eth0 parent=bridge-vlan10
$ lxc config device set <instance> eth0 parent=bridge-vlan10
```


