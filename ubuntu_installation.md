## ipv6

[setting](https://github.com/tuna/ipv6.tsinghua.edu.cn/blob/master/isatap.md)

清华大学ISATAP隧道配置方法

Windows 环境（Windows 7及以上系统适用）

以管理员身份运行cmd命令，进入命令行模式，输入如下命令

```
netsh int ipv6 isatap set router isatap.tsinghua.edu.cn
netsh int ipv6 isatap set state enable
```

以上两条命令分别为设定ISATAP路由器和启用ISATAP隧道。

此后，通过 ipconfig 应该可以看到一个 2402:f000:1:1501: 为前缀的v6地址，hostid为200:5efe:x.x.x.x， 其中x.x.x.x 为您的真实的IPv4地址，即可访问IPv6资源。

以下操作为非必须。如果按照上述提示操作以后仍无法正常访问IPv6站点，可以尝试：

+ 右键点击桌面“计算机”图标，选择“管理”，展开“服务和应用程序”，选择“服务”，确认“IP Helper”服务已开启；

+ 确认Teredo隧道已经关闭（管理员模式在命令行运行netsh int teredo set state disable）；

+ 确认原生IPv6已经关闭（Internet 协议版本 6 (TCP/IPv6)前的对勾取消，位置在控制面板→网络和Internet→网络和共享中心→更改适配器设置→双击本地连接→属性）；

+ 尝试重启系统。


## installation of ubuntu along with window10

### partition

install option: 其他选项

挂载点: `/`, 20G+

swap area: 15G+

`/boot`, 200M+

`/home`, the more, the better

choose boot partition

### Time zone

[solution](https://blog.otorb.com/2017/01/15/solve-the-linux-and-windows-time-synchronization-problem/)

```shell
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
sudo hwclock --localtime --systohc
```

### boot optional

change boot order: check the current boot order while booting. Normally entry 0 is ubuntu, entry 1 is ubuntu recovery mode, entry 2 is windows... Then edit `etc/default/grub` by executing `sudo gedit /etc/default/grub`. Then change the `GRUB_DEFAULT=0` to any entry you wish. Finally, executing `sudo update-grub` to update your grub.

#### grub启动背景

默认使用系统自带的壁纸, 目录是/usr/share/backgrounds, 把图片文件移动到/boot/grub目录下, 更新grub的时候会自动识别.

```shell
sudo mv mypicture.jpg /boot/grub
sudo update-grub
```
### ubuntu美化

`sudo apt-get install unity-tweak-tool`

可以修改主题, 图标, 修改工作区数量,热区等.

system load indicator, 用来监视系统的各项性能信息.

```shell
sudo add-apt-repository ppa:indicator-multiload/stable-daily
sudo apt-get update
sudo apt-get install indicator-multiload
```

Variety is a wallpaper changer.

```shell
sudo add-apt-repository ppa:peterlevi/ppa
sudo apt-get update
sudo apt-get install variety
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
    "server": ["[::0]", "0.0.0.0"], # [::0] represents ipv6; "0.0.0.0" represents ipv4
    "server_port":8888,
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
    "server":"ipv4address or ipv6address",
    "server_port":8888,
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

**adjust themes or fontsize**

[jupyterthemes](https://github.com/dunovank/jupyter-themes)

```
# install jupyterthemes
pip install jupyterthemes

# upgrade to latest version
pip install --upgrade jupyterthemes
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


### ROOT

1. Build Prerequisites

   https://root.cern.ch/build-prerequisites

   Required packages:

```shell
sudo apt-get install git dpkg-dev cmake g++ gcc binutils libx11-dev libxpm-dev libxft-dev libxext-dev
```

  	Optional packages:

```shell
sudo apt-get install gfortran libssl-dev libpcre3-dev xlibmesa-glu-dev libglew1.5-dev libftgl-dev libmysqlclient-dev libfftw3-dev libcfitsio-dev graphviz-dev libavahi-compat-libdnssd-dev libldap2-dev python-dev libxml2-dev libkrb5-dev libgsl0-dev libqt4-dev
```




2. Install ROOT(version 6.8.6)

```shell
mkdir root(ROOTDIR)
cd root
wget https://root.cern.ch/download/root_v6.08.06.source.tar.gz
# wget https://root.cern.ch/download/root_<version>.source.tar.gz
tar zxvf root_v6.08.06.source.tar.gz
mv root_v6.08.06.source.tar.gz /tmp
mv root-6.08.06 src
mkdir root-6.08.06
mv src root-6.08.06
cd root-6.08.06
mkdir build install
echo cmake -DCMAKE_INSTALL_PREFIX=../install -Dgdml=ON -Dminuit2=ON -Droofit=ON -Dhttp=ON -Dldap=OFF ../src > cmakeCommand.txt
cd build
`cat ../cmakeCommand.txt` >& cmake.log
less cmake.log
# [check everything looks okay]
make -j$N
make install
```

If ROOT is used for python3, add `-Dpython3=ON` during cmake.

```
cmake -DCMAKE_INSTALL_PREFIX=../install -Dpython3=ON -Dgdml=ON -Dminuit2=ON -Droofit=ON -Dhttp=ON -Dldap=OFF ../src
```

3. set environment variables

   Before using ROOT, one should set environment variables of ROOT with:

```shell
source path-to-root/install/bin/thisroot.sh
```

   	Or, one can set environment variables in ~/.zshrc

   ```shell
cd /home/pxy/Software/MJSW/root/root-6.08.06/install/bin
source thisroot.sh
cd ~
   ```


### CLHEP

```shell
mkdir CLHEP
cd $SWDIR/CLHEP
wget http://proj-clhep.web.cern.ch/proj-clhep/DISTRIBUTION/tarFiles/clhep-2.3.4.4.tgz
tar zxvf clhep-2.3.4.4.tgz
mv clhep-2.3.4.4.tgz /tmp
cd 2.3.4.4
mkdir build install
echo cmake -DCMAKE_INSTALL_PREFIX=../install ../CLHEP > cmakeCommand.txt
cd build
`cat ../cmakeCommand.txt` >& cmake.log
less cmake.log
# [check everything looks okay]
make -j$N
make install
```

Then, set the environment variable of CLHEP

```shell
export CLHEP_BASE_DIR=/home/pxy/Software/CLHEP/2.3.4.4/CLHEP
```

### Geant4

[yangzw's method](http://hep.tsinghua.edu.cn/~yangzw/CourseDataAna/2015/install_geant4_and_root_on_ubuntu.html)

required package according to yangzw's method.

```shell
sudo apt-get install libxerces-c-dev libmotif-dev libglw1-mesa-dev inventor-dev opticalraytracer libpythia8-dev pythia8-doc-html pythia8-doc-worksheet pythia8-examples
```

Actually, during the installation with the following cmake command, only two packages are required:

```shell
sudo apt-get install libxerces-c-dev libxmu-dev
```

Some optional but suggested packages:

```shell
sudo apt-get install vim vim-doc vim-scripts emacs colordiff lftp dkms build-essential libzlcore-dev libxmu-dev
```

There are something wrong between MaGe and Geant4 multithread. So, don't use multi thread, i.e using:

```-DGEANT4_BUILD_MULTITHREADED=OFF```


If gdml is planed to be used for MaGe, the option of GDML should be set to be `-DGEANT4_USE_GDML=ON`, the default value is OFF. seeing from [Geant4 build options](https://geant4.web.cern.ch/geant4/UserDocumentation/UsersGuides/InstallationGuide/html/ch02s03.html)

```shell
cd $SWDIR/geant4
wget http://geant4.web.cern.ch/geant4/support/source/geant4.10.03.p01.tar.gz
tar zxvf geant4.10.03.p01.tar.gz
mv geant4.10.03.p01.tar.gz /tmp
cd geant4.10.03.p01
mkdir build
echo cmake -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DCMAKE_INSTALL_PREFIX=../install -DCMAKE_BUILD_TYPE=RelWithDebInfo -DGEANT4_INSTALL_DATA=ON -DCLHEP_ROOT_DIR=/home/pxy/Software/CLHEP/2.3.4.4/install -DCMAKE_COMPILER_IS_GNUCXX=ON -DGEANT4_USE_OPENGL_X11=ON -DGEANT4_USE_RAYTRACER_X11=ON -DGEANT4_USE_GDML=ON -DGEANT4_USE_QT=ON -DGEANT4_BUILD_MULTITHREADED=ON ../src > cmakeCommand.txt
cd build
`cat ../cmakeCommand.txt` >& cmake.log
less cmake.log
# [check everything looks okay]
make -j$N
make install
```

PS: cmake options

`DCMAKE_INSTALL_PREFIX=/home/pxy/Software/MJSW/geant4/geant4.10.03.p01` is the `install directory`

`/home/pxy/Software/MJSW/geant4/geant4.10.03.p01` in the end of command is the `source directory `(including makefile)

the current directory is the `build directory`

With command of `-DGEANT4_INSTALL_DATA=ON`, Geant4 datasets missing from `GEANT4_INSTALL_DATADIR` will be downloaded and installed. If there are any downloaded datasets already, using `-DGEANT4_INSTALL_DATADIR=/path/to/G4data`.



Set environment variables

Before using ROOT, one should set environment variables of ROOT with:

```shell
source path-to-geant4/install/bin/geant4.sh
```

   Or, one can set environment variables in ~/.zshrc

```shell
cd ~/Software/MJSW/geant4/geant4.10.03.p01/bin
source geant4.sh
cd ~
```

[How to Run Example](http://geant4.web.cern.ch/geant4/UserDocumentation/Doxygen/examples_doc/html/README_HowToRun.html)

```shell
cd path_to_exampleXYZ     # go to directory which contains your example
mkdir exampleXYZ_build
cd exampleXYZ_build
cmake -DGeant4_DIR=path_to_Geant4_installation/lib[64]/Geant4-10.2.0/ ../exampleXYZ
make -j N exampleXYZ      # "N" is the number of processes 
make install			  # optional(better not)
./exampleXYZ			  # run the example
```


