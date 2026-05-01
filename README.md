# CamAPI for PiCamera2

## Overview

CamAPI for picamera2 is a headless HTTP API for the Raspberry Pi camera module, built on the picamera2 Python library and using Flask. This project provides API endpoints to configure camera settings, capture photos, stream video, and manage images without any frontend UI.

## Features

- **HTTP Camera Control:** Configure camera settings via API calls, including rotation, exposure, white balance, and sensor mode.
- **Capture Photos:** Take still images through POST requests and store them in the gallery.
- **Image Management:** List, delete, download, and edit images through API endpoints.
- **Streaming Support:** Use the video and snapshot URLs directly in applications such as OBS, VLC, Home Assistant, and OctoPrint.
- **Multi-Camera Support:** Supports multiple Pi cameras on Raspberry Pi 5 and returns per-camera metadata.

## Headless API Upgrade

This project is now a headless API server instead of a UI-driven web application. It exposes camera controls, image capture, streaming, and gallery management through HTTP endpoints only, making it ideal for building a separate frontend.

- **Project Name:** Renamed from CamUI to CamAPI for picamera2
- **Headless First:** No frontend templates or UI assets are required by the server.
- **API-Driven:** All interactions are available through JSON endpoints and file downloads.
- **Streaming and Snapshots:** Video streams and snapshots remain available via dedicated endpoints.
- **Metadata Support:** Sensor metadata is accessible through API calls.

## What is Picamera2 Library

This project utilizes the Picamera2 Python library. Picamera2 is the libcamera-based replacement for Picamera which was a Python interface to the Raspberry Pi's legacy camera stack. 
For more information about Picamera2, visit [Picamera2 GitHub Repository](https://github.com/raspberrypi/picamera2).

## Getting Started

Note: Please also see [Compatibility](#compatibilty) below

### Preinstalls / Dependencies

As of September 2024 the Bookworm version of Raspberry Pi OS (Desktop) has the required dependencies preinstalled, so you can skip to **Installation** below. If you are using the Lite version you will need to install the following:
- [flask](https://flask.palletsprojects.com/en/3.0.x/installation/#install-flask)
- [Picamera2](https://github.com/raspberrypi/picamera2)

### Installation

1. Update Raspberry Pi OS: 
```bash
sudo apt update && sudo apt upgrade -y
```
2. Clone the repository to your Raspberry Pi:
```bash
git clone https://github.com/monkeymademe/picamera2-CamAPI.git
```
3. Enter the directory:
```bash
cd CamAPI
```
4. Run the application and access the API through your browser or API client.
```bash
python app.py
```
5. From your browser or API client, on a device connected to the same network, use the following base URL: `http://**Your IP**:8080/`

## Running as a service 

- Run the following command and note down the location for python which should look like `/usr/bin/python`: `which python`
- Go to `/etc/systemd/system/`
- Create and edit the service file `sudo nano picamera2-camapi.service`
- Paste this into the file, replacing the `ExecStart` path with your Python executable and cloned repo location:
  
```bash
[Unit]
Description=CamAPI Server
After=network.target
[Service]
Type=simple
ExecStart=/usr/bin/python /home/pi/CamAPI/app.py
Restart=always
[Install]
WantedBy=multi-user.target
```
- Save the file
- Run `sudo systemctl start picamera2-camapi.service` to start the service 
- Run `sudo systemctl status picamera2-camapi.service` to check the service
- Run `sudo systemctl enable picamera2-camapi.service` to enable it at boot
  
## Compatibilty

- **Raspberry Pi OS / Debian**

Please be aware that due to dependencies on newer versions of Picamera2 (see below) and Libcamera this project only works on Raspberry Pi OS Bookworm (or newer). Issues have been reported with older versions (e.g. Bullseye) not functioning due to libcamera no longer being updated on older versions of the Raspberry Pi OS. The recommendation, even on older Pi's, is to use Bookworm (or newer).

- **Picamera2**

Please check [Picamera installation Requirements](https://github.com/raspberrypi/picamera2?tab=readme-ov-file#installation). Your operating system may not be compatible with Picamera2.

There has been some reported issues with the PiCamera2 on older Raspberry Pi's: ```OSError: [Errno 12] Cannot allocate memory``` https://github.com/raspberrypi/picamera2/issues/972#issuecomment-1980573868

- **Hardware**

Tested on Raspberry Pi Camera Module v3 which has focus settings. v1 is untested but if you see any bugs please post an issue. v2 and HQ has been tested settings like Auto focus that are unique to Camera Module v3 are filtered and removed when an older camera is used.

Raspberry Pi Compatibilty: 

- Pi 5 (8GB): Perfect
- Pi 5 (4GB): Perfect
- Pi 4 (4GB): Perfect
- Pi 3B: Perfect
- Pi Zero v2: Slower lower frame rate on feed but very useable
- Pi Zero v1: Untested
- Older Pi's (Model A, 2B etc): Untested but expected not to work well.
