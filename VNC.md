# VNC
## Installation
```
sudo apt-get install tigervnc-standalone-server
```

## Configuration
```
vi ~/.vnc/xstartup
```

Contents should look like 
```
#!/bin/sh
p[]
unset SESSION_MANAGER

[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
dbus-launch --exit-with-session gnome-session &
```

## Running

To run vnc use the following command:
```
vncserver -localhost no
```

# Ubuntu Bult in Screen Sharing (VNC)

Go to settings -> sharing -> screen sharing
Turn on screensharing and set a password
Open command prompt and run the following command to disable encryption (we are doing this to allow the vncviewer client on windows to connect as otherwise it has problems)
```
gsettings set org.gnome.Vino require-encryption false
```
ref. https://askubuntu.com/questions/1126714vnc-viewer-unable-to-connect-encryption-issue