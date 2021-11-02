Guide creation date: 30-Oct-2021 
# Step 3: Native Home-Assistant installation on a Raspberry Pi
Native Home Assistant installation on a Raspberry Pi without using the Docker image.

Previous topic: [Step 2: Upgrading Python on the Raspberry Pi.](https://github.com/JurgenVanGorp/Step2-Upgrading-Python-on-a-Raspberry-Pi)

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

**Info** During installation you will get multiple warnings that pip should be upgraded. You may be tempted to indeed upgrade pip to the latest version with 
```
python3 -m pip install --upgrade setuptools pip
python3 -m pip install --upgrade pip
```
This could result in you e.g. having pip version 21.3.1, where Home-Assistant is not yet compatible with this release. 

Install the packages Home-Assistant will need.

```
python -m pip install py-spy
sudo apt install libopenjp2-7 -y
sudo apt install libtiff5 -y
python3 -m pip install -U pip setuptools
sudo apt install libjpeg-dev zlib1g-dev -y
python3 -m pip install Pillow
```

## Install Home-Assistant

Install Development packages
```
sudo apt-get install python3-dev python3-venv python3-pip -y
```

**IMPORTANT**: The following steps need to be executed in the right order.

Create a system account with proper rights, and in its own home directory.

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

Install the wheel package handler. You can find a [thorough explanation of wheels here](https://realpython.com/python-wheels/).

```
python3 -m pip install wheel
```

Install Home-Assistant from scratch.

```
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

If you see errors here, you may want to go over the previous steps again, but do verify online if you see error messages.

If all is well, reboot to make sure that HA starts properly as a system service.

```
sudo reboot
```

## What in case of errors?

The internet will provide a lot of information already. One case of errors could be the trusted networks issue. After a reboot, check the status with:

```
sudo systemctl status homeassistant@homeassistant
```

If you see the following error in the log file:

```
Unable to load auth provider homeassistant: libffi.so.7
```

... try the following. First find the location of your home assistant. if you followed the guide above, this is expected to be at */home/homeassistant/.homeassistant*.

```
sudo find / | grep configuration.yaml
```

This would give you something like:

```
pi@raspberry:~ $ sudo find / | grep configuration.yaml
/home/homeassistant/.homeassistant/configuration.yaml
```

Go into the .homeassistant directory, and edit the configuration.yaml file.

```
cd /home/homeassistant/.homeassistant
sudo nano configuration.yaml
```

Add the following lines at the end of the file. Make sure to update the network address to match your own address.

```
auth_providers:
  - type: homeassistant
  - type: trusted_networks
    trusted_networks:
      - 192.168.1.0/24
```

## Finally

Under the assumption that you installed Home Assistant on a RPi with e.g. IP address 192.168.1.200, you can verify that it is properly working with a web browser on your computer. The default port for HA is 8123.

http://192.168.1.200:8123

The default logon is *root* without a password.

Continue with the standard HA configuration that you can find on [The Home Assistant website](https://www.home-assistant.io/docs/configuration/).

Next topic: [Step 4: Doing Multi I/O Control with the Raspberry Pi and an MCP23017.](https://github.com/JurgenVanGorp/Step4-MCP23017-multi-IO-control-on-a-Raspberry-Pi-with-I2C)

