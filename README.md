# Headless Nicotine+ with VNC for Raspberry Pi (ARM64) and AMD64 Processors

Yet another [realies/soulseek-docker](https://github.com/realies/soulseek-docker ) fork! 
kokmok made a great RPI fork, but unfortunately hasn't updated it in 5 years, so that's where this repo comes in! Latest image uses Nicotine+ v3.3.2, and the Dockerfile allows for rebuilding with future versions. Huge thanks again to [realies](https://github.com/realies) for all the hard work of getting soulseek to work over noVNC, and [kokmok](https://github.com/kokmok/rpi-nicotine-novnc) for giving me clues to get this working.



![image](https://github.com/hockeygoalie35/nicotine-novnc-rpi/assets/7758029/78c27ee5-691d-44f7-b254-73de58a0aa28)




## Setup

Here's a recommended file structure on your raspberry pi:

```
home/pi/Appdata
  -> nicotine
     -->downloads
     -->incomplete  # Optional
     -->received    # Optional
     -->shares
```
Here's a bash script that will do just that in `home/pi/`
```bash
mkdir -p /home/pi/Appdata/nicotine/data
mkdir -p /home/pi/Appdata/nicotine/downloads
mkdir -p /home/pi/Appdata/nicotine/incomplete
mkdir -p /home/pi/Appdata/nicotine/received
mkdir -p /home/pi/Appdata/nicotine/shares
```

# Container Setup
## Docker Compose
Now edit docker-compose.yml, changing the `- /path/to/Nicotine/data` to your newly created data directory, etc.
If needed, change the UID and GID to match your user. It is advised not to run this container as root.
Then in the project directory, run `docker compose up -d`

## Portainer
Copy the contents of docker-compose.yml and paste it into a stack, changing the paths and UID/GID as above. 

*Make sure to use the correct image tag!*
```yaml
services:
  nicotine:
    image: hockeygoalie35/nicotine-novnc-rpi:arm64 #Or armv7 for 32bit
    container_name: nicotine
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
    volumes:
      - /home/pi/Appdata/nicotine/data:/data
      - /home/pi/Appdata/nicotine/downloads:/data/.local/share/nicotine/downloads
      - /home/pi/Appdata/nicotine/incomplete:/data/.local/share/nicotine/incomplete
      - /home/pi/Appdata/nicotine/received:/data/.local/share/nicotine/received
      - /home/pi/Appdata/nicotine/shares:/data/shares
    ports:
      - 6080:6080
    restart: unless-stopped
```


Once the container is up and running, in your web browser go to RPIHOSTNAME:6080 and you should be greeted with Nicotine+

# *Important*
After the first boot up, Nicotine will create a default config file, which sets header_bar to TRUE. For whatever reason, this header bar and settings button don't show up in the noVNC window. I've written a python script that monitors the config file and sets header_bar to FALSE, which brings the search bar and other tabs back into the GUI. Once it changes the config, restart the container with `docker restart nicotine`


## Before - Lack of search bar
![image](https://github.com/hockeygoalie35/nicotine-novnc-rpi/assets/7758029/6d00fbb0-9210-4c5a-be16-8d43172c7768)




## Container STDOUT - py script fixing config
![image](https://github.com/hockeygoalie35/nicotine-novnc-rpi/assets/7758029/976245f1-4e31-4d32-b5c7-5870a2baca12)



## After - Search bar restored after restart

![image](https://github.com/hockeygoalie35/nicotine-novnc-rpi/assets/7758029/e8be0d89-9d36-402f-91a1-27d9da7f7019)


If for whatever reason the script doesn't fix it, you can go to: data/.config/nicotine/ and edit config, setting: header_bar = False


## Building from Dockerfile (For devs)
Use the following commands to build it from the Dockerfiles

32-bit:
`docker build --platform linux/arm/v7 -t hockeygoalie35/nicotine-novnc-rpi:armv7 . -f ./Dockerfile-armv7`

64-Bit:
`docker build --platform linux/arm64 -t hockeygoalie35/nicotine-novnc-rpi:arm64 . -f ./Dockerfile-arm64`

AMD64 (Intel/AMD processors)
64-Bit:
`docker build --platform linux/arm64 -t hockeygoalie35/nicotine-novnc-rpi:arm64 . -f ./Dockerfile-arm64`

