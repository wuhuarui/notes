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
# Edit the configuration file
sudo vim /etc/shadowsocks-libev/config.json

# Edit the default configuration for debian
sudo vim /etc/default/shadowsocks-libev

# Start the service
sudo /etc/init.d/shadowsocks-libev start    # for sysvinit, or
sudo systemctl start shadowsocks-libev      # for systemd
```

### google chrome installation

```
sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
sudo apt-get update
sudo apt-get install google-chrome-stable
```
