# Dev Environment

## RealVNC

```bash
cd ~/Downloads

wget https://downloads.realvnc.com/download/file/vnc.files/VNC-Server-6.9.0-Linux-x64.deb?_ga=2.8804277.1474791596.1646795775-1964236855.1646795775

sudo dpkg -i VNC-Server-*-Linux-x64.deb
sudo apt install -f

sudo systemctl start vncserver-x11-serviced

sudo systemctl enable vncserver-x11-serviced
```

