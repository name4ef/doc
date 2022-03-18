### nvidia optimus
#### solution for work nvidia only
 1. install nvidia driver
 2. get PCI address of nvidia card: `lspci|grep -E "VGA|3D"` (01:00.0 is 1:0:0)
 3. add to /etc/X11/xorg.conf:
```conf
Section "Module"
    Load "modesetting"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID "<pci address here>"
    Option "AllowEmptyInitialConfiguration"
EndSection
```
 4. add to ~/.xinitrc:
```sh
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
##### for kdm
add to /etc/kde/kdm/Xsetup:
```sh
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
xrandr --dpi 96
```
and `killall kdm` to /etc/kde/kdm/Xreset:
[1]: https://wiki.archlinux.org/index.php/NVIDIA_Optimus_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)

### write dirrectory to DVD
```sh
emerge --ask app-cdr/cdrtools # for gentoo
mkisofs -o some.iso some/
cdrecord dev=/dev/cdrom some.iso
# For check:
dd if=/dev/cdrom of=/tmp/cdimg1.iso
md5sum /tmp/cdimg1.iso some.iso
rm /tmp/cdimg1.iso
```
[1]: https://wiki.gentoo.org/wiki/CD/DVD/BD_writing

### resize partition (use with carefully may damage data)
```sh
# reducing the size
umount /dev/sda3
e2fsck -f /dev/sda3
resize2fs /dev/sda3 670G # 670G is new size
fdisk /dev/sda # remove sda3 and create it with new size: 670G
e2fsck -f /dev/sda3
mount /dev/sda3

# increasing the size
umount /dev/sda4
e2fsck -f /dev/sda4
cfdisk /dev/sda
    # remove sda3, sda4;
    # create moved to place of start sda2;
    # create sda4 with new availbse space
reboot
e2fsck -f /dev/sda4
resize2fs /dev/sda4
e2fsck -f /dev/sda4
mount /dev/sda4
```

### simple internet sharing for subnet
```sh
sysctl net/ipv4/ip_forward=1
iptables -t nat -A POSTROUTING -o enp41s0 -j MASQUERADE
```

### clear cache
```sh
sync; echo 1 > /proc/sys/vm/drop_caches # clear PageCache only
sync; echo 2 > /proc/sys/vm/drop_caches # clear dentries and inodes
sync; echo 3 > /proc/sys/vm/drop_caches # clear pagecache, dentries, inodes
```
[1]: https://www.tecmint.com/clear-ram-memory-cache-buffer-and-swap-space-on-linux/

### creating core dump files if program is crushed
```sh
# check limit seze for core dump file:
ulimit -c
# set unlimit or some size (for more security)
ulimit -c unlimited # or <SIZE>
```
[1]: https://stackoverflow.com/questions/6152232/how-to-generate-core-dump-file-in-ubuntu
