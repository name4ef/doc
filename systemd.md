```sh
systemctl --type=service # show current services
systemctl --type=service -a # show all services
journalctl -f --no-hostname # logs
systemctl disable avahi-daemon
systemctl mask systemd-timesyncd # force disabling service
```
