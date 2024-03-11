# How to make a read only OrangePI 3B Ubuntu server

Tested with `Orangepi3b_1.0.4_ubuntu_jammy_server_linux5.10.160.img (9355457a4a2438e2cf33a6158a413c86d3513b09)`

`sudo su`

`apt-get update && apt-get upgrade`

`apt-get remove --purge wolfram-engine triggerhappy anacron logrotate dphys-swapfile xserver-common lightdm unattended-upgrades`

`systemctl disable x11-common`

`systemctl disable bootlogs`

`systemctl disable console-setup`

`apt-get install busybox-syslogd`

`dpkg --purge rsyslog`

`rm /var/lib/systemd/random-seed`

`ln -s /tmp/random-seed /var/lib/systemd/random-seed`

`nano /lib/systemd/system/systemd-random-seed.service`

and add

```
ExecStartPre=/bin/echo "" >/tmp/random-seed
```

`systemctl daemon-reload`

`swapoff -a`

`apt-get autoremove --purge`

`nano /etc/bash.bashrc`

and add at the end

```bash
set_bash_prompt() {

fs_mode=$(mount | sed -n -e "s/^\/dev\/.* on \/ .*(\(r[w|o]\).*/\1/p")
PS1='\[\033[01;32m\]\u@\h${fs_mode:+($fs_mode)}\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

}

alias ro='sudo mount -o remount,ro / ; sudo mount -o remount,ro /boot'
alias rw='sudo mount -o remount,rw / ; sudo mount -o remount,rw /boot'

PROMPT_COMMAND=set_bash_prompt
```

`source /etc/bash.bashrc`

`rm -rf /var/run /var/spool /var/lock`
`ln -s /tmp /var/run`
`ln -s /tmp /var/spool`
`ln -s /tmp /var/lock`

`nano /lib/systemd/system/NetworkManager.service`

and add

```
ExecStartPre=ln -s /run/dbus /var/run/dbus
```

`nano /etc/fstab`

add tmpfs paths and ro flag, example 

before

```
UUID=76310e5b-e207-4c0e-baa3-23ee8aaec715 / ext4 defaults,noatime,commit=600,errors=remount-ro 0 1
UUID=D9B2-1AB9 /boot vfat defaults 0 2
tmpfs /tmp tmpfs defaults,nosuid 0 0
```

after

```
UUID=76310e5b-e207-4c0e-baa3-23ee8aaec715 / ext4 defaults,noatime,ro,commit=600,errors=remount-ro 0 1
UUID=D9B2-1AB9 /boot vfat defaults,ro 0 2
tmpfs           /tmp            tmpfs   nosuid,nodev         0       0
tmpfs           /var/log        tmpfs   nosuid,nodev         0       0
tmpfs           /var/tmp        tmpfs   nosuid,nodev         0       0`
```


`nano /etc/bash.bash_logout`

add at the end

```bash
mount -o remount,ro /
mount -o remount,ro /boot
```

`sudo reboot`