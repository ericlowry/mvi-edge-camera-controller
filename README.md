# mvi-edge-camera-controller

The application lets you use a rapberry pi with an attached camera as an input device for Maximo Visual Inspection Edge.

## Prerequisites

### MVI Edge environment

In MVI Edge, create an *Image Folder* input source.  Name your input source device something like "hq-camera-1"

Create a station and a *Single Shot* triggered inspection using your new input device.  Name your trigger something like "triggers/hq-camera-1"

Put your inspection into *collecting mode*.

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

Step 4: Test the camera configuration

```
./picamera-controller -v --preview
```

If the .env configuration file is correct, the camera will start sending preview images to MVI Edge.  You can view the images in MVI Edge by opening the input source and clicking "Test input source".

Press Ctrl-C to stop sending the preview images.

Step 5: Create some pass / fail rules for your inspection

Alert Type: MQTT Checked
Topic: `alerts/hq-camera-1`
Message:
```
{
  "sn": "{{.trigger.sn}}",
  "seq": "{{.trigger.seq}}",
  "rule": "{{.result.Name}}",
  "result": "{{.result.Result}}"
}
```

Step 6: Start the camera controller

```
./picamera-controller
```

Enter a serial # for the image and press Enter (or just press enter and a number will be generated based on the system time).

