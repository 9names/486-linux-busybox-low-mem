# 486-linux-busybox-low-mem
Experiments in running Linux on a low RAM 486, simulated with QEMU

### To recreate these experiments as tested:

Grab a copy of the i486 musl cross compiler
https://musl.cc/i486-linux-musl-cross.tgz

Busybox
https://www.busybox.net/downloads/busybox-1.32.0.tar.bz2

Linux kernel
https://git.kernel.org/torvalds/t/linux-5.8-rc1.tar.gz

qemu-system-i386
Download/install using your distro's package manager

qemu-static-i386 (only if you want to test your busybox/other user binaries first)
Download/install using your distro's package manager


Extract the musl compiler somewhere convenient, then add i486-linux-musl-cross/bin to your path
eg
```
export PATH=$PATH:/opt/i486-linux-musl-cross/bin
```

Extract all the sources, clone/download this repo somewhere (i'll refer to it as being in ~/Downloads)

# Building Busybox
cd to the busybox folder, then copy the Busybox config into it
```
cd ~/Downloads/busybox-1.32.0
cp ~/Downloads/486-linux-busybox-low-mem/busybox_config/.config ./

CROSS_COMPILE=i486-linux-musl- make menuconfig
CROSS_COMPILE=i486-linux-musl- make
```

# Building Linux
```
cd ~/Downloads/linux-5.8-rc1
cp ~/Downloads/486-linux-busybox-low-mem/linux_config/.config ./
cp ~/Downloads/486-linux-busybox-low-mem/rootfs ./ -R
cp ~/Downloads/busybox-1.32.0/busybox initramfs
CROSS_COMPILE=i486-linux-musl- make menuconfig
CROSS_COMPILE=i486-linux-musl- make
```

# Running your newly built system
From within the linux folder
```
qemu-system-i386 -cpu 486 -kernel arch/x86/boot/bzImage -m 8M -append 'console=ttyS0,115200' -serial mon:stdio -nographic
```

or, if you're looking to see how little ram you can get away with
```
qemu-system-i386 -cpu 486 -kernel arch/x86/boot/bzImage -m 3666K -append 'console=ttyS0,115200' -serial mon:stdio -nographic
```

You can also boot qemu in graphical mode, but I find that less useful
```
qemu-system-i386 -cpu 486 -kernel arch/x86/boot/bzImage -m 8M
```

# Notes:
Compressing the initramfs requires more memory - since it's so small anyway, I recommend not bothering

Compressing the kernel does not affect runtime memory usage at all - if there's sufficient memory for the uncompressed kernel and the decompression buffers, then you can use any compression method you like. I did not run into issues with the kernel sizes I was getting with a minimal config. XZ should give best compression, and therefore best boot speeds from slow media like floppy disk.

ProcFS as configured here uses about 100K of ram.

printk support (currently disabled) uses about 200K of ram

###Useful commands for checking memory usage:
```
cat /proc/meminfo
cat /proc/iomem
```