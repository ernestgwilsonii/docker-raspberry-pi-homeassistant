##########################################
# docker-raspberry-pi-homeassistant      #
# Home Assistant Docker for Raspberry Pi # 
##########################################

# REF: https://www.home-assistant.io/
# REF: https://www.home-assistant.io/docs/installation/docker/

###############################################################################
# Download - https://hub.docker.com/r/homeassistant/raspberrypi3-homeassistant/tags
sudo docker pull homeassistant/raspberrypi3-homeassistant:0.89.1
sudo docker images
###############################################################################


###############################################################################
# First time setup #
####################
# Create bind mounted directory
sudo mkdir -p /opt/homeassistant


#############################
# Initialize via "docker run"
sudo docker run --init -d --name="home-assistant" -v /opt/homeassistant/:/config -v /etc/localtime:/etc/localtime:ro -p 8123:8123 homeassistant/raspberrypi3-homeassistant

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
