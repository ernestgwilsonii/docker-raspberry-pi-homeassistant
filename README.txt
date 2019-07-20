##########################################
# docker-raspberry-pi-homeassistant      #
# Home Assistant Docker for Raspberry Pi # 
##########################################

# Use the latest "Buster" Raspbian Lite OS image on the Raspberry Pi 3B+
# REF: https://www.raspberrypi.org/downloads/raspbian/
# Then setup networking (WiFi or Ethernet) and enable SSH (as typical on any new Pi)


# Connect via SSH
ssh pi@<PiIPAddress>
sudo bash

# Set static IP addresses on LAN and WLAN
LAN_IP=192.168.1.13
WLAN_IP=192.168.1.14
GATEWAY_IP=192.168.1.1
echo " " >> /etc/dhcpcd.conf 
echo "# LAN" >> /etc/dhcpcd.conf 
echo "interface eth0" >> /etc/dhcpcd.conf 
echo "static ip_address=$LAN_IP/24" >> /etc/dhcpcd.conf 
echo "static routers=$GATEWAY_IP" >> /etc/dhcpcd.conf 
echo "static domain_name_servers=8.8.8.8" >> /etc/dhcpcd.conf 
echo " " >> /etc/dhcpcd.conf 
echo "# WLAN" >> /etc/dhcpcd.conf 
echo "interface wlan0" >> /etc/dhcpcd.conf 
echo "static ip_address=$WLAN_IP/24" >> /etc/dhcpcd.conf 
echo "static routers=$GATEWAY_IP" >> /etc/dhcpcd.conf
echo "static domain_name_servers=8.8.8.8" >> /etc/dhcpcd.conf 


# Apply updates
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y
reboot
#rpi-update

# Install some standard packages
sudo apt-get -y install apparmor-utils avahi-daemon dbus gcc git htop jq libffi-dev libssl-dev mc network-manager  portaudio19-dev python-dev
# Note: If you are using Debian 10 "Buster" or newer, AppArmor is enabled by default

# Install Docker on Buster
sudo rm /etc/apt/sources.list.d/docker.list;
curl -sL get.docker.com | sed 's/9)/10)/' | sh
sudo usermod -aG docker pi
# Log out of SSH and reconnect:
docker --version

# Use the "test" version (like 19.03.0-rc3) of Docker instead of "stable" (18.09.0) - Another option may be "nightly"
#sudo bash
#echo 'deb [arch=armhf] https://download.docker.com/linux/raspbian stretch test' > /etc/apt/sources.list.d/docker.list
#apt-get update
#apt-cache madison 'docker-ce'
#apt-get upgrade
#sudo reboot
#docker --version

# Install docker compose
sudo apt-get -y install python-pip
pip install --upgrade docker-compose

# Initialize Docker Swarm mode
sudo docker swarm init --advertise-addr 192.168.1.13

# Create a place to save local data
sudo bash
mkdir -p /opt/hassio

# Create a symlink so hassio documentation maps up to /usr/share/hassio
mkdir -p /usr/share/hassio
#ln -s /opt/hassio /usr/share/hassio
ln -s /opt/hassio /usr/share

# Install Home Assistant https://www.home-assistant.io
# Use the Hass.io system https://www.home-assistant.io/hassio/ so you get add-ons!
# Docker container "hassio" installer for a generic Linux system!
# https://github.com/home-assistant/hassio-installer
curl -sL https://raw.githubusercontent.com/home-assistant/hassio-installer/master/hassio_install.sh | bash -s -- -d /opt/hassio -m raspberrypi3

# Verify that the hassio supervisor container started (it will download the homeassistant container)
docker ps
ls -alF /opt/hassio

# Verify that the hassio supervisor (hassio docker supervisor) is running
systemctl status hassio-supervisor.service
cat /etc/systemd/system/hassio-apparmor.service
#systemctl stop hassio-supervisor.service
#systemctl disable hassio-supervisor.service

# WARNING: Do not delete the hassio related containers by mistake when using: sudo docker system prune -af
# Do create a filter to protect from accidental deletion:
sudo mkdir -p ~/.docker
sudo echo '{"pruneFilters":["label!=homeassistant","label!=hassio_supervisor","label!=addon*"]}' > ~/.docker/config.json

# Buster updates on Raspberry Pi
apt-get --allow-releaseinfo-change update
apt-get update
apt-get upgrade

# Open a browser and setup Home Assistant
# http://YourPiIpAdress:8123


####################
# Hass.io Add-ons! #
####################

# Create basic setup for MQTT
mkdir -p /usr/share/hassio/share/mosquitto
echo "acl_file /share/mosquitto/accesscontrollist" > /usr/share/hassio/share/mosquitto/acl.conf
echo "user hassio" > /usr/share/hassio/share/mosquitto/accesscontrollist
echo "topic readwrite #" >> /usr/share/hassio/share/mosquitto/accesscontrollist
echo "" >> /usr/share/hassio/share/mosquitto/accesscontrollist
echo "user nodered" >> /usr/share/hassio/share/mosquitto/accesscontrollist
echo "topic readwrite #" >> /usr/share/hassio/share/mosquitto/accesscontrollist
cat /usr/share/hassio/share/mosquitto/accesscontrollist

# Mosquitto MQTT broker
# REF: https://www.home-assistant.io/addons/mosquitto/
# Config (for MQTT in the Hass.io add-on web interface)
{
  "logins": [
    {
      "username": "hassio",
      "password": "ThePasswordGoesHere"
    },
    {
      "username": "nodered",
      "password": "ThePasswordGoesHere"
    }
  ],
  "anonymous": false,
  "customize": {
    "active": true,
    "folder": "mosquitto"
  },
  "certfile": "fullchain.pem",
  "keyfile": "privkey.pem"
}

# Node-RED
# REF: https://github.com/hassio-addons/addon-node-red
# Config (for Node-RED in the Hass.io add-on web interface)
{
  "credential_secret": "ThePasswordGoesHere",
  "dark_mode": false,
  "http_node": {
    "username": "nodered",
    "password": "ThePasswordGoesHere"
  },
  "http_static": {
    "username": "nodered",
    "password": "ThePasswordGoesHere"
  },
  "ssl": false,
  "certfile": "fullchain.pem",
  "keyfile": "privkey.pem",
  "require_ssl": true,
  "system_packages": [],
  "npm_packages": [
    "node-red-admin",
    "node-red-contrib-alexa-local",
    "node-red-contrib-function-npm",
    "node-red-contrib-slack"
  ],
  "init_commands": []
}

###############################################################################
