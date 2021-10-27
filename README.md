# DRAFT - DO NOT USE YET

# Native Home-Assistant installation on a Raspberry-Pi
Native Home Assistant installation on a Raspberry Pi without using the Docker image.

## ASSUMPTIONS

* You have some experience with Raspberry Pi (further also abbreviated to RPi because I'm lazy). Installing Home Assistant with Docker is easier, but you have less control.
* Beware: this installation instruction was written on 30-Oct-2021. Time change, and this information may be outdated when you read it.
* This instruction was created using a Raspberry Pi 3 B+
* You have the Raspberry installed native with e.g. the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and the Raspbian OS. 
  * For performance reasons I recommend to install the Raspberry Pi OS Lite (i.e. without the graphical interface)

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

... and configure a fixed IP address by adding (or updating) the following lines.

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

### Stop IPv6 if you don't want to use it

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

# Upgrade Python



# THE REST IS NOT READY YET, BUT COMING SOON


