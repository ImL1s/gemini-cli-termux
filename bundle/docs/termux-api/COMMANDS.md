# Termux-API Commands Reference

**Completeness**: 100% of Termux-API commands documented **Date**: 2025-12-17

---

## Overview

This guide documents all available Termux-API commands, with parameters,
expected output, and integration priority.

**Prerequisite**: `pkg install termux-api` + Termux:API App installed from
F-Droid

---

## Commands by Category

### 1. System Information

#### termux-battery-status

**Description**: Device battery status **Parameters**: None **Output**: JSON

```json
{
  "health": "GOOD",
  "percentage": 85,
  "plugged": "UNPLUGGED",
  "status": "DISCHARGING",
  "temperature": 28.5,
  "current": -450000
}
```

**Priority**: HIGH **Notes**: Useful for battery-based automation

---

#### termux-audio-info

**Description**: Audio device information **Parameters**: None **Output**: JSON
with speaker/bluetooth/headset status **Priority**: MEDIUM

---

#### termux-wifi-connectioninfo

**Description**: Current WiFi connection info **Parameters**: None **Output**:
JSON with SSID, BSSID, IP, link_speed, rssi **Priority**: HIGH

---

#### termux-wifi-scaninfo

**Description**: Scan available WiFi networks **Parameters**: None **Output**:
JSON array of networks **Priority**: LOW

---

#### termux-telephony-deviceinfo

**Description**: Telephony device info **Parameters**: None **Output**: JSON
with IMEI, network_operator, sim_state, etc. **Priority**: MEDIUM **Privacy**:
Contains sensitive data

---

#### termux-telephony-cellinfo

**Description**: Cellular cell info **Parameters**: None **Output**: JSON with
cell tower info **Priority**: LOW

---

#### termux-info

**Description**: Termux system info **Parameters**: None **Output**: Text with
versions and paths **Priority**: HIGH (debug)

---

### 2. Notifications

#### termux-toast

**Description**: Show toast message **Parameters**:

- `-s` short duration
- `-g gravity` (top, middle, bottom)
- `-c color` background
- `-C color` text color **Input**: Text from stdin or argument **Output**: None
  **Priority**: HIGH **Example**: `echo "Hello" | termux-toast -g top`

---

#### termux-notification

**Description**: Create persistent notification **Parameters**:

- `-t title`
- `-c content`
- `--icon icon_name`
- `--id notification_id`
- `--priority high|low|default`
- `--action action_name`
- `--on-delete command`
- `--button1 label:command`
- `--button2 label:command`
- `--button3 label:command`
- `--led-color RRGGBB`
- `--led-on ms`
- `--led-off ms`
- `--sound`
- `--vibrate pattern`
- `--type default|media`
- `--group group_name`
- `--alert-once`
- `--ongoing`
- `--image-path path` **Output**: None **Priority**: HIGH **Example**:
  `termux-notification -t "Title" -c "Content" --id mynotif`

---

#### termux-notification-remove

**Description**: Remove notification **Parameters**: `--id notification_id`
**Priority**: MEDIUM

---

#### termux-notification-list

**Description**: List active notifications **Output**: JSON array **Priority**:
LOW

---

### 3. Clipboard

#### termux-clipboard-get

**Description**: Read clipboard **Parameters**: None **Output**: Clipboard
content to stdout **Priority**: HIGH

---

#### termux-clipboard-set

**Description**: Write to clipboard **Input**: Text from stdin **Output**: None
**Priority**: HIGH **Example**: `echo "text" | termux-clipboard-set`

---

### 4. Media

#### termux-camera-info

**Description**: Available cameras info **Output**: JSON with id, facing,
resolutions **Priority**: MEDIUM

---

#### termux-camera-photo

**Description**: Take photo **Parameters**:

- `-c camera_id` (0=back, 1=front)
- `output_file` **Output**: JPEG File **Priority**: MEDIUM **Example**:
  `termux-camera-photo -c 0 photo.jpg`

---

#### termux-microphone-record

**Description**: Record audio **Parameters**:

- `-f output_file`
- `-l limit_seconds`
- `-e encoder` (aac, amr_nb, amr_wb)
- `-b bitrate`
- `-r sample_rate`
- `-c channels` (1, 2)
- `-d` (stop recording)
- `-i` (info current recording) **Priority**: MEDIUM

---

#### termux-media-player

**Description**: Control media player **Parameters**:

- `play file`
- `pause`
- `stop`
- `info` **Priority**: LOW

---

#### termux-media-scan

**Description**: Scan media files **Parameters**: `-r` recursive, file/directory
path **Priority**: LOW

---

#### termux-tts-speak

**Description**: Text-to-Speech **Parameters**:

- `-e engine`
- `-l language`
- `-n region`
- `-v variant`
- `-p pitch` (default 1.0)
- `-r rate` (default 1.0)
- `-s stream` (NOTIFICATION, MUSIC, etc.) **Input**: Text from stdin
  **Priority**: HIGH **Example**: `echo "Hello" | termux-tts-speak -l en`

---

#### termux-tts-engines

**Description**: List available TTS engines **Output**: JSON array **Priority**:
LOW

---

#### termux-speech-to-text

**Description**: Speech recognition **Parameters**: None (starts listening)
**Output**: Recognized text **Priority**: MEDIUM

---

### 5. Location

#### termux-location

**Description**: Get GPS location **Parameters**:

- `-p provider` (gps, network, passive)
- `-r request` (once, last, updates) **Output**: JSON with latitude, longitude,
  altitude, accuracy, etc. **Priority**: HIGH **Example**:
  `termux-location -p gps -r once`

---

### 6. Sensors & Hardware

#### termux-sensor

**Description**: Read sensors **Parameters**:

- `-l` list sensors
- `-s sensor_name`
- `-d delay_ms`
- `-n count`
- `-c` cleanup **Output**: JSON with sensor values **Priority**: MEDIUM

---

#### termux-torch

**Description**: Control flashlight **Parameters**: `on` | `off` **Priority**:
MEDIUM

---

#### termux-vibrate

**Description**: Vibrate **Parameters**:

- `-d duration_ms`
- `-f` force (even in silent mode) **Priority**: MEDIUM **Example**:
  `termux-vibrate -d 500`

---

#### termux-brightness

**Description**: Set brightness **Parameters**: `0-255` or `auto` **Priority**:
LOW

---

#### termux-infrared-frequencies

**Description**: Supported IR frequencies **Output**: JSON array of frequency
ranges **Priority**: LOW

---

#### termux-infrared-transmit

**Description**: Transmit IR **Parameters**: `-f frequency` pattern
**Priority**: LOW

---

#### termux-fingerprint

**Description**: Fingerprint authentication **Output**: JSON with auth_result
**Priority**: HIGH (security) **Notes**: Requires biometric hardware

---

### 7. Communication

#### termux-sms-send

**Description**: Send SMS **Parameters**:

- `-n number` (recipient)
- `-s slot` (SIM slot) **Input**: Message from stdin **Priority**: HIGH (with
  caution) **Privacy**: CRITICAL

---

#### termux-sms-inbox

**Description**: Read SMS inbox (deprecated) **Use**: termux-sms-list

---

#### termux-sms-list

**Description**: List SMS **Parameters**:

- `-l limit`
- `-o offset`
- `-t type` (inbox, sent, draft, all)
- `-n` (show phone numbers)
- `-d` (show dates) **Output**: JSON array **Privacy**: CRITICAL

---

#### termux-telephony-call

**Description**: Make call **Parameters**: `phone_number` **Privacy**: CRITICAL

---

#### termux-call-log

**Description**: Call log **Parameters**:

- `-l limit`
- `-o offset` **Output**: JSON array **Privacy**: CRITICAL

---

#### termux-contact-list

**Description**: Contact list **Output**: JSON array with name, number
**Privacy**: CRITICAL

---

### 8. Storage & Files

#### termux-download

**Description**: Download file using system downloader **Parameters**:

- `-d description`
- `-t title`
- `url` **Priority**: MEDIUM

---

#### termux-share

**Description**: Share file/text **Parameters**:

- `-a action` (edit, send, view)
- `-c content-type`
- `-d` (default activity)
- `-t title` **Input**: File or stdin **Priority**: MEDIUM

---

#### termux-open

**Description**: Open file with associated app **Parameters**:

- `--send` (send action)
- `--view` (view action)
- `--chooser` (show chooser)
- `--content-type type`
- `file_path` **Priority**: MEDIUM

---

#### termux-open-url

**Description**: Open URL in browser **Parameters**: `url` **Priority**: MEDIUM

---

#### termux-storage-get

**Description**: Request file from Android storage **Parameters**: `output_file`
**Priority**: LOW

---

#### termux-saf-\* (Storage Access Framework)

**Commands**:

- `termux-saf-create` - Create file
- `termux-saf-dirs` - List SAF directories
- `termux-saf-ls` - List content
- `termux-saf-managedir` - Manage dir access
- `termux-saf-mkdir` - Create directory
- `termux-saf-read` - Read file
- `termux-saf-rm` - Remove file
- `termux-saf-stat` - File info
- `termux-saf-write` - Write file **Priority**: LOW **Notes**: For external
  storage access on Android 11+

---

### 9. System Control

#### termux-volume

**Description**: Control volume **Parameters**:

- Without parameters: show volumes
- `stream volume` (set volume) **Streams**: alarm, music, notification, ring,
  system, call **Priority**: MEDIUM

---

#### termux-wake-lock

**Description**: Acquire wake lock (prevent sleep) **Priority**: MEDIUM

---

#### termux-wake-unlock

**Description**: Release wake lock **Priority**: MEDIUM

---

#### termux-wallpaper

**Description**: Set wallpaper **Parameters**:

- `-f file`
- `-u url`
- `-l` (lockscreen) **Priority**: LOW

---

#### termux-wifi-enable

**Description**: Enable/disable WiFi **Parameters**: `true` | `false`
**Priority**: LOW **Notes**: May require special permissions

---

### 10. Dialogs

#### termux-dialog

**Description**: Show interactive dialog **Parameters**:

- `-t title`
- `-l` list widget
- `-i` text input
- `-m` multi-select
- `-p` password input
- `-r` radio buttons
- `-s` spinner
- `-d` date picker
- `-T` time picker
- `-c` confirm dialog
- `-C` counter
- `--values v1,v2,v3`
- `--range min,max` **Output**: JSON with result **Priority**: MEDIUM

---

### 11. Other

#### termux-nfc

**Description**: NFC operations **Parameters**: Various for reading/writing tags
**Priority**: LOW

---

#### termux-usb

**Description**: USB device info **Parameters**:

- `-l` list devices
- `-r` request permissions
- `-e command` execute with USB permissions **Priority**: LOW

---

#### termux-job-scheduler

**Description**: Schedule periodic jobs **Parameters**:

- `--pending` list jobs
- `--cancel-all` cancel all
- `--cancel id` cancel specific
- `--job-id id`
- `--script path`
- `--period-ms ms`
- `--network type`
- `--battery-not-low`
- `--storage-not-low`
- `--charging`
- `--idle`
- `--persisted` **Priority**: MEDIUM

---

#### termux-keystore

**Description**: Cryptographic key management **Parameters**:

- `list` list keys
- `delete alias` delete key
- `generate alias [-a algorithm] [-s size]` generate key
- `sign alias` sign data
- `verify alias` verify signature **Priority**: LOW (advanced)

---

## Integration Priority

### Phase 1 - Core (HIGH)

1. termux-battery-status
2. termux-clipboard-get
3. termux-clipboard-set
4. termux-toast
5. termux-notification
6. termux-tts-speak
7. termux-wifi-connectioninfo
8. termux-info

### Phase 2 - Extended (MEDIUM)

1. termux-location
2. termux-camera-photo
3. termux-vibrate
4. termux-torch
5. termux-dialog
6. termux-volume
7. termux-audio-info
8. termux-sensor

### Phase 3 - Advanced (LOW)

1. termux-sms-send (with caution)
2. termux-microphone-record
3. termux-speech-to-text
4. termux-job-scheduler
5. termux-saf-\*
6. Others

---

## Security Considerations

### Commands with Privacy Implications

- `termux-sms-*` - SMS Access
- `termux-call-log` - Call Logs
- `termux-contact-list` - Contacts
- `termux-telephony-deviceinfo` - IMEI
- `termux-location` - Location

### Recommendations

1. **Mandatory user confirmation** for privacy-sensitive commands
2. **Do not include** sensitive output in LLM context unless necessary
3. **Audit log** for critical commands
4. **Rate limiting** to prevent abuse

---

_Author: DioNanos_
