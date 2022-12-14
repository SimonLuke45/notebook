## Upgrade Fedora 34 to verison 36

```bash
dnf upgrade
dnf --refresh upgrade
dnf install dnf-plugin-system-upgrade --best
dnf system-upgrade download --refresh --releasever=36
dnf system-upgrade reboot
yum update
```

Network configure:

```bash
# sed -i 's@^#\?\(DNS=.*\)@DNS=10.83.50.140 8.8.8.8@g' /etc/systemd/resolved.conf
systemctl restart systemd-resolved
rm -f /etc/resolv.conf; ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

## Install xrdp:

```bash
yum group install "KDE Plasma Workspaces"
yum install -y xrdp
systemctl enable --now xrdp
# If Firewalld is running, allow RDP port.
firewall-cmd --add-port=3389/tcp --permanent
firewall-cmd --reload
```
xrdp support sound: [pulseaudio-module-xrdp](https://github.com/neutrinolabs/pulseaudio-module-xrdp/wiki/Build-on-Fedora)

```bash
dnf install mock
usermod -a -G mock $USER
# Log out and log in again
dnf download --source pulseaudio
mock ./pulseaudio-*.src.rpm

yum remove pipewire-pulseaudio
yum install autoconf automake make libtool libtool-ltdl-devel pulseaudio-libs-devel git
git clone https://github.com/neutrinolabs/pulseaudio-module-xrdp.git
cd pulseaudio-module-xrdp
export PULSE_DIR=/var/lib/mock/fedora-36-x86_64/root/builddir/build/BUILD/pulseaudio-15.0
./bootstrap && ./configure PULSE_DIR=$PULSE_DIR &&　make && make install
ls $(pkg-config --variable=modlibexecdir libpulse) | grep xrdp
#If you can see module-xrdp-sink.so and module-xrdp-source.so, PulseAudio modules are properly built and installed.

yum install paman pamix paprefs pasystray pavucontrol
# Setting pulaeaudio as default audio device:
# Open application "PulseAudio Preferences/Simultaneous Output/Add virtual output device for simultaneous ouput on all local sound cards"
```
migrate pulseaudio-xrdp to other Fedora:

```bash
tar jcvf pulseaudio.tar.bz /var/lib/mock/fedora-36-x86_64/root/builddir/build/BUILD/pulseaudio-15.0 /var/lib/mock/pulseaudio-module-xrdp
cd / && tar jxvf pulseaudio.tar.bz && /var/lib/mock/pulseaudio-module-xrdp && make install
ls $(pkg-config --variable=modlibexecdir libpulse) | grep xrdp
#If you can see module-xrdp-sink.so and module-xrdp-source.so, PulseAudio modules are properly built and installed.

yum install paman pamix paprefs pasystray pavucontrol
# Setting pulaeaudio as default audio device:
# Open application "PulseAudio Preferences/Simultaneous Output/Add virtual output device for simultaneous ouput on all local sound cards"
```

## Install chrome browser:

```bash
# CentOS
cat << EOF > /etc/yum.repos.d/google-chrome.repo
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/x86_64
enabled=1
gpgcheck=1
gpgkey=https://dl.google.com/linux/linux_signing_key.pub
EOF

yum install google-chrome-stable

# Fedora
yum install fedora-workstation-repositories
yum config-manager --set-enabled google-chrome
yum install google-chrome-stable
```

## Install Microsoft Edge browser:

```bash
rpm --import https://packages.microsoft.com/keys/microsoft.asc
yum-config-manager --add-repo https://packages.microsoft.com/yumrepos/edge
yum install microsoft-edge-stable
```

## Install vscode:

```bash
cat << EOF > /etc/yum.repos.d/vscode.repo
[code]
name=Visual Studio Code
baseurl=https://packages.microsoft.com/yumrepos/vscode
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc
EOF

rpm --import https://packages.microsoft.com/keys/microsoft.asc
yum install code
```

## Install Chinese fonts:

```bash
yum install google-noto-sans-simplified-chinese-fonts
yum install google-noto-cjk-fonts google-noto-sans-cjk-fonts

yum install ibus-rime
git clone https://github.com/Openvingen/rime-zhengma.git
cp rime-zhengma/{zhengma,zmbig,zmb,zmjd,pinyin123}.*.yaml  /usr/share/rime-data
sed -i -e  '/schema_list:/a\ \ - schema: zhengma' /usr/share/rime-data/default.yam

cat << EOF > /etc/locale.conf
LANG=zh_CN.UTF-8
LC_ALL=zh_CN.UTF-8
LC_CTYPE=zh_CN.UTF-8
EOF
```

## Install Podman & Docker container

```bash
yum-config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
systemctl enable --now docker

# podman
dnf install podman

# support systemd in container
mkdir -p /sys/fs/cgroup/systemd && mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
```

## Install tmux/zsh

```bash
yum install zsh tmux autojump
```

## Set ssh tunnel server

```bash
# On public IP server: 52.1.1.1
adduser -s /sbin/nologin temp
yes '1J$yZqk{8kgMxbw' | passwd temp
sed -i 's/.*GatewayPorts.*/GatewayPorts yes/' /etc/ssh/sshd_config
systemctl restart sshd

# On private server: target is 10.216.29.41:22
ssh -p 8822 -NR 0.0.0.0:9922:10.216.29.41:22 temp@52.1.1.1
# now 52.1.1.1:9922 is same 10.216.29.41:22

# On other client
ssh -p 9922 <10.216.29.41 user>@52.1.1.1
```
