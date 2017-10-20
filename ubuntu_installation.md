## installation of ubuntu along with window10

### Time zone

[solution](https://blog.otorb.com/2017/01/15/solve-the-linux-and-windows-time-synchronization-problem/)

```shell
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
sudo hwclock --localtime --systohc
```

### zsh and oh-my-zsh

[Install zsh](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH)

```shell
sudo apt install zsh
```

**change to zsh, it may works after restart the system**

```shell
chsh -s $(which zsh)
```

[Install oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

**install git**

```shell
sudo apt install git
```

**via curl**

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

**Or via wget**

```shell
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

change theme to candy:

```shell
vim ~/.zshrc
```

### albert

[installation of albert](https://albertlauncher.github.io/docs/installing/)

```shell
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update
sudo apt-get install albert
```

设置开机启动:

open albert->search startup applications->添加->名称: Albert; 命令: albert

### gpu drives

[nvidia drives](http://www.jianshu.com/p/c0f3a503df69)

### ipv6 in tsinghua

create configuration file `config.sh`

```shell
#!/bin/bash
REMOTE_IP6="2402:f000:1:1501:200:5efe"
REMOTE_IP4="166.111.21.1"

IFACE4=`ip route show|grep default|sed -e 's/^default.*dev \([^ ]\+\).*$/\1/'`
IP4=`ip addr show dev $IFACE4 | grep -m 1 'inet\ ' | sed -e 's/^.*inet \([^ \\]\+\)\/.*$/\1/'`

sudo ip tunnel del sit1  # 删除已经创建的设备，若没有则忽略
sudo ip tunnel add sit1 mode sit remote $REMOTE_IP4 local $IP4
sudo ip link set dev sit1 up
sudo ip -6 addr add $REMOTE_IP6:$IP4/64 dev sit1
sudo ip -6 route add default via $REMOTE_IP6:$REMOTE_IP4 dev sit1
```

Then, `source config.sh` 

### shadowsocks

#### [install shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)

For Ubuntu 14.04 and 16.04 users, please install from PPA:

```shell
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev
sudo apt-get update
sudo apt install shadowsocks-libev
```

#### arrange an oversea server

vultr

#### server configuration

```shell
# Create and edit the configuration file config.json
ss-server -c /path-to-config.json
nohup ss-server -c /path-to-config.json >output.file 2>&1 &
```

an example of config.json

```
{
    "server": ["[::0]", "0.0.0.0"] # [::0] represents ipv6; "0.0.0.0" represents ipv4
    "server_port":8888
    "local_port":1080,
    "password":"password",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```

#### user configuration

```shell
# Create and edit the configuration file config.json
ss-local -c /path-to-config.json
nohup ss-local -c /path-to-config.json >output.file 2>&1 &
```

an example of config.json
```
{
    "server":"ipv4address or ipv6address"
    "server_port":8888
    "local_port":1080,
    "password":"password",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```

### google chrome installation

```shell
sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
sudo apt-get update
sudo apt-get install google-chrome-stable
```

### vs code

[installation](http://tipsonubuntu.com/2017/03/03/install-ms-visual-studio-code-ubuntu-16-0416-10/)

```shell
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update
sudo apt install code
```

### sougou pinyin

[installation](http://ubuntuhandbook.org/index.php/2016/07/2-best-chinese-pinyin-im-ubuntu-16-04/)

```shell
sudo apt install fcitx fcitx-googlepinyin fcitx-table-wbpy fcitx-pinyin fcitx-sunpinyin
```


### python and jupyter notebook

**install python3**

To see which version of Python 3 you have installed, open a command prompt and run

```
python3 --version
```

If you are using Ubuntu 16.10 or newer, then you can easily install Python 3.6 with the following commands:

```shell
sudo apt-get update
sudo apt-get install python3.6
```

If you’re using another version of Ubuntu (e.g. the latest LTS release), we recommend using the deadsnakes PPA to install Python 3.6:

```shell
sudo add-apt-repository ppa:fkrull/deadsnakes
sudo apt-get update
sudo apt-get install python3.6
```

**install pip3**

```shell
sudo apt-get install python3-pip
```

**install jupyter, use pip if you want to use python2**

```shell
sudo pip3 install --upgrade pip
sudo pip3 install jupyter
```

**install numpy, matplotlib...**

```shell
sudo pip3 install numpy
```

**check pip3 list**

```shell
pip3 list
pip3 show numpy
```

### typora

[installation](http://support.typora.io/Typora-on-Linux/)

```shell
# optional, but recommended
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE

# add Typora's repository
sudo add-apt-repository 'deb https://typora.io ./linux/'
sudo apt-get update

# install typora
sudo apt-get install typora
```

### GNU octave

Free Software alternative to Matlab

```shell
sudo add-apt-repository ppa:octave/stable
sudo apt-get update
sudo apt-get install octave
```

### media player

mpv media player

```shell
sudo add-apt-repository ppa:mc3man/mpv-tests
sudo apt-get update
sudo apt-get install mpv
```

### QOwnNotes

```shell
sudo add-apt-repository ppa:pbek/qownnotes
sudo apt-get update
sudo apt-get install qownnotes
```

### search ppa

[website](https://launchpad.net/ubuntu)

### cmake



### other 

```shell
sudo apt-get install htop
```

check disk status: use `sudo fdisk -l`
