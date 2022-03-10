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

## Visual Studio Code

```bash
# Installing Visual Studio Code with apt
sudo apt update
sudo apt install software-properties-common apt-transport-https wget

wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"

sudo apt install code


# Installing Visual Studio Code as a Snap Package

sudo snap install code --classic


# Use with root

code --user-data-dir="~/.vscode"
```

