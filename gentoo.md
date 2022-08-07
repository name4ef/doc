### install amd64
https://mirror.yandex.ru/gentoo-distfiles/releases/amd64/autobuilds/current-install-amd64-minimal/
#### up network and run sshd
```sh
wpa_passphrase <SSID> >> /etc/wpa_supplicant.conf
wpa_supplicant -D wext -i wlp1s0 -c /etc/wpa_supplicant.conf -B
ntpd -q -g # TODO test with ntp-client only
/etc/init.d/sshd start
passwd # set password for root
```
#### chroot && poweroff
```sh
export _ROOT=/dev/sda3 &&
mount $_ROOT /mnt/gentoo && cd /mnt/gentoo &&
mount -t proc /proc proc/ && mount -R /sys sys/ && mount -R /dev dev/ &&
chroot . /bin/bash &&
cd && umount -l /mnt/gentoo/dev{/shm,/pts,} && umount -R /mnt/gentoo &&
poweroff
```
#### basic action
 - create efi, swap and root partitions over cfdisk (gpt)
```sh
 # WARNING: partitions of sda will be overwrited
export  _EFI=/dev/sda1 &&
export _SWAP=/dev/sda2 &&
export _ROOT=/dev/sda3 &&
mkfs.vfat $_EFI &&
mkswap $_SWAP && swapon $_SWAP &&
mkfs.ext4 $_ROOT && mount $_ROOT /mnt/gentoo && cd /mnt/gentoo &&
links https://mirror.yandex.ru/gentoo-distfiles/releases/amd64/autobuilds/
 # or links https://www.gentoo.org/downloads/mirrors/
 #   select mirror
 #   go to *releases/amd64/autobuilds/current-stage3-amd64* directory
```
- downloads:
    - *stage3-amd64-<YYYYMMDDTHHMMSSZ>.tar.xz*
    - *stage3-amd64-<YYYYMMDDTHHMMSSZ>.tar.xz.DIGESTS*
```sh
sha512sum --ignore-missing -c *.DIGESTS
tar xpf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner &&
cat << EOF >> etc/portage/make.conf &&
GENTOO_MIRRORS="http://mirror.yandex.ru/gentoo-distfiles/" # mirrorselect -i -o >> etc/portage/make.conf
EOF
mkdir etc/portage/repos.conf &&
cp usr/share/portage/config/repos.conf etc/portage/repos.conf/gentoo.conf &&
cp -L /etc/resolv.conf etc/ &&
mount -t proc /proc proc/ && mount -R /sys sys/ && mount -R /dev dev/ &&
chroot . /bin/bash
```
```sh
source /etc/profile && export PS1="(chroot) ${PS1}" &&
emerge-webrsync &&
echo "Asia/Tomsk" > /etc/timezone && emerge --config sys-libs/timezone-data &&
emerge net-misc/ntp && ntpd -q -g && # TODO test with ntp-client only
echo "export LANG='en_US.UTF8'" >> /etc/profile.env &&
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen &&
locale-gen &&
eselect locale list
```
 - eselect locale set 4 # (en_US.utf8)
```sh
env-update && source /etc/profile && export PS1="(chroot) ${PS1}" &&
echo "sys-kernel/linux-firmware linux-fw-redistributable no-source-code" >> /etc/portage/package.license &&
echo "app-editors/vim python" >> /etc/portage/package.use/app-editors &&
emerge \
    sys-kernel/gentoo-sources \
    sys-kernel/genkernel \
    sys-kernel/linux-firmware \
    app-admin/sysklogd \
    sys-fs/e2fsprogs \
    net-misc/dhcpcd \
    sys-apps/ethtool \
    app-editors/vim \
    dev-util/ctags \
    app-misc/task \
    app-misc/tmux \
    dev-python/six \
    dev-python/pip \
    app-text/tree \
    sys-process/htop \
    dev-vcs/git \
    media-fonts/hack \
    app-shells/gentoo-zsh-completions \
    &&
genkernel all &&
cat << EOF >> /etc/fstab
$_EFI /boot/efi vfat noauto  0 1
$_SWAP none      swap sw      0 0
$_ROOT /         ext4 noatime 0 1
EOF
rc-update add sysklogd default &&
rc-update add sshd default &&
rc-update add ntpd && # TODO test with ntp-client only
echo -e "\nPlease set root password:" &&
passwd
```
#### boot over lilo
```sh
emerge sys-boot/lilo &&
liloconfig && vim /etc/lilo.conf && lilo
```
May be useful for qemu:
```lilo
lba32
compact
boot = /dev/vda
map = /boot/map
install = menu
menu-scheme = Wb:Yr:Wb:Wb
prompt
timeout = 00
vga = normal
disk = /dev/vda bios=0x80 max-partitions=7

image = /boot/vmlinuz-5.4.60-gentoo-x86_64
    initrd = /boot/initramfs-5.4.60-gentoo-x86_64.img
    label = "Linux"
    root = /dev/vda3
    append="nomodeset"
    read-only
```
#### boot over grub (UEFI)
```sh
emerge sys-boot/grub:2 &&
mkdir -p /boot/efi && mount /boot/efi &&
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf &&
grub-install --target=x86_64-efi --efi-directory=/boot/efi &&
grub-mkconfig -o /boot/grub/grub.cfg &&
umount /boot/efi
```
#### configure wirelss network
 - emerge net-wireless/wpa_supplicant
 - wpa_passphrase <SSID> >> /etc/wpa_supplicant/wpa_supplicant.conf
 - vim /etc/conf.d/wpa_supplicant
    wpa_supplicant_args="-D wext -i wlp1s0 -c /etc/wpa_supplicant/wpa_supplicant.conf -B"
 - rc-update add wpa_supplicant # /etc/init.d/wpa_supplicant start
#### final steps
```sh
export _USER=user1 &&
useradd -d /home/$_USER -G users,wheel -m $_USER &&
passwd $_USER &&
vim /etc/{conf.d/hostname,hosts} && # set hostname
exit

cd && umount -l /mnt/gentoo/dev{/shm,/pts,} && umount -R /mnt/gentoo &&
reboot
```
[1]: https://www.gentoo.org/get-started/
[2]: https://wiki.gentoo.org/wiki/Handbook:AMD64

### distcc for emerge
 1. ssh root@calculate
     1. emerge --ask sys-devel/distcc
     2. vim /etc/conf.d/distccd
         ```sh
         # add some like this:
         DISTCCD_OPTS="--port 3632 -N 15
         --log-level notice --log-file /var/log/distccd.log
         --allow 192.168.0.4 --allow 192.168.0.5"
         ```
     3. rc-update add distccd default
     4. rc-service distccd start
 2. distcc-config --set-hosts "localhost,cpp,lzo calculate,cpp,lzo"
 3. vim /etc/portage/make.conf
     ```conf
     # 1 remote hosts with 6 cores each = 6 cores remote
     # 1 local host with 8 cores = 8 cores local
     # total number of cores is 14, so N = 2*14+1 and M=8
     MAKEOPTS="-j29 -l8"
     FEATURES="distcc"
     ```
[1]: https://wiki.gentoo.org/wiki/Distcc

### can't ping under regular user (ping: socket: Operation not permitted)
```sh
emerge -C net-misc/iputils
emerge net-misc/iputils
```

### write iso image of windows distribution to usb flash
```sh
emerge -av woeusb
woeusb --device Win10_1909_Russian_x64.iso --target-filesystem NTFS /dev/sdc
```

### set hostname
```sh
vim /etc/{conf.d/hostname,hosts}
hostname <hostname>
```

### usage only needed network interface (ex. wired only)
```sh
vim /etc/conf.d/net
    config_enp2s0="dhcp"
    # or for static:
    #config_enp37s0="10.0.0.109 netmask 255.255.255.0 brd 192.168.0.255"
    #routes_enp37s0="default via 10.0.0.1"
cd /etc/init.d
ln -s net.lo net.enp2s0
rc-update add net.enp2s0 default
```
[1]: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System#Automatically_start_networking_at_boot

### making bootable gentoo LiveUSB
```sh
emerge --ask sys-fs/dosfstools sys-boot/syslinux
cfdisk /dev/sdb
    gpt, 1G linux fs
mkfs.fat -F 16 /dev/sdb1
dd if=/usr/share/syslinux/mbr.bin of=/dev/sdb
mkdir -p /mnt/{iso,usb}
mount -o loop,ro -t iso9660 /path/to/isofile.iso /mnt/iso
mount -t vfat /dev/sdb1 /mnt/usb

 # Copying and moving files:
cp -r /mnt/iso/* /mnt/usb
mv /mnt/usb/isolinux/* /mnt/usb
mv /mnt/usb/isolinux.cfg /mnt/usb/syslinux.cfg
rm -rf /mnt/usb/isolinux*
mv /mnt/usb/memtest86 /mnt/usb/memtest

sed -i \
    -e "s:cdroot:cdroot slowusb:" \
    -e "s:kernel memtest86:kernel memtest:" /mnt/usb/syslinux.cfg
umount /mnt/{iso,usb}
syslinux /dev/sdb1 # install the syslinux bootloader on the USB drive
```
[1]: https://wiki.gentoo.org/wiki/LiveUSB/Guide

### masked by: ~amd64 keyword
```sh
echo 'ACCEPT_KEYWORDS="~amd64"' >> /etc/portage/make.conf
```

### reset tvheadend password
*Will be reset all settings besides password*
```sh
/etc/init.d/tvheadend stop
rm -rf /var/lib/tvheadend
/etc/init.d/tvheadend start
```

### enable wake on lan
```sh
emerge -av sys-apps/ethtool

 # Checking and set current WOL status
 #   d - disable
 #   g - wake of MagicPackettm
ethtool enp4s0 | grep -i wake
    Supports Wake-on: pumbg
    Wake-on: d
ethtool -s enp4s0 wol g

vim /etc/conf.d/net
    ethtool_change_enp4s0="wol g"
cd /etc/init.d; ln -s net.lo net.enp4s0
rc-update add net.enp4s0 default
openrc
```
[1]: https://wiki.gentoo.org/wiki/Power_management/Ethernet
[2]: https://wiki.gentoo.org/wiki/Handbook:X86/Networking/Introduction

### install xserver and dwm and so on
```sh
echo 'USE="consolekit -elogind -systemd"' >> /etc/portage/make.conf
emerge -a xorg-server xdm dwm dmenu x11-terms/st x11-apps/setxkbmap

emerge --ask --verbose --depclean x11-wm/dwm
qlist -IC 'x11-base/*' 'x11-drivers/*'
quse elogind
```

### packages
#### remove
```sh
emerge --deselect dev-python/six
emerge -cvp
emerge -c
 # emerge -avC sys-apps/sysvinit # TODO read about -C
```
[1]: https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet#Package_removal
#### upgrade
```sh
unset LD_LIBRARY_PATH CCACHE_DIR
emerge --ask --update --deep --newuse @world
emerge --ask --depclean
```
##### helpful when conflict of update
```sh
qdepends -QF '%{CAT}/%{PN}:%{SLOT}' ^dev-qt/qtcore-5.15.5-r1:5/5.15.5
 # Note: The ^ is significant, as is using the package in CAT/PN:SLOT/SUBSLOT syntax.
emerge --ignore-default-opts -va1 $( \
    qdepends -QF '%{CAT}/%{PN}:%{SLOT}' '^dev-qt/qtcore-5.15.5-r1:5/5.15.5' \
    | cut -d ":" -f 1,2 )

emerge --update --newuse --deep --with-bdeps=y @world
```
[1]: https://wiki.gentoo.org/wiki/Upgrading_Gentoo
#### search all version of package
```sh
equery list -po vim
```

### qemu
```sh
echo "app-emulation/qemu usb" >> /etc/portage/package.use
emerge -a app-emulation/qemu sys-apps/usbutils
usermod -aG kvm usb user1

export DISK=ubuntu-18.qcow2
qemu-system-x86_64 \
    -cpu host -enable-kvm -machine accel=kvm \
    -smp 4,sockets=1,cores=2,threads=2 \
    -m 4G \
    -drive file=$DISK,if=virtio \
    -net user,hostfwd=tcp::2222-:22 -net nic \
    -display default,show-cursor=on -vga virtio \
    -usb -device usb-tablet \
    -usb -device usb-ehci -device usb-host,vendorid=0x0572,productid=0xc68a
```
#### pci-passthrough
```sh
lscpu | grep -E "vmx|svm" # VT-x or AMD-V must be supported by the processor
 # Go into BIOS (EFI) settings and turn on VT-d and IOMMU support
 # Note: Some EFI doesn't have IOMMU configuration settings
vim /etc/default/grub
    # Intel (ex. TODO)
    GRUB_CMDLINE_LINUX="iommu=pt intel_iommu=on pcie_acs_override=downstream,multifunction"
    # AMD (ex. AMD Ryzen 5 3400G with Radeon Vega Graphics)
    GRUB_CMDLINE_LINUX="iommu=pt amd_iommu=on pcie_acs_override=downstream,multifunction"
grub-mkconfig -o /boot/grub/grub.cfg
reboot

dmesg | grep 'IOMMU enabled'
for d in /sys/kernel/iommu_groups/*/devices/*; do
    n=${d#*/iommu_groups/*}; n=${n%%/*}
    printf 'IOMMU Group %s ' "$n"
    lspci -nns "${d##*/}"
done # In IOMMU group 13 wireless networking controller only - it's fine; or apply ACS patch
lspci -nn | grep Wireless
    03:00.0 Network controller [0280]: Qualcomm Atheros AR9485 Wireless Network Adapter [168c:0032] (rev 01)
vim /etc/modprobe.d/vfio.conf
    options vfio-pci ids=168c:0032
reboot

echo '0000:03:00.0' > /sys/bus/pci/devices/0000\:03\:00.0/driver/unbind
modprobe vfio-pci

export DISK=ubuntu-18.qcow2
qemu-system-x86_64 \
    -cpu host -enable-kvm -machine accel=kvm \
    -smp 4,sockets=1,cores=2,threads=2 \
    -m 4G \
    -drive file=$DISK,if=virtio \
    -net user,hostfwd=tcp::2222-:22 -net nic \
    -display default,show-cursor=on -vga virtio \
    -usb -device usb-tablet \
    -device vfio-pci,host=03:00.0
```
[1]: https://wiki.gentoo.org/wiki/GPU_passthrough_with_libvirt_qemu_kvm
[2]: https://davidyat.es/2016/09/08/gpu-passthrough/
[3]: https://www.kernel.org/doc/Documentation/vfio.txt
[4]: https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Ensuring_that_the_groups_are_valid
##### applying ACS patch
```sh
git clone https://github.com/feniksa/gentoo_ACS_override_patch.git /etc/portage/patches
emerge -av gentoo-sources
genkernel kernel
```
#### boot trought uefi
```sh
emerge sys-firmware/edk2-ovmf
qemu-system-x86_64 \
    -bios /usr/share/edk2-ovmf/OVMF_CODE.fd \
    -cdrom /usr/share/edk2-ovmf/UefiShell.iso
```
#### set framebuffer resolution (1920x1080)
```sh
emerge -a x11-drivers/xf86-video-qxl
```
#### Unbind VTconsoles
```sh
echo 0 > /sys/class/vtconsole/vtcon0/bind
echo 0 > /sys/class/vtconsole/vtcon1/bind
```
#### Dumping rom
```sh
echo '0000:05:00.0' > /sys/bus/pci/drivers/vfio-pci/unbind
echo 1 > /sys/bus/pci/devices/0000\:05\:00.0/rom
cat /sys/bus/pci/devices/0000\:05\:00.0/rom > vega.bin
echo 0 > /sys/bus/pci/devices/0000\:05\:00.0/rom
echo '0000:05:00.0' > /sys/bus/pci/drivers/vfio-pci/bind
```
#### use cifs for share directory
```sh
emerge -av net-fs/samba
qemu-system-x86_64 \
    -device e1000,netdev=net0 -netdev user,id=net0,smb=/home/user1/soft \
 # \\10.0.2.4\qemu
```

### grub automaticly detect other OS
```sh
emerge --ask --newuse sys-boot/os-prober
grub-mkconfig -o /tmp/grub.cfg # place for grub config
```

### show file of package
```sh
emerge app-portage/gentoolkit
equery files --tree sys-firmware/edk2-ovmf
```
[1]: https://wiki.gentoo.org/wiki/Equery

### play file through gst-launch to fbdev
```sh
emerge \
    media-libs/gstreamer \
    media-libs/gst-plugins-good \
    media-plugins/gst-plugins-libav
EXTRA_ECONF="--enable-fbdev" emerge media-libs/gst-plugins-bad
usermod -aG video user1
gst-launch-1.0 filesrc location=video03.mp4 \
    ! qtdemux \
    ! h264parse \
    ! avdec_h264 \
    ! videoconvert \
    ! deinterlace \
    ! fbdevsink
```
[1]: https://lists.archive.carbon60.com/gentoo/user/166724
[2]: https://wiki.gentoo.org/wiki/Knowledge_Base:Overriding_environment_variables_per_package

### monitorix and so on
```sh
emerge -av \
    www-misc/monitorix \
    app-admin/hddtemp \
    sys-apps/lm-sensors \
    sys-apps/smartmontools
rc-update add monitorix default
openrc
```

### connect trackpad via bluetooth
```sh
emerge -av net-wireless/bluez
vim /etc/bluetooth/input.conf
    UserspaceHID=true
rc-update add bluetooth default
openrc

bluetoothctl
    list
    scan on
    connect <mac>
    trust <mac>
    scan off
```
[1]: https://wiki.gentoo.org/wiki/Bluetooth/pl
[1]: https://wiki.gentoo.org/wiki/Bluetooth_input_devices

### run baical server (AppImage application)
```sh
emerge -av sys-fs/fuse:0

DISPLAY=:0 ./BaicalServer.x64.v5.3.1.AppImage
```
[1]: http://baical.net/index.html
[2]: https://wiki.gentoo.org/wiki/AppImage

### installation and configure of pulseaudio
```sh
vim /etc/portage/make.conf
    USE="pulseaudio"
    ACCEPT_LICENSE="NPSL no-source-code linux-fw-redistributable"
emerge --ask --changed-use --deep --update --newuse @world
```
[1]: https://wiki.gentoo.org/wiki/PulseAudio

### install and configure proprietary nvidia drivers (v455.38)
```sh
eselect kernel list
eselect kernel set 4 # it was 5.9.11
genkernel all
grub-mkconfig -o /boot/grub/grub.cfg

vim /etc/portage/make.conf
    ACCEPT_LICENSE="NVIDIA-r2"
emerge -av x11-drivers/nvidia-drivers

vim /etc/X11/xorg.conf.d/nvidia.conf
    Section "Module"
        Load "modesetting"
    EndSection
    Section "Device"
        Identifier "nvidia"
        Driver "nvidia"
        BusID "1:0:0"
        Option "AllowEmptyInitialConfiguration"
    EndSection
vim /usr/lib64/X11/xdm/Xsetup_0 # go from /etc/X11/xdm/xdm-config
    xrandr --setprovideroutputsource modesetting NVIDIA-0
    xrandr --output eDP-1-1 --off
    xrandr --output HDMI-1-2 --mode 2560x1080
vim /etc/modprobe.d/blacklist.conf
    blacklist nouveau
```

### taskopen
```sh
emerge -av dev-perl/JSON x11-misc/xdg-utils

git clone https://github.com/jschlatow/taskopen
cd taskopen; cp taskopen ~/.bin
```

### set qutebrowser as default
```sh
ls /usr/share/applications/org.qutebrowser.qutebrowser.desktop
xdg-settings set default-web-browser org.qutebrowser.qutebrowser.desktop
```

### set mupdf for pdf & djviewer for djvu
```sh
emerge -av mupdf app-text/djview

file --mime some.pdf # get name of mimetype for pdf: applications/pdf
file --mime some.djvu # get name of mimetype for djvu: image/vnd.djvu
xdg-mime query default application/pdf # get current value: calibre-ebook-viewer.desktop
xdg-mime query default image/vnd.djvu # get current value: calibre-ebook-viewer.desktop
ls /usr/share/applications/mupdf.desktop
ls /usr/share/applications/djvulibre-djview4.desktop
xdg-mime default mupdf.desktop "application/pdf"
xdg-mime default djvulibre-djview4.desktop "image/vnd.djvu"
xdg-mime default djvulibre-djview4.desktop "image/vnd.djvu+multipage"
```

### bluetooth headset
```sh
vim /etc/portage/package.use
    media-sound/pulseaudio native-headset ofono-headset bluetooth dbus
emerge -av media-sound/pulseaudio
pulseaudio -k; pulseaudio --start
```
[1]: https://wiki.gentoo.org/wiki/Bluetooth_headset

### transmission
```sh
emerge -av transmission tremc
vim /var/lib/transmission/config/settings.json
    "rpc-host-whitelist-enabled": false,
    "rpc-whitelist-enabled": false,
    "download-dir": "/home/efim/myHomeDir/Downloads",
    "incomplete-dir": "/var/lib/transmission/downloads",
    "incomplete-dir-enabled": true,
rc-update add transmission-daemon default
openrc
```
[1]: https://addons.mozilla.org/en-US/firefox/addon/transmission-easy-client

### use second screen over x11vnc
*under work*
```sh
vim /etc/X11/xorg.conf.d/nvidia.conf
    Section "Device"
        Identifier "intelgpu0"
        Driver     "intel"
        Option     "VirtualHeads" "2"
    EndSection
/etc/init.d/xdm restart

xrandr --newmode "1280x720_60.00" 74.48  1280 1336 1472 1664  720 721 724 746  -HSync +Vsync
xrandr --addmode VIRTUAL1 "1280x720_60.00"
xrandr --output VIRTUAL1 --right-of eDP1

x11vnc -forever -bg -geometry 1280x720 -shared -noprimary \
    -auth /var/lib/xdm/authdir/authfiles/A\:0-CkV3z6 -display :0 \
    -clip 1280x720+1920+0 -threads -noxdamage
```
[1]: https://magazine.odroid.com/article/multi-screen-desktop-using-vnc-part-2-an-improved-and-simplified-version/

### one of solution for rebuild kernel
```sh
genkernel --menuconfig --no-clean all
```

### fixing: Bluetooth: hci0: don't support firmware rome 0x1020200
```sh
emerge -av sys-firmware/bluez-firmware
echo "net-wireless/bluez deprecated" >> /etc/portage/package.use
emerge -av net-wireless/bluez
hciconfig hci0 up # see: dmesg -Hw
```
*Bug fixed on kernel since 5.11.2*
[1]: https://github.com/BrandomRobor/btusb-210681-fix/blob/main/rome_fix.patch

### something like solution for fix trouble when can't play sound through hdmi
```sh
echo "options snd_hda_intel probe_oly=0" >> /etc/modprobe.d/snd.conf
```
Errors in dmesg continues to be:
```
[  +0.000020] snd_hda_intel 0000:00:03.0: spurious response 0x0:0x0, last cmd=0x470503
[  +0.000021] snd_hda_intel 0000:00:03.0: spurious response 0x0:0x0, last cmd=0x570503
[  +0.000020] snd_hda_intel 0000:00:03.0: spurious response 0x0:0x0, last cmd=0x670503
[  +0.000021] snd_hda_intel 0000:00:03.0: spurious response 0x0:0x0, last cmd=0x770503
```
but sound can be play through hdmi output
[1]: https://github.com/linux-surface/linux-surface/issues/39

### prosody
*under work*
```sh
emerge -av net-im/prosody
vim /etc/jabber/prosody.cfg.lua
```
[1]: https://prosody.im/doc/configure

### when install Qt via online installer
If install Qt via online installer (qt-unified-linux-x64-4.0.1-1-online.run)
may has error about *krb* like this:
*error while loading shared libraries: libgssapi_krb5.so.2: cannot open shared...*
for fix it need install mit-krb5: `emerge -av app-crypt/mit-krb5`.

### configure of pptp client
```sh
emerge -av net-dialup/pptpclient
pptpsetup \
    --create "<tunnelname>" \
    --server "<hostname>" \
    --username "<username>" \
    --password "<password>"
pppd call "<tunnelname>"
route add -net 192.168.70.0/24 gw 192.168.70.1

killall pppd # for disconnect
```

### docker
```sh
emerge -av \
    app-emulation/docker \
    app-emulation/docker-compose \
    app-emulation/docker-machine
emerge -av app-emulation/virtualbox
gpasswd -a <user> vboxusers
modprobe vboxdrv vboxnetadp vboxnetflt
docker-machine create default
eval $(docker-machine env default)
cat << EOF > default-compose.yaml
version: '2'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: wordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
EOF
docker-compose up
```

### nvtop
```sh
emerge -av app-portage/layman
layman-updater -R
layman -a guru
emerge -av nvtop
nvtop
```

### connect to termux sshd over usb cabel
```sh
emerge -av dev-util/android-tools

 # at phone:
 # 1. go to Settings > About device
 # 2. tap the Build number 7 times to make developer options available
 # 3. go to Settings > System > Developer Options
 # 4. now hit Enable USB-Debugging

adb devices # list of attached devices
adb forward tcp:6100 tcp:7100 # forware local port 6100 to port 7100 of phone

 # run in termux
whoami # get username for connet
passwd # and set password for this user
sshd -p 7100

ssh u0_a166@localhost -p 6100 # now may connect to sshd of termux
```
[1]: https://wiki.gentoo.org/wiki/Android/adb

### ccache
``` sh
emerge -av dev-util/ccache
vim /etc/portage/make.conf
    FEATURES="ccache"
    CCACHE_DIR="/var/cache/ccache"
vim ~/.zshrc
    export PATH="/usr/lib/ccache/bin${PATH:+:}${PATH}"
    export CCACHE_DIR="/var/cache/ccache"
ccache -s # show statistics
```

### ranger with support of view images in alacritty
```sh
emerge -av media-gfx/ueberzug
cat << EOF >> $HOME/.config/ranger/rc.conf
set preview_images true
set preview_images_method ueberzug
EOF
```
[1]: https://github.com/ranger/ranger/wiki/Image-Previews

### use weak password
```sh
sed 's/enforce=everyone/enforce=none/' -i /etc/security/passwdqc.conf
```

### some of solution for search package which contains file
```sh
emerge -av app-portage/pfl
e-file $(which mkfs.ext4)
```

### sdcv
```sh
emerge -av \
    app-text/sdcv \
    app-dicts/stardict-quick-ru-en \
    app-dicts/stardict-freedict-eng-rus
```

### ebuilds
#### writing ebuilds
```
mkdir /var/db/repos/SOME_REPO_NAME; cd /var/db/repos/SOME_REPO_NAME
mkdir sci-libs/libsigrok4DSL; cd sci-libs/libsigrok4DSL
vim libsigrok4DSL-0.2.0.ebuild
ebuild ./libsigrok4DSL-0.2.0.ebuild manifest clean install
```
#### ebuilds repository
```sh
emerge -av app-eselect/eselect-repository
eselect repository create SOME_REPO_NAME
```
##### put to git (if needed)
```sh
cd /var/db/repos/SOME_REPO_NAME
git init .; git add *; git commit -m "some message"; # git ...
eselect repository remove -f SOME_REPO_NAME
eselect repository add SOME_REPO_NAME git url://some/path/to/git/repo
emerge --sync SOME_REPO_NAME
emerge -av libsigrok4DSL
```
[1]: https://wiki.gentoo.org/wiki/Handbook:Parts/Portage/CustomTree#Defining_a_custom_repository
[2]: https://devmanual.gentoo.org/ebuild-writing/variables/index.html
[3]: https://devmanual.gentoo.org/general-concepts/autotools/index.html

### building of gentoo's admincd iso
*under work*
```sh
emerge -av catalyst pixz
git clone https://gitweb.gentoo.org/proj/releng.git
mkdir -p /var/tmp/catalyst/builds/default/

wget https://gentoo.osuosl.org/releases/amd64/autobuilds/current-stage3-amd64-openrc/stage3-amd64-openrc-20220619T170540Z.tar.xz \
    -O /var/tmp/catalyst/builds/default/stage3-amd64-openrc-latest.tar.xz

catalyst -s latest

cat << EOF > stage1-openrc.sed
s#@REPO_DIR@#$(pwd)/releng#g
s#@TIMESTAMP@#latest#g
EOF
catalyst -f <(sed -f stage1-openrc.sed \
    releng/releases/specs/amd64/stage1-openrc.spec)

cat << EOF > installcd-stage1.sed
s#@REPO_DIR@#$(pwd)/releng#g
s#@TIMESTAMP@#latest#g
/livecd.packages/ {
  a     app-editors/vim
}
EOF
catalyst -f <(sed -f installcd-stage1.sed \
    releng/releases/specs/amd64/installcd-stage1.spec)

echo 'GRUB_PLATFORMS="efi-64 efi-32"' >> /etc/portage/make.conf
emerge -av grub memtest86+
cat << EOF > installcd-stage2-minimal.sed
s#@REPO_DIR@#$(pwd)/releng#g
s#@TIMESTAMP@#latest#g
EOF
catalyst -f <(sed -f installcd-stage2-minimal.sed \
    releng/releases/specs/amd64/installcd-stage2-minimal.spec)
```
[1]: https://github.com/gentoo/catalyst/blob/master/examples/livecd-stage2_template.spec

### set usb0 for USB network card
```sh
cat << EOF >> /etc/udev/rules.d/net.rules
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="01:23:45:67:89:ab", NAME="usb0"
EOF
udevadm control --reload-rules && udevadm trigger
```
[1]: https://unix.stackexchange.com/questions/386162/how-to-set-up-an-usb-ethernet-interface-in-linux
