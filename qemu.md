passthrough
-----------
 - ubuntu-18
```sh
sudo apt-get update
sudo apt-get install qemu-kvm seabios qemu-utils hugepages ovmf
kvm-ok
    INFO: /dev/kvm exists
    KVM acceleration can be used
lsmod |grep kvm
    kvm_intel             253952  0
    kvm                   655360  1 kvm_intel
    
## setting up hugepages
#hugeadm --explain
#sudo vim /etc/default/qemu-kvm
#    KVM_HUGEPAGES=1
#grep "Hugepagesize:" /proc/meminfo
#    Hugepagesize:       2048 kB
##amount_of_RAM / hugepagesize + small_buffer = vm.nr_hugepages
##8192M / 2048k + 7.5% = 4300
#sudo vim /etc/sysctl.conf
#    vm.nr_hugepages = 4300
    
# creating the vm
dd if=/dev/zero of=ubuntu20.img bs=1M seek=24000 count=0
qemu-system-x86_64 -curses
```
[1]: https://davidyat.es/2016/09/08/gpu-passthrough/

```bash
#!/usr/bin/env bash

export DISK=ubuntu-18.qcow2
    printf "change vnc password\n%s\n" qemu | qemu-system-x86_64 \
    -vnc :0,password -monitor stdio \
    -cpu host -enable-kvm -machine accel=kvm \
    -smp 4,sockets=1,cores=2,threads=2 \
    -m 4G \
    -drive file=$DISK,if=virtio \
    -net user,hostfwd=tcp::2222-:22 -net nic \
    -display default,show-cursor=on -vga virtio \
    -usb -device usb-tablet \
    -device vfio-pci,host=05:00.0
```

setting framebuffer resolution 1920x1080
----------------------------------------
```sh
# at guest:
vim /etc/default/grub
    GRUB_GFXMODE=1920x1080x32
    GRUB_GFXPAYLOAD_LINUX=1920x1080x32
update-grub2 # for ubuntu
# at host:
qemu-system-x86_64 -vga vmware
# NOTE: with `-device qxl-vga,vgamem_mb=128` this method does not work
#       and resolution of fb0 is 1024x768 always
```

was helpful for passthrough of usb modem
----------------------------------------
```sh
/etc/init.d/udev stop
echo -n "usb3" > /sys/bus/usb/drivers/usb/unbind
echo -n "usb3" > /sys/bus/usb/drivers/usb/bind
```

increase disk size: `qemu-img resize windows-10.qcow2 64G`
