# OmniCam - Face Recognition System

OmniCam is a **real-time face recognition system** built with [DeepFace](https://github.com/serengil/deepface) and OpenCV.  
It recognizes authorized users, detects intruders, and keeps track of their status using a flag system.

## ✨ Features

- 🚀 Fast recognition with **SFace** model (optimized for CPU).
- ✅ Tracks authorized users with `flag=1`.
- 🚨 Detects intruders with `flag=-1` (stable, requires multiple frames before confirmation).
- 🎥 Real-time detection from webcam.
- 🟢 Stable labels with smoothing (no flickering).
- 🔒 Authorized face embeddings loaded from `data/authorized/`.

## 🧭 Architecture

- **Backend**: Python FastAPI (`src/server.py`), OpenCV capture, DeepFace (SFace) embeddings, MJPEG stream, event queue, WebSocket live push.
- **Frontend**: React UI (`frontend/`) showing live feed, latest alert card, history, and a joystick for manual pan/tilt.
- **Storage**: Supabase bucket for intruder snapshots + table for metadata.
- **Alerts**: Telegram Bot message + photo (optional).
- **Hardware (optional)**: ESP32 over serial for PAN/TILT servo control.

## 📁 Project Layout

```
OmniCam/
├─ src/
│  └─ server.py           # FastAPI app, camera capture, recognition, APIs
├─ frontend/
│  └─ src/App.js          # React dashboard (live feed, events, joystick)
├─ data/
│  └─ authorized/         # One folder per person → face images
├─ snapshots/             # Saved annotated snapshots for events
├─ requirements.txt       # Python dependencies
└─ README.md
```

## ✅ Requirements

- Python 3.10+
- pip
- Windows with a webcam (project tunes capture backends for Windows)
- Optional: Node 18+ (for running the React UI)

## 🔐 Environment variables

Create a `.env` file in the project root:

```
SUPABASE_URL=your_supabase_project_url
SUPABASE_KEY=your_supabase_service_or_anon_key

# Optional: Telegram alerts
TELEGRAM_BOT_TOKEN=123456:abc...
TELEGRAM_CHAT_ID=your_chat_id
```

Notes:

- Configure a Supabase storage bucket named `intruder-photos` and a table `intruders` to store metadata (see code in `save_intruder`).
- Telegram is optional; if not set, alerts are skipped.
- ESP32 is optional; if not connected, servo API will return an error.

## 📸 Prepare authorized faces

Place images under `data/authorized/<PERSON_NAME>/image1.jpg` (and more images). Example:

```
data/
  authorized/
    Alice/
      1.jpg
      2.jpg
    Bob/
      a.png
      b.png
```

Embeddings are built at startup from these folders.

## ▶️ Run the backend

Install Python deps and start the API server:

```
pip install -r requirements.txt
uvicorn src.server:app --host 0.0.0.0 --port 8000 --reload
```

On first run, the app scans for working cameras across Windows backends (Media Foundation, DirectShow). If none are found, it uses a fallback configuration.

### Key endpoints

- `GET /camera_feed/{camera_id}`: MJPEG stream (use 1 for first camera). Example: `http://localhost:8000/camera_feed/1`
- `GET /cameras`: List detected cameras
- `GET /cameras/refresh`: Rescan cameras
- `GET /events`: Last 50 events
- `GET /events/latest`: Most recent event
- `GET /snapshot/{event_id}`: Fetch saved snapshot by event id
- `GET /flags`: Current flags per identity (`1` authorized, `-1` intruder)
- `GET /stats`: System stats (counts, active cameras, ws clients)
- `WS /ws/events`: WebSocket stream of live events
- `POST /servo_control` JSON `{ "direction": "PAN:LEFT" }` or `PAN:RIGHT`, `TILT:UP`, `TILT:DOWN` (requires ESP32)

## 🖥️ View the system

You can quickly test without the React app:

- Open `http://localhost:8000/camera_feed/1` in a browser to see the MJPEG stream.
- Poll `http://localhost:8000/events` to see detections, or connect to `ws://localhost:8000/ws/events` to receive live events.

## 🌐 Frontend (React) [optional]

The React dashboard in `frontend/` consumes the API and WebSocket:

- Live stream embedded via `<iframe>` from `/camera_feed/1`
- Latest intruder alert card with snapshot
- Event history with thumbnails
- Joystick for manual servo control (POST `/servo_control`)

If you want to run it as a dev server, initialize a React tooling setup (e.g., Vite/Cra) in `frontend/` or serve with your existing workspace setup, ensuring it runs on `http://localhost:3000` (CORS is configured for that origin). Update the UI to your environment as needed.

## 🔧 Troubleshooting

- "No working cameras found": Ensure the webcam is available and not used by another app; try running as admin on Windows.
- MJPEG not loading in the browser: Check firewall rules for port 8000; verify `uvicorn` logs for stream output.
- Supabase upload fails: Confirm bucket `intruder-photos` exists and storage policies allow uploads; verify `SUPABASE_URL`/`SUPABASE_KEY`.
- Telegram not sending: Verify `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID` and that the bot can message the chat.
- Servo control fails: Confirm the correct COM port in `src/server.py` (default `COM5`) and that ESP32 is connected at 115200 baud.

## 🧰 Tech Stack

- Python, FastAPI, OpenCV, DeepFace (SFace)
- React, WebSockets
- Supabase Storage + Table
- Telegram Bot API
- Optional: ESP32 over Serial for PAN/TILT
