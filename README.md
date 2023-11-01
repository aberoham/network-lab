Various network related labs
============================

I use those labs to test various stuff. Most of them are tailored to
my need. The most recent ones are more likely to work than the older
ones.

They are expected to run without being root on top of an up-to-date
Debian sid. Some of them are using User-Mode-Linux, some other are
using KVM.

`lab-generic` should always contain the latest iteration of the lab
and be used as a base for other labs.

For labs exposing some serial console, you can use `socat` to connect:

```
socat $(tty),raw,echo=0 UNIX-CONNECT:/tmp/tmp.N3BTQ07qSG/vm-IOS1-serial2.pipe
```

Previously, labs were self-contained. This was done to avoid any
breakage when introducing "new features". However, this didn't work as
expected and labs become broken because of external changes (kernel
changes, systemd changes, etc.). Therefore, new labs are now sourcing
some common files (in `common/`). This means that older labs may broke
due to more recent changes. In this case, get the latest commit for a
lab (`git log --oneline -1 lab-generic` for example) and get a
checkout for it (`git checkout 22f22864632a`).

This lab doesn't work on Ubuntu because they are missing VDE support
in QEMU. The easiest way is to use `schroot` to work on a Debian
Bookworm:

```
#!/bin/sh
sudo debootstrap bookworm bookworm
cat <<EOF | sudo tee /etc/schroot/chroot.d/lab
[lab]
type=directory
description=Network lab
directory=$PWD/bookworm
users=$USER
shell=$SHELL
profile=default
EOF
sudo schroot -c lab apt install --no-install-recommends \
    qemu-system-x86 sudo tmux busybox ssh vde2 python3 binutils \
    dosfstools mtools ansible jq linux-image-amd64 nginx rsyslog
```

You may need to uncomment `/dev/shm` line in
`/etc/schroot/default/fstab`. You also have to add yourself to `kvm`
group (`sudo adduser $USER kvm`). Then, run:

    schroot -c lab ./setup /boot/$(schroot -c lab ls /boot | grep vmlinuz | head -1)

The `./setup` script takes a Linux kernel as first argument. Have a
look at
[eudyptula-boot](https://github.com/vincentbernat/eudyptula-boot/blob/master/minimal-configuration)'s
`minimal-configuration` script to compile a kernel that should work
with the lab as well.

License
-------

All the labs are distributed under the ISC license:

> Permission to use, copy, modify, and/or distribute this software for any
> purpose with or without fee is hereby granted, provided that the above
> copyright notice and this permission notice appear in all copies.
>
> THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
> WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
> MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
> ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
> WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
> ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
> OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

Other tools
-----------

There exist many other tools to run network labs that may not be as
hacky as this one:

 - [CORE](http://www.nrl.navy.mil/itd/ncs/products/core). It uses
   Linux network namespaces and provides a GUI tool. This is a very
   good tool. Please, have a look at it. It doesn't use disk images
   and the whole lab configuration fits into a single file that's easy
   to share. Integration with Quagga or BIRD is very good.

 - [GNS3](http://www.gns3.com/). It uses virtual machines and
   emulators to build the network. It also comes with a GUI tool. You
   can emulate Cisco, Juniper, Arista and other brands network
   equipments. However, it relies heavily on disk images for anything
   else than Cisco devices and it makes it harder to share your work
   on GitHub.

You will find a more comprehensive list (with tests) on [Brian Linkletter's blog](http://www.brianlinkletter.com/open-source-network-simulators/).
