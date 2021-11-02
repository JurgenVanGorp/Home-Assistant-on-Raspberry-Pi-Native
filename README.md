Guide creation date: 30-Oct-2021 
# Step 2: Native Home-Assistant installation on a Raspberry Pi
Native Home Assistant installation on a Raspberry Pi without using the Docker image.

Previous topic: [Step 1: Initial setup of the Raspberry Pi.](https://github.com/JurgenVanGorp/Setting-up-a-Raspberry-Pi)

## PREREQUISITES

* Before starting here, make sure you have a Raspberry Pi with a basic Raspbian OS installed. You can follow [Step 1: setting up the Raspberry Pi](https://github.com/JurgenVanGorp/Step1-Setting-up-the-Raspberry-Pi) if you still need to do that.

* It is expected that you have some experience with Raspberry Pi (further also abbreviated to RPi) and that you have the intention to install Home Assistant native. Installing Home Assistant (further also abbreviated to HH) [with Docker is easier to set up](https://www.home-assistant.io/installation/raspberrypi/), but I created this guide because I will install my own drivers later on.

* Beware: this installation instruction was updated on 01-Nov-2021. Times change, and this information may be outdated when you read it.

* This instruction was created using a Raspberry Pi 3 B+, I would expect it to also work on the Pi 4.

## Why installing Home-Assistant Native on a Raspberry Pi?

Know why you want to install Home-Assistant native. The below guideline looks easy at first, but your debugging skills will be tested when e.g. upgrades result in library incompatibilities. If you prefer doing it the easy way: [the Home Assistant home](https://www.home-assistant.io/installation "The excellent Home Assistant docs") provides an excellent description of how to install HA in a docker with minimal effort.

I installed Home-Assistant without docker because I wanted more control for directly driving an MCP23017 over I2C. I had instabilities when using the Docker version with the HA built-in MCP23017 library, resulting in a dangerous situation when switches were turned on, but missed the off command. So, I decided to install Home Assistant without using the docker image, and wrote my own I2C drivers for the MCP23017. It is working for six months now without missing a single command.

## Pre-Install necessary packages

Home Assistant needs several libraries installed upfront for a smooth installation. You may detect more, but the below list is already a good start.

**INFO**: Do mind that Raspbian has two Python versions installed typically: Python V2 V3. Just typing the command *python* may default to Version 2, while Home Assistant needs Version 3. You can enter the command *python --version* to verify the version. In any case, make sure to explictly use *python3* or *pip3* in the commands below to be on the safe side.

Let's start with installing pip3 and updating libraries.

```
sudo apt install python3-pip
sudo apt update -y
sudo apt upgrade -y
sudo apt autoremove -y
```

Install the packages Home-Assistant will need.

```
python3 -m pip install py-spy
sudo apt install libopenjp2-7 -y
sudo apt install libtiff5 -y
python3 -m pip install -U pip setuptools
sudo apt install libjpeg-dev zlib1g-dev -y
python3 -m pip install Pillow
sudo apt-get install python3-dev python3-venv python3-pip -y
```

## Install Home-Assistant

**IMPORTANT**: The following steps need to be executed in the right order. Don't miss a step.

Create a system account with proper rights, and in its own home directory.

```
sudo useradd -rm homeassistant -G dialout,gpio,i2c
cd /srv
sudo mkdir homeassistant
sudo chown homeassistant:homeassistant homeassistant
```

Create a virtual Python3 environment that will be used to run HA isolated.

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

In the .service file, enter (or copy-paste) the following lines.

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

Verify that HA has properly started with:

```
sudo systemctl status homeassistant@homeassistant
```

If you see errors here, you may want to go over the previous steps again, but do verify any error messages.

If all is well, reboot to make sure that HA starts properly as a system service.

```
sudo reboot
```

## Finally

Under the assumption that you installed Home Assistant on a RPi with e.g. IP address 192.168.1.200, you can verify that it is properly working with a web browser on your computer. The default port for HA is 8123.

http://192.168.1.200:8123

At the first logon, Home Assistant will ask you for a Username and Password that will be used for further configuration.

Continue with the standard HA configuration that you can find on [The Home Assistant website](https://www.home-assistant.io/docs/configuration/).

Next topic: [Step 3: Doing Multi I/O Control with the Raspberry Pi and an MCP23017.](https://github.com/JurgenVanGorp/MCP23017-multi-IO-control-on-a-Raspberry-Pi-with-I2C)

