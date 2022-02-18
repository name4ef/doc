- check server or desktop version: `dpkg -l ubuntu-desktop`
- disable service: `update-rc.d tvheadend disable`
- install pip: `apt install python3-pip`
- show files of package: `apt-file list tmux`

### seting mirror
```sh
# example for 18.04 (bionic)
vim /etc/apt/sources.list
    deb http://mirror.yandex.ru/ubuntu/ bionic main restricted universe multiverse
    deb http://mirror.yandex.ru/ubuntu/ bionic-backports main restricted universe multiverse
    deb http://mirror.yandex.ru/ubuntu/ bionic-proposed main restricted universe multiverse
    deb http://mirror.yandex.ru/ubuntu/ bionic-security main restricted universe multiverse
    deb http://mirror.yandex.ru/ubuntu/ bionic-updates main restricted universe multiverse
apt update
```
