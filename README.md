# Create and run a docker container that is able to run snap packages

This script allows you to create docker containers that are able to run and
build snap packages.

**WARNING NOTE**: This will create a container with **security** options **disabled**, this is an unsupported setup, if you have multiple snap packages inside the same container they will be able to break out of the confinement and see each others data and processes. Use this setup to build or test single snap packages but **do not rely on security inside the container**.

```
usage: build.sh [options]

  -c|--containername <name> (default: snappy)
  -i|--imagename <name> (default: snapd)
```

## Examples

Creating a container with defaults (image: snapd, container name: snappy):

```
$ sudo apt install docker.io
$ ./build.sh
```

If you want to create subsequent other containers using the same image, use the --containername option with a subsequent run of the ./build.sh script.

```
$ ./build.sh -c second
$ sudo docker exec second snap list
Name  Version    Rev   Developer  Notes
core  16-2.26.4  2092  canonical  -
$
```

### Installing and running a snap package:

This will install the htop snap and will show the running processes inside the container after connecting the right snap interfaces.

```
$ sudo docker exec snappy snap install htop
htop 2.0.2 from 'maxiberta' installed
$ sudo docker exec snappy snap connect htop:process-control
$ sudo docker exec snappy snap connect htop:system-observe
$ sudo docker exec -ti snappy htop
```

### Building snaps using the snapcraft snap package (using the default "snappy" name):

Install some required debs, install the snapcraft snap package to build snap packages, pull some remote branch and build a snap from it using the snapcraft command.
```
$ sudo docker exec snappy sh -c 'apt -y install git'
$ sudo docker exec snappy snap install snapcraft --edge --classic
$ sudo docker exec snappy sh -c 'git clone https://github.com/ogra1/beaglebone-gadget'
$ sudo docker exec snappy sh -c 'cd beaglebone-gadget; cp cross* snapcraft.yaml; TMPDIR=. snapcraft'
...
./scripts/config_whitelist.txt . 1>&2
Staging uboot
Priming uboot
Snapping 'bbb' |
Snapped bbb_16-0.1_armhf.snap
$
```

### Building an UbuntuCore image for a RaspberryPi3:

Install some debs required to work around a bug in the ubuntu-image classic snap, install ubuntu-image, retrieve the model assertion for a pi3 image using the "snap known" command and build the image using ubuntu-image.
```
$ sudo docker exec snappy sh -c 'apt -y install libparted dosfstools' # work around bug 1694982
Reading package lists... Done
Building dependency tree
Reading state information... Done
...
Setting up libparted2:amd64 (3.2-17) ...
Setting up dosfstools (4.0-2ubuntu1) ...
Processing triggers for libc-bin (2.24-9ubuntu2) ...
$ sudo docker exec snappy snap install ubuntu-image --classic --edge
ubuntu-image (edge) 1.0+snap3 from 'canonical' installed
$ sudo docker exec snappy sh -c "snap known --remote model series=16 model=pi3 brand-id=canonical >pi3.model"
$ sudo docker exec snappy ubuntu-image pi3.model
Fetching core
Fetching pi2-kernel
Fetching pi3
$ sudo docker exec snappy sh -c 'ls *.img'
pi3.img
```
