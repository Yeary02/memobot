# Robot Integration

This folder contains scripts and modules for integrating with NAO robots, enabling bidirectional audio streaming, video capture, hotword detection, and remote control capabilities.

## Overview

The robot integration system provides multiple approaches for audio/video streaming and control:

- **Bidirectional Audio Streaming**: Real-time audio streaming between NAO robot and Mac
- **Video Capture**: Live video feed from NAO cameras
- **Hotword Detection**: Keyword spotting using PocketSphinx
- **Remote Control**: WebSocket and TCP-based command interfaces
- **Audio Playback**: Play audio files and TTS on the robot

## Files

### Audio Streaming

#### `nao_audio_client_v2.py` & `nao_audio_server_v2.py`
**Bidirectional audio streaming (recommended)**

- **Client** (`nao_audio_client_v2.py`): Runs on NAO robot
  - Streams robot microphone audio to Mac
  - Receives audio from Mac and plays on robot speakers
  - Compatible with NAOqi 2.1
  - Uses `ALAudioDevice.sendRemoteBufferToOutput` for playback

- **Server** (`nao_audio_server_v2.py`): Runs on Mac
  - Receives audio from NAO and plays on Mac speakers
  - Captures Mac microphone and sends to robot
  - Uses PyAudio for audio I/O

**Usage:**
```bash
# On Mac (start server first)
python robot/nao_audio_server_v2.py

# On NAO robot
python robot/nao_audio_client_v2.py <ROBOT_IP>
```

**Configuration:**
- Default port: `50005`
- Sample rate: `16000 Hz`
- Format: `16-bit PCM, mono`
- Update `SERVER_IP` in client script to match your Mac's IP address

#### `nao_audio_client.py` & `nao_audio_server.py`
**One-way audio streaming (legacy)**

- **Client** (`nao_audio_client.py`): Streams robot microphone to Mac
- **Server** (`nao_audio_server.py`): Receives and plays audio on Mac

**Usage:**
```bash
# On Mac
python robot/nao_audio_server.py

# On NAO robot
python robot/nao_audio_client.py <ROBOT_IP>
```

### Main Streaming Application

#### `memobot_streamer.py`
**Comprehensive streaming and control application**

Full-featured application that combines:
- Audio capture from NAO with hotword detection
- Video capture from NAO cameras (top/bottom)
- Audio playback to NAO (WAV files, TTS)
- WebSocket server for remote commands
- Real-time waveform visualization

**Features:**
- Hotword detection using PocketSphinx (default: "memobot")
- Prevents hotword triggers during audio playback
- ALMemory event generation on hotword detection
- WebSocket command interface
- OpenCV-based video and audio visualization

**Usage:**
```bash
python robot/memobot_streamer.py \
    --robot-ip <ROBOT_IP> \
    --robot-port 9559 \
    --camera-id 0 \
    --fps 15 \
    --hotword memobot \
    --kws-threshold 1e-25 \
    --ws-host 0.0.0.0 \
    --ws-port 8765
```

**WebSocket Commands:**
```json
// Play WAV from base64
{"type": "play_wav_b64", "wav_b64": "<base64_encoded_wav>"}

// Play WAV from URL
{"type": "play_wav_url", "url": "http://example.com/audio.wav"}

// Text-to-speech
{"type": "say", "text": "Hello, I am MemoBot"}
```

**Controls:**
- Press `q` to quit
- Video window shows live feed with "PLAYBACK" indicator
- Audio window shows real-time waveform

#### `laptop_audio_pull.py`
**Alternative audio streaming with TCP-based commands**

Similar to `memobot_streamer.py` but uses:
- TCP socket for audio streaming (instead of NAOqi callbacks)
- TCP JSON-lines protocol for commands (instead of WebSocket)
- Pull-based audio retrieval from robot

**Usage:**
```bash
python robot/laptop_audio_pull.py \
    --robot-ip <ROBOT_IP> \
    --robot-port 9559 \
    --audio-port 20000 \
    --camera-id 0 \
    --fps 15 \
    --hotword memobot \
    --kws-threshold 1e-25 \
    --cmd-host 0.0.0.0 \
    --cmd-port 8765
```

**TCP Command Protocol:**
Send JSON objects, one per line:
```json
{"type": "play_wav_b64", "wav_b64": "<base64>"}
{"type": "say", "text": "Hello"}
```

### Testing/Debugging

#### `working_audio_pull.py`
**Simple audio test script**

Basic script to verify NAO audio streaming. Prints audio buffer information for debugging.

**Usage:**
```bash
python robot/working_audio_pull.py <ROBOT_IP>
```

## Dependencies

### Required Python Packages
- `naoqi` - NAOqi Python SDK (installed on robot or via Choregraphe)
- `pyaudio` - Audio I/O (for Mac/server side)
- `numpy` - Numerical operations
- `opencv-python` - Video/audio visualization
- `pocketsphinx` - Hotword detection (optional)
- `websocket-server` - WebSocket support (optional, for `memobot_streamer.py`)

### NAOqi Setup
1. Install NAOqi Python SDK on your development machine
2. Ensure robot is on the same network
3. Default NAOqi port: `9559`
4. For audio streaming, robot must have NAOqi 2.1+ (for `_v2` scripts)

### Mac Setup
```bash
# Install PyAudio (may require Homebrew)
brew install portaudio
pip install pyaudio

# Install other dependencies
pip install numpy opencv-python

# Optional: Hotword detection
pip install pocketsphinx

# Optional: WebSocket server
pip install websocket-server
```

## Audio Configuration

### Sample Rates and Formats
- **Sample Rate**: 16000 Hz (standard for NAO)
- **Format**: 16-bit PCM, little-endian
- **Channels**: 
  - Microphone: Mono (channel flag 3 = front mic)
  - Playback: Stereo (converted from mono if needed)

### Channel Flags (ALAudioDevice)
- `0` = ALL channels
- `1` = LEFT channel
- `2` = RIGHT channel
- `3` = FRONT microphone (recommended)
- `4` = REAR microphone

## Network Configuration

### Ports
- `50005` - Bidirectional audio streaming (default)
- `8765` - Command server (WebSocket or TCP)
- `9559` - NAOqi broker (robot default)
- `20000` - Audio pull server (if using `laptop_audio_pull.py`)

### IP Address Configuration
Update the `SERVER_IP` constant in client scripts to match your Mac's IP address:
```python
SERVER_IP = "10.19.171.37"  # Replace with your Mac's IP
```

## Hotword Detection

### Configuration
- **Default hotword**: "memobot"
- **Threshold**: `1e-25` (lower = more sensitive)
- **Debounce**: 1 second between detections

### Customization
Modify the ARPAbet pronunciation in the `HotwordKWS` class if needed:
```python
# Current: "M EH M OW B AA T"
dict_text = "%s M EH M OW B AA T\n" % hotword.upper()
```

### Disabling Hotword Detection
If PocketSphinx is not installed, hotword detection is automatically disabled and the application will continue to function normally.

## Troubleshooting

### Connection Issues
- Verify robot and Mac are on the same network
- Check firewall settings (port 50005, 8765)
- Ensure NAOqi is running on the robot (port 9559)

### Audio Issues
- Verify sample rate matches (16000 Hz)
- Check audio device permissions on Mac
- Ensure robot microphone is not muted
- For playback issues, verify audio format (16-bit PCM, 16kHz)

### NAOqi 2.1 Compatibility
The `_v2` scripts use `sendRemoteBufferToOutput` which is compatible with NAOqi 2.1+. For older versions, use the non-`_v2` scripts.

### Import Errors
- Ensure NAOqi SDK is in Python path
- For robot-side scripts, run on robot or use Choregraphe
- Install missing dependencies: `pip install <package>`

## Architecture Notes

### Audio Flow
1. **Robot → Mac**: NAOqi `ALAudioDevice` callbacks → TCP socket → PyAudio output
2. **Mac → Robot**: PyAudio input → TCP socket → `ALAudioDevice.sendRemoteBufferToOutput`

### Threading Model
- Audio capture: NAOqi callback thread
- Audio playback: Separate thread with timing control
- Command processing: Worker thread with queue
- Network I/O: Dedicated threads per connection

### Memory Management
- Audio buffers use NumPy arrays for efficiency
- Ring buffers for rolling audio history
- Automatic cleanup on shutdown

## Examples

### Basic Audio Streaming
```bash
# Terminal 1: Start server on Mac
python robot/nao_audio_server_v2.py

# Terminal 2: Start client on robot (via SSH or Choregraphe)
python robot/nao_audio_client_v2.py 192.168.1.100
```

### Full-Featured Streaming
```bash
python robot/memobot_streamer.py --robot-ip 192.168.1.100 --camera-id 0
```

### Remote Control via WebSocket
```python
import websocket
import json
import base64

ws = websocket.create_connection("ws://localhost:8765")

# Play audio
with open("audio.wav", "rb") as f:
    wav_b64 = base64.b64encode(f.read()).decode()
    ws.send(json.dumps({"type": "play_wav_b64", "wav_b64": wav_b64}))

# Text-to-speech
ws.send(json.dumps({"type": "say", "text": "Hello from remote control"}))
```

## License

See main project LICENSE file.

COMMAND TO RUN SERVER:
Run with Ingest:
python memobot/robot/mac_master_v7.py --realtime --ingest

Without Ingest:
uv pip install google-genai==1.63.0

Then:
python memobot/robot/mac_master_v10.py --realtime
or: python -m memobot.robot.mac_master_v8 --realtime
