# mlc-ng

This aims to be a simplified version of the [original
mlc](https://github.com/axn/mlc) by @axn. OpenWrt can run inside LXC containers
and therefore and are even supported via
[distrobuilder](https://github.com/lxc/distrobuilder).

Likely this is just a collection of scripts, not a real replacement of *mlc*.

## Use `distrobuilder` to generate metadata and rootfs for LXC

* Install `golang`
* Install `distrobuilder` via `go build github.com/lxc/distrobuilder/distrobuilder
* `sudo $GOPATH/bin/distrobuilder build-lxc mlc.yml`

Now the current folder should contain `meta.tar.xz` and `rootfs.tar.xz`.

## Create `mlc-base` image

This image is the base for all further nodes. Via `overlayfs` the storage usage
is minimized.

* `sudo lxc-create -t local -B overlayfs mlc-base -- -m meta.tar.xz -f rootfs.tar.xz`

## Create `mlc-N` nodes

These nodes run actually the routing protocols, the name schema is `mlc-N`.

```sh
for i in {1..5}; do
    sudo lxc-clone -n mlc-base -N mlc-$i -B overlayfs -s
done
```

## Start your containers and run command

```sh
for i in {1..5}; do
    sudo lxc-start mlc-$i
    sudo lxc-attach mlc-$i -- bmx7 -f0 dev=eth1.11
done
