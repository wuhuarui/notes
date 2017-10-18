## installation of ubuntu along with window10

### Time zone

[solution](http://blog.csdn.net/gatieme/article/details/51883981)

in short: `sudo vim /etc/default/rcS`, change `UTC=yes` to `UTC=no`

### zsh and oh-my-zsh

[Install zsh](https://github.com/robbyrussell/oh-my-zsh/wiki/Installing-ZSH)

```
sudo apt install zsh
```

#change to zsh, it may works after restart the system#

```
chsh -s $(which zsh)
```

[Install oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

#install git#

```
sudo apt install git
```

#via curl#
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

#Or via wget#
```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

change theme to candy:

```
vim ~/.zshrc
```

### albert

[installation of albert](https://albertlauncher.github.io/docs/installing/)

```
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update
sudo apt-get install albert
```

### gpu drives

[nvidia drives](http://www.jianshu.com/p/c0f3a503df69)

### shadowsocks

[installation of shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev)

For Ubuntu 14.04 and 16.04 users, please install from PPA:

```
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:max-c-lv/shadowsocks-libev
sudo apt-get update
sudo apt install shadowsocks-libev
```

#Configure and start the service#

```
# Create and edit the configuration file config.json
ss-local -c /path-to-config.json
```

an example of config.json
```
{
    "server":"0.0.0.0
    "server_port":8888
    "local_port":1080,
    "password":"password",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
```

### google chrome installation

```
sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
sudo apt-get update
sudo apt-get install google-chrome-stable
```

### vs code

[installation](http://tipsonubuntu.com/2017/03/03/install-ms-visual-studio-code-ubuntu-16-0416-10/)

```
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update
sudo apt install code
```

### sougou pinyin

[installation](http://ubuntuhandbook.org/index.php/2016/07/2-best-chinese-pinyin-im-ubuntu-16-04/)

```
sudo apt install fcitx fcitx-googlepinyin fcitx-table-wbpy fcitx-pinyin fcitx-sunpinyin
```


### python and jupyter notebook

#install python3#

To see which version of Python 3 you have installed, open a command prompt and run

```
python3 --version
```

If you are using Ubuntu 16.10 or newer, then you can easily install Python 3.6 with the following commands:

```
sudo apt-get update
sudo apt-get install python3.6
```

If you’re using another version of Ubuntu (e.g. the latest LTS release), we recommend using the deadsnakes PPA to install Python 3.6:

```
sudo add-apt-repository ppa:fkrull/deadsnakes
sudo apt-get update
sudo apt-get install python3.6
```

#install pip3#

```
sudo apt-get install python3-pip
```

#install jupyter, use pip if you want to use python2#

```
sudo pip3 install --upgrade pip
sudo pip3 install jupyter
```

#install numpy, matplotlib...#

```
sudo pip3 install numpy
```

#check pip3 list#

```
pip3 list
pip3 show numpy
```

### other 

```
sudo apt-get install htop
```
