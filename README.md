Guide creation date: 27-Oct-2021 
# Step 3: Native Home-Assistant installation on a Raspberry Pi
Native Home Assistant installation on a Raspberry Pi without using the Docker image.

## PREREQUISITES

1. Before starting here, make sure you have a Raspberry Pi with a basic Raspbian OS installed. You can follow [Step 1: setting up the Raspberry Pi](https://github.com/JurgenVanGorp/Step1-Setting-up-the-Raspberry-Pi) if you still need to do that.
2. Make sure to have the latest Python 3 version installed. If you haven't done that yet, you may want to follow [Step 2: Upgrading Python on a Raspberry Pi](https://github.com/JurgenVanGorp/Step2-Upgrading-Python-on-a-Raspberry-Pi) first.

## Introduction

* You have some experience with Raspberry Pi (further also abbreviated to RPi because I'm lazy). Installing Home Assistant (further also abbreviated to HH for the same reason) [with Docker is easier to set up](https://www.home-assistant.io/installation/raspberrypi/). However, we will install our own drivers later on.
* Beware: this installation instruction was updated on 30-Oct-2021. Times change, and this information may be outdated when you read it.
* This instruction was created using a Raspberry Pi 3 B+

## Why installing Home-Assistant Native on a Raspberry Pi?

Know why you want to install Home-Assistant native, it is not the easiest thing to do. I did it because I needed more control for directly driving an MCP23017 over I2C. I had instabilities when using the Docker version with the built-in MCP23017 library.
* If you prefer doing it the easy way: [the Home Assistant home](https://www.home-assistant.io/installation "The one and only Home Assistant") provides an excellent description of how to do this.
* A bit more complex instruction [can also be found in Github.io](https://sensorsiot.github.io/IOTstack/Containers/Home-Assistant/)

## Pre-Install necessary packages

Home Assistant needs several libraries installed upfront for a smooth installation. You may detect more, but the below list is already a good start.

```
python -m pip install py-spy
sudo apt-get install libopenjp2-7
sudo apt-get install libtiff5
python3 -m pip install -U pip setuptools
sudo apt-get install libjpeg-dev zlib1g-dev
python3 -m pip install Pillow
```

## Install Home-Assistant

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

## Set Home-Assistant to automatic boot

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

## Finally

Under the assumption that you installed Home Assistant on a RPi with e.g. IP address 192.168.1.200, you can verify that it is properly working with a web browser on your computer. The default port for HA is 8123.

http://192.168.1.200:8123

The default logon is *root* without a password.

Continue with the standard HA configuration that you can find on [The Home Assistant website](https://www.home-assistant.io/docs/configuration/).

--- end of file
