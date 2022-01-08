timewarrior: install and integrate with taskwarrior
---------------------------------------------------
```sh
brew install timewarrior
mkdir -p ~/.task/hooks/
cp /usr/local/Cellar/timewarrior/1.2.0/share/doc/timew/ext/on-modify.timewarrior ~/.task/hooks/
chmod +x ~/.task/hooks/on-modify.timewarrior
```
[1]: https://timewarrior.net/docs/taskwarrior.html

vim + taskwiki
--------------
```sh
brew install vim
export PATH=$PATH:/usr/local/bin/ # add to .~/.zshrc
```
# fix: "ModuleNotFoundError: No module named 'six'"
```sh
pip3 install six tasklib # pip3.9 may be helpfull
```

forwarding traffic between interfaces
-------------------------------------
/etc/pf.conf:
```pf
int_if = "vboxnet0"
ext_if = "en0"
lan_net = "192.168.56.0/24"
ext_addr = "192.168.8.100"

#
# com.apple anchor point
#
scrub-anchor "com.apple/*"
nat on $ext_if from $lan_net to any -> $ext_addr
#nat on $ext_if from $int_if:network to any -> ($ext_if)
#pass out on $ext_if from $lan_net to any nat-to $ext_addr
nat-anchor "com.apple/*"
rdr-anchor "com.apple/*"
dummynet-anchor "com.apple/*"
anchor "com.apple/*"
load anchor "com.apple" from "/etc/pf.anchors/com.apple"
```

```sh
sysctl net.inet.ip.forwarding=1
pfctl -f /etc/pf.conf -e
```

format usb flash to ext3
------------------------
```sh
brew install e2fsprogs
diskutil list # get partition name (disk2s1)
diskutil umountDisk disk2
sudo $(brew --prefix e2fsprogs)/sbin/mkfs.ext3 /dev/disk2s1
```
[1]: https://apple.stackexchange.com/questions/171506/formatting-usb-disk-as-ext3-on-mac

mount ext3 (ro)
---------------
```sh
brew cask install osxfuse
brew install ext4fuse 
sudo ext4fuse /dev/disk2s1 mnt -o allow_other
```
[1]: https://hackmylinux.com/2018/02/18/how-to-mount-and-read-a-linux-partition-on-a-mac-ext2-ext3-ext4/

write iso image to usb flash
----------------------------
```sh
diskutil list # get partition name (disk2)
diskutil umountDisk disk2
sudo dd bs=1m if=<PATH TO ISO> of=/dev/disk2
diskutil eject disk2
```

build Xcode project over commend line
-------------------------------------
```sh
# For fix trouble: "xcode-select: error: tool 'xcodebuild' requires Xcode, but 
# active developer directory '/Library/Developer/CommandLineTools' is a command
# line tools instance" run this:
# sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
cd ~/dev/tir/TextXCode7/projects/osx; # Go to project dirrectory
xcodebuild -list -project Test.xcodeproj; # List all information about project
xcodebuild -scheme "Test Debug" build; # Building "Test Debug" scheme
~/Library/Developer/Xcode/DerivedData/Test-fhwmxqkzecgaspckjkghrxesoood/Build/Products/Debug/Test.app/Contents/MacOS/Test
xcodebuild -scheme "Test Debug" SYMROOT=../../build; # Debug build: ../../build/Debug/Test.app
xcodebuild -scheme "Test Release" DSTROOT=../../build archive; # Release build: ../../build/Applications/Test.app
```
```make
all:
	xcodebuild -project projects/osx/Test.xcodeproj -scheme "Test Debug" SYMROOT=../../build

run:
	./build/Debug/Test.app/Contents/MacOS/Test
```
[1]: https://developer.apple.com/library/archive/technotes/tn2339/_index.html#//apple_ref/doc/uid/DTS40014588-CH1-HOW_DO_I_BUILD_MY_PROJECTS_FROM_THE_COMMAND_LINE_
[2]: https://stackoverflow.com/questions/17980759/xcode-select-active-developer-directory-error

Display notification from the Mac command line
----------------------------------------------
https://code-maven.com/display-notification-from-the-mac-command-line

analog of some Linux's commands
-------------------------------
- show routes: `netstat -rlnf inet`
- netstat: `lsof -nP -iTCP | grep LISTEN`
- ldd: `otool -L`
- lsusb: `system_profiler SPUSBDataType`

running qemu vm
---------------
```sh
brew install qemu

#dd if=/dev/zero of=ubuntu.img bs=1g seek=24 count=0
qemu-img create -f qcow2 ubuntu.qcow2 24G

    #-display curses,show-cursor=on -vga virtio \
    #-cdrom ~/iso/ubuntu-16.04.6-desktop-amd64.iso \
qemu-system-x86_64 \
    -cpu host -enable-kvm -machine accel=hvf \
    -smp 4,sockets=1,cores=2,threads=2 \
    -m 4G \
    -drive file=ubuntu.qcow2,if=virtio \
    -net user,hostfwd=tcp::2222-:22 -net nic \
    -display default,show-cursor=on -vga virtio \
    -usb -device usb-tablet
```

fixed: qemu-system-x86_64: Error: HV_ERROR
------------------------------------------
```sh
cat << EOF > app.entitlements
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>com.apple.security.hypervisor</key>
    <true/>
  </dict>
</plist>
EOF
codesign -s - --entitlements app.entitlements --force $(which qemu-system-x86_64)
```
[1]: https://www.reddit.com/r/qemu_kvm/comments/kdhmuj/qemu_hvf_support_for_mac_os_x_bug_sur_hv_error/

distcc on macos
---------------
http://eevee.cc/2017/05/05/use-ccache-distcc-dmucs-to-accelerate-builds/

set hostname
------------
```sh
sudo su
scutil --set HostName <hostname>
```
[1]: https://osxdaily.com/2010/09/06/change-your-mac-hostname-via-terminal/

run python project in docker
----------------------------
```sh
brew install docker
brew install docker-compose
brew install docker-machine xhyve docker-machine-driver-xhyve
sudo chown root:wheel \
    $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve 
sudo chmod u+s \
    $(brew --prefix)/opt/docker-machine-driver-xhyve/bin/docker-machine-driver-xhyve
brew install --cask virtualbox
#docker-machine create default --driver xhyve --xhyve-experimental-nfs-share
docker-machine create default
eval $(docker-machine env default)
docker-machine ssh default
cat << EOF > docker-compose.yaml
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
docker-compose up -d --build --remove-orphans

cd path/to/project; ln -s docker-compose_dev.yml docker-compose.yml
docker network create agentovwebbackend_agentov_network
docker-compose up
```
[1]: https://docs.docker.com/compose/
[2]:- https://pilsniak.com/how-to-install-docker-on-mac-os-using-brew

install sshfs
-------------
```sh
brew install --cask macfuse
brew install gromgit/fuse/sshfs-mac
```

run "Rig'n'Roll-3" game under wine
----------------------------------
```sh
# we're have ios: DALNOBOI_3_GOLD.iso
brew install wine-stable
wine64 --version # wine-5.0
hdiutil mount samfs/Soft/DALNOBOI_3_GOLD.iso # /dev/disk2 /Volumes/DALNOBOI_3_GOLD

cd /Volumes/DALNOBOI_3_GOLD
wine64 setup.exe
# 0009:err:module:__wine_process_init L"Z:\\Volumes\\DALNOBOI_3_GOLD\\setup.exe" not supported on this system


mkdir ~/DALNOBOI_3_GOLD; cd ~/DALNOBOI_3_GOLD
cp /Volumes/DALNOBOI_3_GOLD/setup.exe .
wine64 setup.exe
# 0009:err:module:__wine_process_init L"Z:\\Users\\efim\\DALNOBOI_3_GOLD\\setup.exe" not supported on this system
rm -rf ~/.wine
export WINEARCH=win32
wine64 setup.exe
# wine: WINEARCH set to win32 but '/Users/efim/.wine' is a 64-bit installation

cd ~/DALNOBOI_3_GOLD
wine setup.exe
# zsh: bad CPU type in executable: wine
file $(which wine) # /usr/local/bin/wine: Mach-O executable i386

brew install crossover
/Applications/CrossOver.app/Contents/SharedSupport/CrossOver/bin/wineloader32on64 setup.exe
# 0009:err:environ:run_wineboot failed to start wineboot c00000e5
# and so on...
```
[1]: https://wiki.winehq.org/MacOS
