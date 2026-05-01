# CamAPI - Headless Camera API for picamera2

## Overview

CamAPI is a headless REST API for controlling Raspberry Pi cameras using picamera2. It provides endpoints for camera configuration, image capture, video streaming, and image gallery management.

## Base URL
```
http://localhost:8080
```

## Authentication
No authentication required for local development.

## Response Format
All responses are in JSON format unless otherwise specified.

---

## Core Endpoints

### GET /
Get API status and connected cameras.

**Response:**
```json
{
  "status": "ok",
  "version": "2.0.0 - BETA",
  "title": "CamAPI - for picamera2",
  "cameras": [
    {
      "camera_num": 0,
      "camera_info": {
        "Num": 0,
        "Model": "imx219",
        "Is_Pi_Cam": true,
        "Has_Config": false,
        "Config_Location": "default_imx219.json"
      },
      "module_info": {
        "sensor_model": "imx219",
        "sensor_name": "Sony IMX219",
        "resolution": "3280x2464",
        "description": "8MP Raspberry Pi Camera v2"
      }
    }
  ]
}
```

---

## Camera Information

### GET /camera_info_{camera_num}
Get detailed information about a specific camera.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Example:** `GET /camera_info_0`

**Response:**
```json
{
  "camera_num": 0,
  "camera_data": {
    "sensor_model": "imx219",
    "sensor_name": "Sony IMX219",
    "resolution": "3280x2464",
    "description": "8MP Raspberry Pi Camera v2"
  }
}
```

**Error Response (404):**
```json
{
  "error": "Camera not found"
}
```

---

## Camera Control

### GET /camera_{camera_num}
Get camera settings and controls for desktop interface.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Response:**
```json
{
  "camera": {
    "Num": 0,
    "Model": "imx219"
  },
  "settings": {
    "sections": [
      {
        "name": "Exposure",
        "settings": [
          {
            "id": "ExposureTime",
            "name": "Exposure Time",
            "value": 10000,
            "min": 1,
            "max": 100000,
            "enabled": true
          }
        ]
      }
    ]
  },
  "sensor_modes": [
    {
      "size": [3280, 2464],
      "bit_depth": 10
    }
  ],
  "active_mode_index": 0,
  "last_image": "pimage_camera_0_1640995200.jpg",
  "profiles": [
    {
      "filename": "profile1.json",
      "model": "imx219"
    }
  ],
  "mode": "desktop"
}
```

### GET /camera_mobile_{camera_num}
Get camera settings optimized for mobile interface.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Response:** Same as `/camera_{camera_num}` but with `"mode": "mobile"`

---

## Image Capture

### POST /capture_still_{camera_num}
Capture a high-quality still image.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Example:** `POST /capture_still_0`

**Success Response (200):**
```json
{
  "success": true,
  "message": "Image captured successfully",
  "image": "pimage_camera_0_1640995200.jpg"
}
```

**Error Response (404):**
```json
{
  "success": false,
  "message": "Camera not found"
}
```

**Error Response (500):**
```json
{
  "success": false,
  "message": "Failed to capture image"
}
```

### GET /snapshot_{camera_num}
Take a snapshot from the live video feed and return the image file.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Response:** JPEG image file (binary)

---

## Video Streaming

### GET /video_feed_{camera_num}
Get MJPEG video stream from camera.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Response:** `multipart/x-mixed-replace; boundary=frame` stream

### POST /toggle_video_feed
Start or stop video streaming for a camera.

**Request Body:**
```json
{
  "camera_num": 0,
  "enable": true
}
```

**Success Response:**
```json
{
  "success": true
}
```

### POST /preview_{camera_num}
Start camera preview.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Success Response:**
```json
{
  "success": true
}
```

---

## Camera Settings

### POST /update_setting
Update a camera control setting.

**Request Body:**
```json
{
  "camera_num": 0,
  "id": "ExposureTime",
  "value": 20000
}
```

**Success Response:**
```json
{
  "success": true,
  "message": "Received setting update for Camera 0: ExposureTime -> 20000"
}
```

### POST /set_sensor_mode
Change the camera sensor mode.

**Request Body:**
```json
{
  "camera_num": 0,
  "sensor_mode": 1
}
```

**Success Response:**
```json
{
  "status": "done",
  "new_mode": 1
}
```

### GET /camera_controls
Get all available camera controls.

**Response:**
```json
{
  "controls": {
    "ExposureTime": {
      "name": "Exposure Time",
      "min": 1,
      "max": 100000,
      "default": 10000
    }
  }
}
```

---

## Camera Profiles

### GET /get_camera_profile?camera_num={num}
Get current camera profile settings.

**Parameters:**
- `camera_num` (query): Camera number (integer)

**Response:**
```json
{
  "success": true,
  "camera_profile": {
    "sensor_mode": 0,
    "controls": {
      "ExposureTime": 10000,
      "AnalogueGain": 1.0
    }
  }
}
```

### POST /save_profile_{camera_num}
Save current camera settings as a profile.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Request Body:**
```json
{
  "filename": "my_profile"
}
```

**Success Response:**
```json
{
  "message": "Profile 'my_profile' saved successfully"
}
```

### POST /reset_profile_{camera_num}
Reset camera to default settings.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Success Response:**
```json
{
  "success": true,
  "message": "Profile reset to default"
}
```

### POST /load_profile
Load a saved camera profile.

**Request Body:**
```json
{
  "profile_name": "my_profile.json",
  "camera_num": 0
}
```

**Success Response:**
```json
{
  "success": true
}
```

### GET /get_profiles
Get list of all saved camera profiles.

**Response:**
```json
[
  {
    "filename": "profile1.json",
    "model": "imx219"
  }
]
```

---

## Metadata

### GET /fetch_metadata_{camera_num}
Get camera metadata.

**Parameters:**
- `camera_num` (path): Camera number (integer)

**Response:** Camera metadata object

---

## GPIO Control

### GET /gpio_setup
Get GPIO pin configuration.

**Response:**
```json
{
  "gpio_pins": [
    {
      "pin": 17,
      "name": "GPIO17",
      "mode": "output"
    }
  ]
}
```

---

## Image Gallery

### GET /image_gallery?page={page}
Get paginated list of captured images.

**Parameters:**
- `page` (query, optional): Page number (default: 1)

**Response:**
```json
{
  "image_files": [
    {
      "filename": "pimage_camera_0_1640995200.jpg",
      "timestamp": "2022-01-01 12:00:00",
      "has_dng": false,
      "dng_file": "pimage_camera_0_1640995200.dng",
      "width": 3280,
      "height": 2464
    }
  ],
  "page": 1,
  "total_pages": 5,
  "start_page": 1,
  "end_page": 5
}
```

### GET /get_image_for_page?page={page}
Get images for a specific page (alternative to /image_gallery).

**Parameters:**
- `page` (query, optional): Page number (default: 1)

**Response:** Same as `/image_gallery`

### GET /view_image/{filename}
View/download an image file.

**Parameters:**
- `filename` (path): Image filename

**Response:** Image file (binary)

### DELETE /delete_image/{filename}
Delete an image file.

**Parameters:**
- `filename` (path): Image filename

**Success Response:**
```json
{
  "success": true,
  "message": "Image 'filename.jpg' deleted successfully."
}
```

### GET /image_edit/{filename}
Get image edit information.

**Parameters:**
- `filename` (path): Image filename

**Response:**
```json
{
  "filename": "image.jpg",
  "path": "/path/to/image.jpg"
}
```

### POST /apply_filters
Apply image filters (brightness, contrast, rotation).

**Request Body:**
```json
{
  "filename": "image.jpg",
  "brightness": 1.2,
  "contrast": 0.8,
  "rotation": 90
}
```

**Success Response:**
```json
{
  "success": true,
  "edited_filename": "edited_image.jpg",
  "download_url": "/download_image/edited_image.jpg"
}
```

### GET /download_image/{filename}
Download an edited image.

**Parameters:**
- `filename` (path): Edited image filename

**Response:** Image file (binary)

### POST /save_edit
Save image edits.

**Request Body:**
```json
{
  "filename": "image.jpg",
  "edits": {
    "brightness": 1.2,
    "rotation": 90
  },
  "saveOption": "replace",
  "newFilename": "edited_image.jpg"
}
```

---

## System Control

### GET /about
Get API information.

**Response:**
```json
{
  "title": "CamAPI - for picamera2",
  "version": "2.0.0 - BETA",
  "description": "CamAPI headless camera API for picamera2"
}
```

### GET /system_settings
Get system settings.

**Response:**
```json
{
  "firmware_control": false,
  "camera_modules": [
    {
      "sensor_model": "imx219",
      "sensor_name": "Sony IMX219"
    }
  ]
}
```

### POST /set_camera_config
Configure camera overlay in boot config.

**Request Body:**
```json
{
  "sensor_model": "imx219"
}
```

**Success Response:**
```json
{
  "message": "Camera 'imx219' set in boot config!"
}
```

### POST /reset_camera_detection
Reset camera detection to automatic.

**Success Response:**
```json
{
  "message": "Camera detection reset to automatic."
}
```

### POST /shutdown
Shutdown the system.

**Success Response:**
```json
{
  "message": "System is shutting down."
}
```

### POST /restart
Restart the system.

**Success Response:**
```json
{
  "message": "System is restarting."
}
```

---

## Miscellaneous

### GET /docs
Redirect to API documentation.

**Response:** Redirect to `/apidocs`

### GET /set_theme/{theme}
Set theme (available in headless mode).

**Parameters:**
- `theme` (path): Theme name

**Response:**
```json
{
  "success": true,
  "ok": true,
  "theme": "theme_name",
  "message": "Theme endpoint is available in headless mode"
}
```

### GET /beta
Beta endpoint status.

**Response:**
```json
{
  "status": "beta",
  "message": "CamAPI headless endpoint available"
}
```

---

## Error Codes

- `200`: Success
- `400`: Bad Request (invalid parameters)
- `404`: Resource not found (camera, image)
- `500`: Internal server error

## Content Types

- JSON responses: `application/json`
- Image responses: `image/jpeg`
- Video streams: `multipart/x-mixed-replace; boundary=frame`

## Rate Limiting

- Image capture is rate limited to prevent overlapping captures
- Default limit: 1 capture per 2 seconds per camera

## File Storage

- Images stored in: `static/gallery/`
- Camera profiles stored in: `static/camera_profiles/`
- Configuration files: `camera-last-config.json`, `camera-module-info.json`

---

## Usage Examples

### Python (requests library)

```python
import requests

# Get API status
response = requests.get('http://localhost:8080/')
print(response.json())

# Capture image
response = requests.post('http://localhost:8080/capture_still_0')
print(response.json())

# Get camera settings
response = requests.get('http://localhost:8080/camera_0')
print(response.json())
```

### JavaScript (fetch API)

```javascript
// Get API status
fetch('http://localhost:8080/')
  .then(response => response.json())
  .then(data => console.log(data));

// Capture image
fetch('http://localhost:8080/capture_still_0', {
  method: 'POST'
})
  .then(response => response.json())
  .then(data => console.log(data));
```

### cURL

```bash
# Get API status
curl http://localhost:8080/

# Capture image
curl -X POST http://localhost:8080/capture_still_0

# Get camera info
curl http://localhost:8080/camera_info_0
```