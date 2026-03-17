# MemoBot - Memory Layer for Robots

Semantic memory storage and retrieval for humanoid robots. MemoBot enables robots to continuously record their environment and intelligently recall information. It uses a client-server architecture to stream audio and video, providing real-time conversational interactions powered by AI models like Gemini Live.

## Architecture

The system is split into two primary components:

<<<<<<< HEAD
If import errors:
uv pip install -e .

uv pip install google-genai==1.63.0
=======
```text
┌─────────────────┐                    ┌─────────────────┐
│  Robot Client   │ ──(Raw TCP)──────▶ │  Mac/Linux      │
│  (Microphone,   │ ◀─(Raw TCP)─────── │  Master Server  │
│   Camera,       │                    │  (Audio/Video   │
│   Speaker)      │                    │   Processing,   │
└─────────────────┘                    │   Gemini Live)  │
                                       └─────────────────┘
```

1. **Server (`mac_master_v10.py`)**: Runs on a robust host machine (Mac/Linux). It handles heavy processing like Hotword detection (Picovoice), Active Speaker Detection (TalkNet), Video Processing, and real-time audio interaction with the Gemini Live API.
2. **Client (`robot_client_v6.py` or `mock_audio_client_v2.py`)**: A lightweight script running directly on the physical robot (e.g., NAO) or locally as a mock client. It records raw camera/mic data, sends it over TCP, and plays back the audio responses.

## Environment Setup

We use [`uv`](https://github.com/astral-sh/uv) for fast and reliable Python environment management.

### 1. Install `uv`

If you don't already have `uv` installed, run:

```bash
# On Mac/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. Install Dependencies

Clone the repository and set up the virtual environment:

```bash
git clone https://github.com/your-org/memobot.git
cd memobot

# Sync dependencies and create a managed virtual environment
uv sync

# Activate the virtual environment
source .venv/bin/activate
```

*(Note: On the physical robot, `uv` may also be used to sync dependencies, or you can rely on the minimal client scripts which only require standard libraries or lightweight dependencies.)*

## Configuration

Copy the environment template and set up your API keys. Create a `.env` file in the root directory:

```bash
# .env
PICOVOICE_ACCESS_KEY=your_picovoice_key # Required for wake word detection (e.g., 'jarvis')
GOOGLE_API_KEY=your_google_api_key      # Required for Gemini Live integration
OPENAI_API_KEY=sk-...                   # For embeddings/text fallback
TWELVE_LABS_API_KEY=tlk_...             # For video ingestion (if enabled)
```

## Running the System

Ensure you have activated your virtual environment before running the server or mock client on your Mac/Linux host.

### 1. Start the Server (Mac / Linux)

The master server handles all intelligence and routes audio back to the client.

```bash
# Start the server with real-time Gemini Live interaction
python memobot/robot/mac_master_v10.py --realtime

# Start the server with both real-time interaction AND memory ingestion
python memobot/robot/mac_master_v10.py --realtime --ingest
```

### 2. Start the Client

#### Option A: Running on a Physical Robot

Copy the project files to the robot. The client script streams its camera and microphone directly to the master server.

```bash
# On the robot (e.g. NAO)
python memobot/robot/robot_client_v6.py --mac-ip <YOUR_MAC_SERVER_IP>
```

#### Option B: Running a Mock Client (Mac / Linux)

If you don't have a robot, you can use the mock client, which will use your computer's built-in webcam and microphone.

```bash
# In a new terminal on your Mac/Linux machine
source .venv/bin/activate

# Run the mock client pointing to your Mac Server IP (e.g., localhost)
python memobot/robot/mock_audio_client_v2.py --host 127.0.0.1
```

## Troubleshooting

- **Missing Modules / Import Errors:** Make sure you're running the scripts from the project root directory and that the `uv` virtual environment is activated (`source .venv/bin/activate`).
- **Cannot Connect / Connection Refused:** Ensure that the host server IP is correctly specified when starting the client and that ports `50005`, `50006`, `50007`, `50009`, and `50010` are open in your server firewall.
- **No Audio Output:** The mock client might need specific device IDs. Run `python memobot/robot/mock_audio_client_v2.py --list-output-devices` to find your speaker index, then pass it with `--output-device <index>`.

## Legacy Backend Features

*(Note: Earlier versions relied on a Docker Compose backend. You can still refer to `ARCHITECTURE.md` and `USER_MANUAL.md` for historical design docs, but the `mac_master_v10.py` flow is the current standard for low-latency realtime interaction.)*
>>>>>>> 44c0c90c711afe2b76cc7433cadb1c86dc1f4b00
