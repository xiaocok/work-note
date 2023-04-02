
地址：

https://github.com/bcicen/ctop

##　安装

### Debian/Ubuntu
```yaml
echo "deb http://packages.azlux.fr/debian/ buster main" | sudo tee /etc/apt/sources.list.d/azlux.list
wget -qO - https://azlux.fr/repo.gpg.key | sudo apt-key add -
sudo apt update
sudo apt install docker-ctop
```

### Arch
ctop is available for Arch in the [AUR](https://aur.archlinux.org/packages/ctop-bin/)

### Linux (Generic)
```shell
sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.5/ctop-0.7.5-linux-amd64 -O /usr/local/bin/ctop
sudo chmod +x /usr/local/bin/ctop
```

### OS X
```shell
brew install ctop
```
or
```shell
sudo curl -Lo /usr/local/bin/ctop https://github.com/bcicen/ctop/releases/download/v0.7.5/ctop-0.7.5-darwin-amd64
sudo chmod +x /usr/local/bin/ctop
```

### Docker
```shell
docker run --rm -ti \
  --name=ctop \
  --volume /var/run/docker.sock:/var/run/docker.sock:ro \
  quay.io/vektorlab/ctop:latest
```

### 运行说明：
执行参数
```text
-a	show active containers only
-f <string>	set an initial filter string
-h	display help dialog
-i	invert default colors
-r	reverse container sort order
-s	select initial container sort field
-v	output version information and exit
```

快捷键：
```text
<ENTER>	Open container menu
a	    Toggle display of all (running and non-running) containers
f	    Filter displayed containers (esc to clear when open)
H	    Toggle ctop header
h	    Open help dialog
s	    Select container sort field
r	    Reverse container sort order
o	    Open single view
l	    View container logs (t to toggle timestamp when open)
e	    Exec Shell
c	    Configure columns
S	    Save current configuration to file
q	    Quit ctop
```