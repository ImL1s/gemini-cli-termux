# Tool Discovery Setup for Termux-API

**Project**: gemini-cli-termux **Author**: DioNanos **Date**: 2025-12-17

---

## Overview

This guide explains how to set up Tool Discovery to expose Termux-API commands
to Gemini CLI without modifying the core code.

## Prerequisites

1. Termux with termux-api package installed:

   ```bash
   pkg install termux-api
   ```

2. Termux:API App installed from F-Droid

3. Working Gemini CLI:
   ```bash
   gemini --version
   ```

---

## Quick Setup

### 1. Create config directory

```bash
mkdir -p ~/.config/gemini/termux-tools
```

### 2. Create discovery script

```bash
cat > ~/.config/gemini/termux-tools/discovery.sh << 'SCRIPT'
#!/data/data/com.termux/files/usr/bin/bash
# Termux-API Tool Discovery for Gemini CLI
# Returns FunctionDeclarations for Termux commands

cat << 'EOF'
[
  {
    "name": "termux_battery_status",
    "description": "Get device battery status including percentage, health, temperature, and charging state. Returns JSON with: health (GOOD/OVERHEAT/DEAD/etc), percentage (0-100), plugged (AC/USB/WIRELESS/UNPLUGGED), status (CHARGING/DISCHARGING/FULL/NOT_CHARGING), temperature (Celsius), current (microamperes).",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {}
    }
  },
  {
    "name": "termux_clipboard_get",
    "description": "Read the current content of the Android clipboard. Returns the clipboard text content to stdout.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {}
    }
  },
  {
    "name": "termux_clipboard_set",
    "description": "Set the Android clipboard content. The text parameter will be copied to the system clipboard.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "text": {
          "type": "string",
          "description": "Text to copy to clipboard"
        }
      },
      "required": ["text"]
    }
  },
  {
    "name": "termux_toast",
    "description": "Show a toast notification message on screen. Optional parameters control appearance and duration.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "message": {
          "type": "string",
          "description": "Message to display in toast"
        },
        "short": {
          "type": "boolean",
          "description": "Use short duration (default: false for long)"
        },
        "gravity": {
          "type": "string",
          "enum": ["top", "middle", "bottom"],
          "description": "Toast position on screen"
        },
        "background_color": {
          "type": "string",
          "description": "Background color (e.g., 'red', '#FF0000')"
        },
        "text_color": {
          "type": "string",
          "description": "Text color (e.g., 'white', '#FFFFFF')"
        }
      },
      "required": ["message"]
    }
  },
  {
    "name": "termux_notification",
    "description": "Create a persistent notification in the Android notification shade. Supports title, content, icons, actions, buttons, LED, sound, and vibration.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "title": {
          "type": "string",
          "description": "Notification title"
        },
        "content": {
          "type": "string",
          "description": "Notification body text"
        },
        "id": {
          "type": "string",
          "description": "Unique ID to update/remove notification later"
        },
        "priority": {
          "type": "string",
          "enum": ["high", "low", "default"],
          "description": "Notification priority"
        },
        "sound": {
          "type": "boolean",
          "description": "Play notification sound"
        },
        "vibrate": {
          "type": "string",
          "description": "Vibration pattern (e.g., '500,500,500')"
        },
        "ongoing": {
          "type": "boolean",
          "description": "Make notification persistent (cannot be swiped away)"
        }
      },
      "required": ["title", "content"]
    }
  },
  {
    "name": "termux_notification_remove",
    "description": "Remove a notification by its ID.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "description": "Notification ID to remove"
        }
      },
      "required": ["id"]
    }
  },
  {
    "name": "termux_tts_speak",
    "description": "Speak text using Android Text-to-Speech engine. Supports language, pitch, and rate settings.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "text": {
          "type": "string",
          "description": "Text to speak"
        },
        "language": {
          "type": "string",
          "description": "Language code (e.g., 'en', 'it', 'de')"
        },
        "pitch": {
          "type": "number",
          "description": "Pitch multiplier (default: 1.0)"
        },
        "rate": {
          "type": "number",
          "description": "Speech rate multiplier (default: 1.0)"
        }
      },
      "required": ["text"]
    }
  },
  {
    "name": "termux_vibrate",
    "description": "Vibrate the device for specified duration.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "duration": {
          "type": "integer",
          "description": "Vibration duration in milliseconds (default: 1000)"
        },
        "force": {
          "type": "boolean",
          "description": "Vibrate even if device is in silent mode"
        }
      }
    }
  },
  {
    "name": "termux_torch",
    "description": "Control the device flashlight/torch.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "state": {
          "type": "string",
          "enum": ["on", "off"],
          "description": "Turn torch on or off"
        }
      },
      "required": ["state"]
    }
  },
  {
    "name": "termux_wifi_connectioninfo",
    "description": "Get current WiFi connection information. Returns JSON with SSID, BSSID, IP address, link speed, RSSI signal strength, and more.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {}
    }
  },
  {
    "name": "termux_location",
    "description": "Get device GPS location. Returns JSON with latitude, longitude, altitude, accuracy, speed, bearing, and provider information.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "provider": {
          "type": "string",
          "enum": ["gps", "network", "passive"],
          "description": "Location provider (gps=high accuracy, network=fast, passive=lowest power)"
        },
        "request": {
          "type": "string",
          "enum": ["once", "last", "updates"],
          "description": "Request type: once (single fix), last (cached), updates (continuous)"
        }
      }
    }
  },
  {
    "name": "termux_audio_info",
    "description": "Get audio system information including speaker, bluetooth, and headset states.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {}
    }
  },
  {
    "name": "termux_volume",
    "description": "Get or set system volume levels. Without parameters returns all volume levels. With parameters sets specific stream volume.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "stream": {
          "type": "string",
          "enum": ["alarm", "music", "notification", "ring", "system", "call"],
          "description": "Audio stream to control"
        },
        "volume": {
          "type": "integer",
          "description": "Volume level to set (0-15 typically)"
        }
      }
    }
  },
  {
    "name": "termux_brightness",
    "description": "Set screen brightness level.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "brightness": {
          "type": "string",
          "description": "Brightness value 0-255 or 'auto'"
        }
      },
      "required": ["brightness"]
    }
  },
  {
    "name": "termux_camera_info",
    "description": "Get information about available cameras. Returns JSON array with camera IDs, facing direction, and supported resolutions.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {}
    }
  },
  {
    "name": "termux_camera_photo",
    "description": "Take a photo with the device camera.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "camera_id": {
          "type": "integer",
          "description": "Camera ID (0=back, 1=front typically)"
        },
        "output_file": {
          "type": "string",
          "description": "Output file path for the photo"
        }
      },
      "required": ["output_file"]
    }
  },
  {
    "name": "termux_dialog",
    "description": "Show an interactive dialog to the user. Supports various input types.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "title": {
          "type": "string",
          "description": "Dialog title"
        },
        "type": {
          "type": "string",
          "enum": ["confirm", "text", "spinner", "date", "time", "counter"],
          "description": "Dialog type"
        },
        "values": {
          "type": "string",
          "description": "Comma-separated values for spinner/radio"
        },
        "hint": {
          "type": "string",
          "description": "Input hint for text dialogs"
        }
      }
    }
  },
  {
    "name": "termux_share",
    "description": "Share content using Android share intent.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "text": {
          "type": "string",
          "description": "Text to share"
        },
        "file": {
          "type": "string",
          "description": "File path to share"
        },
        "title": {
          "type": "string",
          "description": "Share dialog title"
        },
        "action": {
          "type": "string",
          "enum": ["send", "view", "edit"],
          "description": "Share action type"
        }
      }
    }
  },
  {
    "name": "termux_open_url",
    "description": "Open a URL in the default browser.",
    "parametersJsonSchema": {
      "type": "object",
      "properties": {
        "url": {
          "type": "string",
          "description": "URL to open"
        }
      },
      "required": ["url"]
    }
  }
]
EOF
SCRIPT
chmod +x ~/.config/gemini/termux-tools/discovery.sh
```

### 3. Create call script

```bash
cat > ~/.config/gemini/termux-tools/call.sh << 'SCRIPT'
#!/data/data/com.termux/files/usr/bin/bash
# Termux-API Tool Call dispatcher for Gemini CLI
# Usage: call.sh <tool_name> < params.json

TOOL_NAME="$1"

# Read JSON params from stdin
PARAMS=$(cat)

case "$TOOL_NAME" in
  termux_battery_status)
    termux-battery-status
    ;;

  termux_clipboard_get)
    termux-clipboard-get
    ;;

  termux_clipboard_set)
    TEXT=$(echo "$PARAMS" | jq -r '.text // empty')
    if [ -n "$TEXT" ]; then
      echo "$TEXT" | termux-clipboard-set
      echo '{"status": "ok"}'
    else
      echo '{"error": "text parameter required"}'
      exit 1
    fi
    ;;

  termux_toast)
    MSG=$(echo "$PARAMS" | jq -r '.message // empty')
    SHORT=$(echo "$PARAMS" | jq -r '.short // false')
    GRAVITY=$(echo "$PARAMS" | jq -r '.gravity // empty')
    BG=$(echo "$PARAMS" | jq -r '.background_color // empty')
    TC=$(echo "$PARAMS" | jq -r '.text_color // empty')

    ARGS=""
    [ "$SHORT" = "true" ] && ARGS="$ARGS -s"
    [ -n "$GRAVITY" ] && ARGS="$ARGS -g $GRAVITY"
    [ -n "$BG" ] && ARGS="$ARGS -c $BG"
    [ -n "$TC" ] && ARGS="$ARGS -C $TC"

    echo "$MSG" | termux-toast $ARGS
    echo '{"status": "ok"}'
    ;;

  termux_notification)
    TITLE=$(echo "$PARAMS" | jq -r '.title // empty')
    CONTENT=$(echo "$PARAMS" | jq -r '.content // empty')
    ID=$(echo "$PARAMS" | jq -r '.id // empty')
    PRIORITY=$(echo "$PARAMS" | jq -r '.priority // empty')
    SOUND=$(echo "$PARAMS" | jq -r '.sound // false')
    VIBRATE=$(echo "$PARAMS" | jq -r '.vibrate // empty')
    ONGOING=$(echo "$PARAMS" | jq -r '.ongoing // false')

    ARGS="-t \"$TITLE\" -c \"$CONTENT\""
    [ -n "$ID" ] && ARGS="$ARGS --id $ID"
    [ -n "$PRIORITY" ] && ARGS="$ARGS --priority $PRIORITY"
    [ "$SOUND" = "true" ] && ARGS="$ARGS --sound"
    [ -n "$VIBRATE" ] && ARGS="$ARGS --vibrate $VIBRATE"
    [ "$ONGOING" = "true" ] && ARGS="$ARGS --ongoing"

    eval "termux-notification $ARGS"
    echo '{"status": "ok"}'
    ;;

  termux_notification_remove)
    ID=$(echo "$PARAMS" | jq -r '.id // empty')
    termux-notification-remove --id "$ID"
    echo '{"status": "ok"}'
    ;;

  termux_tts_speak)
    TEXT=$(echo "$PARAMS" | jq -r '.text // empty')
    LANG=$(echo "$PARAMS" | jq -r '.language // empty')
    PITCH=$(echo "$PARAMS" | jq -r '.pitch // empty')
    RATE=$(echo "$PARAMS" | jq -r '.rate // empty')

    ARGS=""
    [ -n "$LANG" ] && ARGS="$ARGS -l $LANG"
    [ -n "$PITCH" ] && ARGS="$ARGS -p $PITCH"
    [ -n "$RATE" ] && ARGS="$ARGS -r $RATE"

    echo "$TEXT" | termux-tts-speak $ARGS
    echo '{"status": "ok"}'
    ;;

  termux_vibrate)
    DUR=$(echo "$PARAMS" | jq -r '.duration // 1000')
    FORCE=$(echo "$PARAMS" | jq -r '.force // false')

    ARGS="-d $DUR"
    [ "$FORCE" = "true" ] && ARGS="$ARGS -f"

    termux-vibrate $ARGS
    echo '{"status": "ok"}'
    ;;

  termux_torch)
    STATE=$(echo "$PARAMS" | jq -r '.state // empty')
    termux-torch "$STATE"
    echo '{"status": "ok"}'
    ;;

  termux_wifi_connectioninfo)
    termux-wifi-connectioninfo
    ;;

  termux_location)
    PROVIDER=$(echo "$PARAMS" | jq -r '.provider // "gps"')
    REQUEST=$(echo "$PARAMS" | jq -r '.request // "once"')
    termux-location -p "$PROVIDER" -r "$REQUEST"
    ;;

  termux_audio_info)
    termux-audio-info
    ;;

  termux_volume)
    STREAM=$(echo "$PARAMS" | jq -r '.stream // empty')
    VOL=$(echo "$PARAMS" | jq -r '.volume // empty')

    if [ -n "$STREAM" ] && [ -n "$VOL" ]; then
      termux-volume "$STREAM" "$VOL"
      echo '{"status": "ok"}'
    else
      termux-volume
    fi
    ;;

  termux_brightness)
    BRIGHTNESS=$(echo "$PARAMS" | jq -r '.brightness // empty')
    termux-brightness "$BRIGHTNESS"
    echo '{"status": "ok"}'
    ;;

  termux_camera_info)
    termux-camera-info
    ;;

  termux_camera_photo)
    CAM_ID=$(echo "$PARAMS" | jq -r '.camera_id // 0')
    OUTPUT=$(echo "$PARAMS" | jq -r '.output_file // empty')
    termux-camera-photo -c "$CAM_ID" "$OUTPUT"
    echo "{\"status\": \"ok\", \"file\": \"$OUTPUT\"}"
    ;;

  termux_dialog)
    TITLE=$(echo "$PARAMS" | jq -r '.title // empty')
    TYPE=$(echo "$PARAMS" | jq -r '.type // "confirm"')
    VALUES=$(echo "$PARAMS" | jq -r '.values // empty')
    HINT=$(echo "$PARAMS" | jq -r '.hint // empty')

    ARGS=""
    [ -n "$TITLE" ] && ARGS="$ARGS -t \"$TITLE\""

    case "$TYPE" in
      confirm) ARGS="$ARGS confirm" ;;
      text) ARGS="$ARGS -i" ;;
      spinner) ARGS="$ARGS spinner --values \"$VALUES\"" ;;
      date) ARGS="$ARGS date" ;;
      time) ARGS="$ARGS time" ;;
      counter) ARGS="$ARGS counter" ;;
    esac

    [ -n "$HINT" ] && ARGS="$ARGS -i \"$HINT\""

    eval "termux-dialog $ARGS"
    ;;

  termux_share)
    TEXT=$(echo "$PARAMS" | jq -r '.text // empty')
    FILE=$(echo "$PARAMS" | jq -r '.file // empty')
    TITLE=$(echo "$PARAMS" | jq -r '.title // empty')
    ACTION=$(echo "$PARAMS" | jq -r '.action // "send"')

    ARGS="-a $ACTION"
    [ -n "$TITLE" ] && ARGS="$ARGS -t \"$TITLE\""

    if [ -n "$TEXT" ]; then
      echo "$TEXT" | termux-share $ARGS
    elif [ -n "$FILE" ]; then
      termux-share $ARGS "$FILE"
    fi
    echo '{"status": "ok"}'
    ;;

  termux_open_url)
    URL=$(echo "$PARAMS" | jq -r '.url // empty')
    termux-open-url "$URL"
    echo '{"status": "ok"}'
    ;;

  *)
    echo "{\"error\": \"Unknown tool: $TOOL_NAME\"}"
    exit 1
    ;;
esac
SCRIPT
chmod +x ~/.config/gemini/termux-tools/call.sh
```

### 4. Configure Gemini CLI

Edit `~/.config/gemini/settings.json`:

```json
{
  "tool_discovery_command": "bash ~/.config/gemini/termux-tools/discovery.sh",
  "tool_call_command": "bash ~/.config/gemini/termux-tools/call.sh"
}
```

### 5. Test

```bash
# Verify discovery
~/.config/gemini/termux-tools/discovery.sh | jq '.[] | .name'

# Verify call
echo '{}' | ~/.config/gemini/termux-tools/call.sh termux_battery_status

# Test with Gemini
gemini "What's my battery status?"
```

---

## Installation Verification

```bash
# Check Termux-API
pkg list-installed | grep termux-api

# Check jq (required for call.sh)
which jq || pkg install jq

# Check permissions
termux-setup-storage  # Grant storage access
```

---

## Troubleshooting

### Tool not found

```
Error: Unknown tool: termux_xxx
```

**Solution**: Verify that the tool is defined in `discovery.sh` and `call.sh`

### Permission denied

```
Error: Permission denied for termux-xxx
```

**Solution**: Open Termux:API app and grant necessary permissions

### JSON parse error

```
Error: jq: parse error
```

**Solution**: Verify that parameters are valid JSON

---

## Extending

To add new commands:

1. Add FunctionDeclaration in `discovery.sh`
2. Add case handler in `call.sh`
3. Test with direct call
4. Verify with Gemini

---

_Author: DioNanos_
