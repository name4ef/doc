 - update groups: `newgrp vboxuser`
 - change login shell: `chsh` (for termux: in termux-tools)
 - convert to utf8: `enca -L ru -x utf8 somefile.txt`

### only console users are allowed to run the X server (fix it)
```sh
echo "allowed_users=anybody" >> /etc/X11/Xwrapper.config
```

### tar-backup over ssh
```sh
tar zcvf - Backup | ssh user@host "cat > Backup/host/Backup.tar.gz"
ssh user@host "cat Backup/host/Backup.tar.gz" | pv | tar zxf -
```
[1]: https://www.cyberciti.biz/faq/howto-use-tar-command-through-network-over-ssh-session/
