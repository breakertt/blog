title: Keep only one gnome terminal window (Ubuntu 22.04)
date: 2022/12/2 13:52:00
categories:
- Linux
toc: true
---

1. Create a script at `~/bin/single_gnome_terminal.sh`

```bash
#!/bin/sh
SERVICE='gnome-terminal'

if ps ax | grep -v grep | grep $SERVICE > /dev/null
then
  wmctrl -xa $SERVICE
else
  $SERVICE
fi
```

2. Enable execution permission

```bash
chmod +x ~/bin/single_gnome_terminal.sh`
```

3. Install `wmctrl`

```bash
sudo apt-get install wmctrl
```

4. Change default terminal

```bash
gsettings set org.gnome.desktop.default-applications.terminal exec single_gnome_terminal.sh
```

References:
1. https://askubuntu.com/a/1194659
2. https://askubuntu.com/a/87109
