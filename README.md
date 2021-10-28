Guide creation date: 27-Oct-2021 
# Step 3: Native Home-Assistant installation on a Raspberry Pi
Native Home Assistant installation on a Raspberry Pi without using the Docker image.

## PREREQUISITES

Before starting here, make sure you have a Raspberry Pi with a basic Raspbian OS installed. You can follow [Step 1: setting up the Raspberry Pi](https://github.com/JurgenVanGorp/Step1-Setting-up-the-Raspberry-Pi) if you still need to do that.

## ASSUMPTIONS

* You have some experience with Raspberry Pi (further also abbreviated to RPi because I'm lazy). Installing Home Assistant (further also abbreviated to HH for the same reason) [with Docker is easier to set up](https://www.home-assistant.io/installation/raspberrypi/). However, we will install our own drivers later on.
* Beware: this installation instruction was updated on 30-Oct-2021. Times change, and this information may be outdated when you read it.
* This instruction was created using a Raspberry Pi 3 B+

## Why installing Home-Assistant Native on a Raspberry Pi?

Know why you want to install Home-Assistant native, it is not the easiest thing to do. I did it because I needed more control for directly driving an MCP23017 over I2C. I had instabilities when using the Docker version with the built-in MCP23017 library.
* If you prefer doing it the easy way: [the Home Assistant home](https://www.home-assistant.io/installation "The one and only Home Assistant") provides an excellent description of how to do this.
* A bit more complex instruction [can also be found in Github.io](https://sensorsiot.github.io/IOTstack/Containers/Home-Assistant/)

## Setting up the Raspberry Pi

If your RPi is already fully up and running and on the network, you may want to skip this step.

It is assumed that you already brought your RPi to the latest OS and software state.

```
sudo apt update -y
sudo apt upgrade -y
sudo apt --autoremove -y
```

### Basic Settings

Start the RPi configuration screen.

```
sudo raspi-config
```

Then configure the following settings (at will):
* hostname
* Enable SSH
* Change the Pi default password
* Enable I2C
* Localization
* Keyboard
* Advanced --> Expand File System to use the full SD Card

Exit and reboot if needed.

```
sudo reboot
```

### Fixed IP

Verify if dhcpcd already works.

```
sudo service dhcpcd status
```

ONLY if it is not configured for automatic boot yet, you can set it to automatic boot.

```
sudo service dhcpcd start
sudo systemctl enable dhcpcd
```

Edit the configuration for the network.

```
sudo nano /etc/dhcpcd.conf
```

... and configure a fixed IP address by adding (or updating) the following lines. Change to your network configuration, of course.

```python
# THIS IS ONLY AN EXAMPLE
interface eth0
static ip_address=192.168.1.200/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8

interface wlan0
static ip_address=192.168.1.201/24
#static routers=192.168.1.1
#static domain_name_servers=192.168.1.1 8.8.8.8
```

type Ctrl-S and Ctrl-X to save and exit.

### You can disable IPv6 if you don't want to use it

Edit the sysctl.

```
sudo nano /etc/sysctl.conf
```

Add the following lines.

```python
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
net.ipv6.conf.eth0.disable_ipv6=1
```

type Ctrl-S and Ctrl-X to save and exit.

## Upgrade Python

The native Raspberry Pi OS may come with an older Python version, while Home-Assistant is using more recent versions. You will need to upgrade Python, or the HA installation will fail.

**Beware:** Your debugging skills may be tested here. Installing the newest version of Python requires you to compile with the right libraries installed.

### Pre-Install necessary packages

Install the following packages that are required by Python when compiling.

```
sudo apt install libssl-dev
sudo apt-get install build-essential libssl-dev libffi-dev python-dev
sudo apt-get install libbz2-dev liblzma-dev libsqlite3-dev libncurses5-dev libgdbm-dev zlib1g-dev libreadline-dev libssl-dev tk-dev uuid-dev libffi-dev
sudo apt-get install libgdbm-compat-dev
```

**Remember**: you may need to install more (or different versions) of the packages. Make sure to verify the logs for warnings and errors when doing the compilation.

### Get the latest Python 3 version

**KUDOS**: I freely used information [from this forum](https://www.raspberrypi.org/forums/viewtopic.php?p=1761359#p1761359) to create the following step-by-step guide.

Download the latest Python 3 (!) version from this location: [https://www.python.org/downloads/source/]. At time of writing this was version 3.10.0.

**Important**: Make sure to download the tarball.

**Beware**: Change the Python version to the one you want to download. The number in the line below may therefore need to be changed.

```
wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
```

### Compile and install Python

Unpack the tarball. Again, change the version number if you are installing a different version as the below.

```
tar xf Python-3.10.0.tgz
```

Go into the directory. Optimize for your system, start the compilation and install.

```
cd Python-3.10.0
./configure --enable-optimizations
make -j 4
sudo make install
```

**Beware**: the compilation takes *really* long. Go for a coffee.

### Set the new Python 3 version as the default

**Kudos**: It took me some time to find this out. My gratitude goes to [the information here](https://linuxconfig.org/how-to-change-from-default-to-alternative-python-version-on-debian-linux) and [some more here](https://stackoverflow.com/questions/62275714/how-to-change-the-default-python-version-in-raspberry-pi). 

There was already a Python installed on your RPi. When "just" starting Python, it will still use the old version. This now needs to be updated.

**Attention**: Don't forget to change the "python3.10" to the version you are installing if it is different.

```
sudo update-alternatives  --install  /usr/bin/python  python  /usr/local/bin/python3.10   1
```

Verify the Python versions with.

```
python --version
python3 --version
pip3 --version
```

All three commands should refer to the new versions you have installed. If that is the case ... then congrats: you have upgraded to the newest Python version.

And just to be on the safe side ...

```
sudo apt update -y
sudo apt upgrade -y
sudo reboot
```


## Home Assistant Installation

### Pre-Install necessary packages

Home Assistant needs several libraries installed upfront for a smooth installation. You may detect more, but the below list is already a good start.

```
python -m pip install py-spy
sudo apt-get install libopenjp2-7
sudo apt-get install libtiff5
python3 -m pip install -U pip setuptools
sudo apt-get install libjpeg-dev zlib1g-dev
python3 -m pip install Pillow
```

### Install Home-Assistant

**IMPORTANT**: The following steps need to be executed in the right order. Don't skip a step, and be carefull

One more update, to be on the safe side.

```
sudo apt-get update && sudo apt-get upgrade -y
```

Install Development packages

```
sudo apt-get install python3-dev python3-venv python3-pip libffi-dev libssl-dev -y
```

Create a system account with proper rights, with its own home directory

```
sudo useradd -rm homeassistant -G dialout,gpio,i2c
cd /srv
sudo mkdir homeassistant
sudo chown homeassistant:homeassistant homeassistant
```

Create a virtual Python3 environment that will be used to run HA.

```
sudo -u homeassistant -H -s
cd /srv/homeassistant/
python3 -m venv .
source bin/activate
```

Install Home-Assistant from scratch.

```
python3 -m pip install wheel
pip3 install homeassistant
```

Exit the virtual environment.

```
exit
```

### Set HA to automatic boot

Create a Home Assistant systemd service with nano.

```
sudo nano /etc/systemd/system/homeassistant@homeassistant.service
```

In the .service file, enter the following lines.

```python
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/srv/homeassistant/bin/hass -c "/home/%i/.homeassistant"

[Install]
WantedBy=multi-user.target
```

type Ctrl-S and Ctrl-X to save and exit.

Enable and start the HA service.

```
sudo systemctl --system daemon-reload
sudo systemctl enable homeassistant@homeassistant
sudo systemctl start homeassistant@homeassistant
```

Verify that HA has properly started with.

```
sudo systemctl status homeassistant@homeassistant
```

If you see errors here, you may want to go over the previous steps again, but do verify if there is an error message.

If all is well, reboot to make sure that HA starts properly as a system service.

```
sudo reboot
```

Under the assumption that you installed Home Assistant on a RPi with IP address 192.168.1.200, you can verify that it is properly working with a web browser on your computer. The default port for HA is 8123.

http://192.168.1.200:8123

The default logon is *root* without a password.

Continue with the standard HA configuration that you can find on [The Home Assistant website](https://www.home-assistant.io/docs/configuration/).

--- end of file
