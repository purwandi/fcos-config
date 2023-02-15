# Setup MD

## Prerequisites

To build your own distribution based on Fedora CoreOS, you will need a Linux system 
(Fedora 33 has been used when writing this article) with ostree, git, rclone and podman.

```
sudo dnf install ostree git rclone podman
```

You will also use cosa, the CoreOS Assembler. Cosa is packaged as a container 
image and podman is used to run it. A shell wrapper to run cosa is provided but 
it has some drawbacks, such as not working on ZSH.

Instead, create a shell script in /usr/local/bin that runs cosa:

```
cat > /usr/local/bin/cosa <<"EOF"
#!/bin/bash

podman run --rm -ti --security-opt label=disable --privileged --user=root                        \
           -v ${PWD}:/srv/ --device /dev/kvm --device /dev/fuse                                  \
           --tmpfs /tmp -v /var/tmp:/var/tmp --name cosa                                         \
           ${COREOS_ASSEMBLER_CONFIG_GIT:+-v $COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}   \
           ${COREOS_ASSEMBLER_GIT:+-v $COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}  \
           ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS}                                            \
           ${COREOS_ASSEMBLER_CONTAINER:-quay.io/coreos-assembler/coreos-assembler:latest} "$@"
EOF
chmod +x /usr/local/bin/cosa
```

## Test your build chain

Cosa requires a dedicated build directory where it will store cached RPMs, built 
ostrees and installation images.

Create a dedicated build directory for cosa.

```
mkdir -p $HOME/tmp/fedora-coreos
```

Move to the build directory and initialize it with the Fedora CoreOS sources.

```
cd $HOME/tmp/fedora-coreos
cosa init --branch stable git@github.com:purwandi/fcos-config.git
```

Then, cosa fetch will fetch the needed RPMs and meta-data.

```
cosa fetch
```

And cosa build will build the ostree and the qemu image.

```
cosa build
```

Finally, run cosa buildextend commands to generate the metal images + the Live ISO image.

```
cosa buildextend-vmware
```