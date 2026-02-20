---
name: sonos-announce
description: Play audio on Sonos with intelligent state restoration - pauses streaming, skips Line-In/TV/Bluetooth, resumes everything.
metadata:
  {
    "openclaw":
      {
        "emoji": "🔊",
        "requires":
          {
            "bins": ["python3", "ffprobe"],
            "pip": ["soco"],
          },
      },
  }
---

# Sonos Announce

Play audio files on Sonos speakers with intelligent state restoration.

## When to use

- User wants to play an announcement on Sonos
- Soundboard effects (airhorn, effects, etc.)
- Any audio playback that should resume previous state

**This skill handles playback only** - audio generation (TTS, ElevenLabs, etc.) is separate.

## Quick Start

```python
import sys
import os
sys.path.insert(0, os.path.dirname(__file__))
from sonos_core import announce

# Play audio and restore previous state
result = announce('/path/to/audio.mp3')
```

## Installation

If dependencies missing, install them:

```bash
pip install soco
```

The skill expects:
- `python3` - Python 3
- `ffprobe` - Part of ffmpeg, for audio duration detection
- `soco` - Python Sonos library

## Core Function

```python
announce(audio_file_path, wait_for_audio=True)
```

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `audio_file_path` | str | required | Path to audio file (mp3, wav, etc.) |
| `wait_for_audio` | bool | True | Wait for audio to finish playing |

### Returns

```python
{
  'coordinators': 2,
  'states': {
    '192.168.1.120': {
      'uri': 'x-sonos-spotify:spotify%3atrack%3a...',
      'position': '0:01:23',
      'queue_position': 5,
      'was_playing': True,
      'is_external': False,
      'transport_state': 'PLAYING',
      'speaker_name': 'Bedroom'
    }
  }
}
```

## State Restoration

| Source Type | Behavior |
|------------|----------|
| Streaming (Spotify, TuneIn, etc.) | Resumed at exact position |
| Line-In | Re-connected to Line-In input |
| TV/HDMI | Re-connected to TV audio |
| Bluetooth | Re-connected to Bluetooth |
| Paused content | Left paused |

## External Input Detection

Automatically detects inputs that cannot be paused:

- `x-rincon:RINCON_*` - Line-In
- `x-rincon-stream:RINCON_*` - Line-In stream  
- `x-sonos-htastream:*` - TV/HDMI (Sonos Home Theater)
- `x-sonos-vanished:*` - Vanished device
- `x-rincon-bt:*` - Bluetooth

## HTTP Server

The module auto-detects your LAN IP and starts an HTTP server to stream audio to Sonos.

**Environment variables (optional):**
```bash
export SONOS_HTTP_HOST=192.168.1.100  # Override auto-detected IP
export SONOS_HTTP_PORT=8888             # Override port (default: 8888)
```

## Soundboard Example

```python
import sys
import os
sys.path.insert(0, os.path.dirname(__file__))
from sonos_core import announce

SOUNDS = {
    'airhorn': '/path/to/sounds/airhorn.mp3',
    'rimshot': '/path/to/sounds/rimshot.mp3',
    'victory': '/path/to/sounds/victory.mp3',
}

def play_sound(name):
    """Play a sound effect."""
    if name in SOUNDS:
        announce(SOUNDS[name])
    else:
        print(f"Unknown sound: {name}")
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No speakers found | Ensure on same network as Sonos speakers |
| Resume not working | Check speakers were playing (not paused) before announcement |
| HTTP server failed | Check port 8888 is available |
| Module import error | Run: `pip install soco` |

## Files

- `sonos_core.py` - Main module with `announce()` function
- `SKILL.md` - This documentation
