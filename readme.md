# PiCamera Notes

Here are some notes on how I've been setting up Pi Cameras.
I have been using the software found [here](https://github.com/silvanmelchior/RPi_Cam_Web_Interface), but it doesn't seem to work with newer versions (greater than bullseye) of Raspberry Pi OS, and it doesn't work with 64-bit systems.

So, I'm going to try and use the [Picamera2 library](https://datasheets.raspberrypi.com/camera/picamera2-manual.pdf).

## Flash the Pi

Use Raspberry Pi Imager and preconfigure it with the WiFi and user info.

## Configure the Pi

Make sure it's connected to the internet and updated (you can configure the connection to WiFi using the Raspberry Pi Imager and setting the custom settings):

```
sudo apt-get update && apt-get upgrade
```

Enable SSH:

```
sudo raspi-config
> System Options -> SSH -> Enable
```

Set a static IP address:

```
sudo nmtui
> Edit a connection
  - Select the WiFi connection
  - Change the IPv4 Connection from Automatic to Manual
  - Set the IP address to something like '192.168.0.55/24'
  - Set the Gateway (or default route) to '192.168.0.1' (you can see this from the connection of another computer on the network using DHCP)
  - Set the DNS servers to '75.75.75.75' and '75.75.76.76' (again, look this up from another computer on the WiFi)
  - Make sure 'Automatically connect' is checked
  - Go back and select quit/ok to save

# Now we'll turn off and on the interface to see if our settings worked
nmcli con show
nmcli con down WiFi
nmcli con up WiFi
ip a
# Verify our Wifi address is 192.168.0.55
# See if we have internet: ping 8.8.8.8
```

## Setup Video Streaming Service

The Python code for creating a streaming server was mostly just copied from here: https://github.com/raspberrypi/picamera2/blob/main/examples/mjpeg_server.py

This only allows one browser to be connected at a time... Not ideal.

After copying `mjpeg_server.py` and `py4cam.service` to the Pi, do the following to create the service for systemd to start the server and keep it running upon boot:

```
sudo mkdir /opt/py4cam
sudo mv mjpeg_server.py /opt/py4cam
sudo mv py4cam.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable py4cam
sudo systemctl start py4cam

# View the logs
journalctl -fu py4cam.service
```

## View the Webpage

You should now be able to open a browser and go to the following page to see the camera feed:

[192.168.0.55:8000](192.168.0.55:8000)
