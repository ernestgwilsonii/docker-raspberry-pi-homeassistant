##########################################
# docker-raspberry-pi-homeassistant      #
# Home Assistant Docker for Raspberry Pi # 
##########################################

```
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
sudo apt-get -y install apparmor-utils avahi-daemon dbus gcc git htop jq libffi-dev libssl-dev mc apparmor-utils portaudio19-dev python-dev
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


#sudo docker run --init -d --name="home-assistant" -v /opt/homeassistant/:/config -v /etc/localtime:/etc/localtime:ro -p 8123:8123 homeassistant/raspberrypi3-homeassistant:0.94.3

# Create a place to save local data
sudo bash
mkdir -p /opt/homeassistant

# Install Home Assistant https://www.home-assistant.io
# Use the Hass.io system https://www.home-assistant.io/hassio/ so you get add-ons!
# Docker container "hassio" installer for a generic Linux system!
# https://github.com/home-assistant/hassio-installer
curl -sL https://raw.githubusercontent.com/home-assistant/hassio-installer/master/hassio_install.sh | bash -s -- -d /opt/homeassistant -m raspberrypi3
docker ps

# WARNING: Do not delete the hassio related containers by mistake when using: sudo docker system prune -af
# Do create a filter to protect from accidental deletion:
sudo mkdir -p ~/.docker
sudo echo '{"pruneFilters":["label!=homeassistant","label!=hassio_supervisor","label!=addon*"]}' > ~/.docker/config.json

# Open a browser and setup Home Assistant
# http://YourPiIpAdress:8123


# STOP HERE #



#
# WARNING: BELOW THIS POINT IS OLD STYLE AND NOT WORKING!
#

# REF: https://www.home-assistant.io/
# REF: https://www.home-assistant.io/docs/installation/docker/

###############################################################################
# Download - https://hub.docker.com/r/homeassistant/raspberrypi3-homeassistant/tags
sudo docker pull homeassistant/raspberrypi3-homeassistant:0.94.3
sudo docker images
###############################################################################


###############################################################################
# First time setup #
####################
# Create bind mounted directory
sudo mkdir -p /opt/homeassistant


#############################
# Initialize via "docker run"
sudo docker run --init -d --name="home-assistant" -v /opt/homeassistant/:/config -v /etc/localtime:/etc/localtime:ro -p 8123:8123 homeassistant/raspberrypi3-homeassistant:0.94.3

# Verify
sudo docker ps
sudo ls -alF

# Login and setup (create) your local Name, Username and Password
# http://IPAddress:8123

# Stop
sudo docker stop home-assistant
#sudo docker start home-assistant


##########
# Deploy #
##########
# Deploy the stack into a Docker Swarm
sudo docker stack deploy -c docker-compose.yml home-assistant
# sudo docker stack rm home-assistant

# Verify
sudo docker service ls | grep home-assistant
sudo docker service logs -f home-assistant

# http://IPAddress:8123
###############################################################################
