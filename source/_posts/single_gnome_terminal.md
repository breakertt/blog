title: Keep only one Gnome-Terminal instance (Ubuntu 22.04)
date: 2022/12/2 13:52:00
categories:
- Linux
toc: true
---

# Method 1 - Remap gnome dock shortcut

1. Add gnome-terminal into dock (favortie), I move it to the third application, please change the number **3** in following article to the number you used.
2. Disable terminal shortcut in **Settings/Kebyaord/Keyboard Shortcuts**
3. Reset the shortcut to for start dock applications `gsettings set org.gnome.shell.keybindings switch-to-application-3 "['<Ctrl><Alt>T']"`

# Method 2 (Xorg only) - Reset gnome default terminal

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
3. https://unix.stackexchange.com/a/510376
