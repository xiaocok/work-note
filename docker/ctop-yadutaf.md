
地址：

https://github.com/yadutaf/ctop

要求：

Python >= 2.6 or Python >= 3.0

### 直接下载安装
```shell
curl -sSl https://raw.githubusercontent.com/yadutaf/ctop/master/cgroup_top.py > /opt/ctop && python /opt/ctop
```

### Python安装
```shell
pip install ctop
ctop
```

### 直接下载安装
```shell
wget https://raw.githubusercontent.com/yadutaf/ctop/master/cgroup_top.py -O ctop
chmod +x ctop
./ctop
```

### docker运行
```shell
docker pull yadutaf/ctop
docker run --volume=/sys/fs/cgroup:/sys/fs/cgroup:ro --volume=/var/run/docker.sock:/var/run/docker.sock -it --rm yadutaf/ctop
# Optionally, to resolve uids to usernames, add '--volume /etc/passwd:/etc/passwd:ro'
```

### 命令行使用说明
命令行参数：
```text
Monitor local cgroups as used by Docker, LXC, SystemD, ...

Usage:
  ctop [--tree] [--refresh=<seconds>] [--columns=<columns>] [--sort-col=<sort-col>] [--follow=<name>] [--fold=<cgroup>, ...] [--type=<container type>, ...]
  ctop (-h | --help)

Options:
  --tree                 Show tree view by default.
  --fold=<name>          Start with <name> cgroup path folded
  --follow=<name>        Follow/highlight cgroup at path.
  --type=TYPE            Only show containers of this type
  --refresh=<seconds>    Refresh display every <seconds> [default: 1].
  --columns=<columns>    List of optional columns to display. Always includes 'name'. [default: owner,processes,memory,cpu-sys,cpu-user,blkio,cpu-time].
  --sort-col=<sort-col>  Select column to sort by initially. Can be changed dynamically. [default: cpu-user]
  -h --help              Show this screen.
```

快捷键：
```text
press p to toggle/pause the refresh and select text.
press f to let selected line follow / stay on the same container. Default: Don't follow.
press q or Ctrl+C to quit.
press F5 to toggle tree/list view. Default: list view.
press ↑ and ↓ to navigate between containers.
press + or - to toggle child cgroup folding
click on title line to select sort column / reverse sort order.
click on any container line to select it.
```

容器的快捷键：(Docker, LXC and OpenVZ)
```text
press a to attach to console output.
press e to open a shell in the container context. Aka 'enter' container.
press s to stop the container (SIGTERM).
press k to kill the container (SIGKILL).
press c to checkpointing the container(OpenVZ only now - run 'vzctl chkpnt CTID')
```
