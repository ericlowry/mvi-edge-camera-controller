# mvi-edge-camera-controller

The application lets you use a rapberry pi with an attached camera as an input device for Maximo Visual Inspection Edge.

## Prerequisites

### MVI Edge environment

In MVI Edge, create an "Image Folder" input device.

Create a station and a "Single Shot" triggered inspection using your new input device.  Name your trigger something like "triggers/<device-name>


### Raspberry Pi 4

A Raspberry Pi 4 running 32bit Raspberry Pi OS (picamera) with an attached camera, like the Pi Camera Module 2 or the Pi HQ Camera.

```
sudo apt-get install python3-picamera
sudo apt-get install python3-dotenv
sudo apt-get install python3-paho-mqtt
```

## Installation

Step 1: Grab the code

```
git clone git@github.ibm.com:elowry/mvi-edge-camera-controller.git
cd mvi-edge-camera-controller
```

Step 2: Get a copy of your MVI Edge environment's certificat:

```
sftp mvi-edge:/opt/ibm/vision-edge/volume/run/var/config/.ssl/visionedgeca.crt
```

Step 3: Configure the controller for your MVI Edge instance

```
cp sample.env .env
nano .env
```

Set `MVI
