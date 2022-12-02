title: Keep only one Gnome-Terminal instance (Ubuntu 22.04 with Xorg)
date: 2022/12/2 13:52:00
categories:
- Linux
toc: true
---

**Note: Unfortunately, I haven't managed a way to activate gnome-terminal window in Wayland. You can use this script in Wayland, but the terminal will not come to foreground.**

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
