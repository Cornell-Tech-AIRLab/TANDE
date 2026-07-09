# AI Avatar to Support Young Adults in Emotion Regulation with Positive Self-Talk

This repository contains the interactive app only: a browser-based 3D avatar interface with speech-driven conversation, optional webcam emotion mirroring, text-to-speech, and lip-sync.

## Overview

- `FAFrontend/r3f-virtual-therapist-frontend`: React + Vite frontend with the 3D avatar, microphone controls, webcam FER, and experiment controls for backchannel behavior.
- `FABackend/r3f-virtual-therapist-backend`: Node/Express backend that generates dialogue, synthesizes speech, and produces lip-sync timing data.

## How the App Works

1. The user speaks through the browser microphone using the Web Speech API.
2. The frontend sends the transcript and dialogue history to the backend.
3. The backend requests a structured reply containing `text`, `facialExpression`, and `animation`.
4. The reply is converted to speech using ElevenLabs when available, or OpenAI TTS otherwise.
5. The generated audio is processed with `ffmpeg` and Rhubarb Lip Sync to create mouth cues.
6. The frontend plays the audio and drives avatar animation, facial expressions, and visemes in sync.
7. While the user is speaking, the app can trigger non-verbal and/or verbal backchannels.

When webcam mirroring is enabled, the browser runs face detection with OpenCV and emotion inference with ONNX Runtime Web, then maps detected emotion to avatar facial expression while the avatar is idle.

## Repository Layout

```text
/
├── FAFrontend/
│   └── r3f-virtual-therapist-frontend/
│       ├── public/
│       ├── src/
│       └── vite.config.js
├── FABackend/
│   └── r3f-virtual-therapist-backend/
│       ├── audios/
│       ├── bin/
│       ├── .env.example
│       ├── index.js
│       └── package.json
└── README.md
```

## Requirements

- Node.js 20 or newer recommended
- npm
- `ffmpeg` installed and available on the command line
- A Chromium-based browser recommended for microphone and speech recognition support
- An OpenAI API key for generated dialogue
- Optional: an ElevenLabs API key for alternative speech synthesis
- The FER model file `emotion-ferplus-8.onnx`, placed in the app's `models` folder alongside `emotion-ferplus-8.txt`

## Platform Notes

- The repository includes Rhubarb Lip Sync binaries for macOS and Windows.
- Linux users must provide a compatible Rhubarb binary and update the backend path configuration if lip-sync is required.
- The backend listens on port `3000`.
- The frontend dev server defaults to port `5173`.

## Installation

### 1. Install backend dependencies

```bash
cd FABackend/r3f-virtual-therapist-backend
npm install
cp .env.example .env
```

Then edit `.env` with valid credentials:

```env
OPENAI_API_KEY=your_openai_api_key
ELEVEN_LABS_API_KEY=your_elevenlabs_api_key
OPENAI_TTS_MODEL=tts-1
OPENAI_TTS_VOICE=alloy
ELEVEN_LABS_MODEL_ID=eleven_flash_v2
ELEVEN_LABS_OUTPUT_FORMAT=mp3_44100_128
```

If `ELEVEN_LABS_API_KEY` is missing, the backend falls back to OpenAI TTS. If `OPENAI_API_KEY` is missing, the app still starts, but chat replies use built-in warning/demo responses instead of generated ones.

### 2. Install frontend dependencies

```bash
cd FAFrontend/r3f-virtual-therapist-frontend
npm install
```

### 3. Install `ffmpeg`

```bash
brew install ffmpeg
```

```bash
sudo apt install ffmpeg
```

```powershell
choco install ffmpeg
```

### 4. Add the FER model manually

The webcam emotion-mirroring model is not bundled in this repository because the ONNX file is too large to distribute through our normal upload flow.

Each person setting up the project must place the model file in the same folder as `emotion-ferplus-8.txt`.

If they are working from the built app shown in `dist/`, place it here:

```text
FAFrontend/r3f-virtual-therapist-frontend/dist/models/emotion-ferplus-8.onnx
```

If they are running the app from source in development, place it here:

```text
FAFrontend/r3f-virtual-therapist-frontend/public/models/emotion-ferplus-8.onnx
```

The app can still run without this file, but the webcam emotion-mirroring feature in `VideoFeed.jsx` will fail until the model is added.

## Running the App

### Start the backend

```bash
cd FABackend/r3f-virtual-therapist-backend
npm start
```

For development with auto-reload:

```bash
npm run dev
```

### Start the frontend

```bash
cd FAFrontend/r3f-virtual-therapist-frontend
npm run dev
```

The frontend uses a Vite proxy that forwards `/api/*` to `http://127.0.0.1:3000` during local development.

### Open the app

Open the Vite URL, typically `http://127.0.0.1:5173`, in a Chromium-based browser. Allow microphone access for speech input, and allow camera access only if you want webcam emotion mirroring.

## Included Features

- Speech-driven avatar conversation
- Structured backend replies with animation and expression labels
- Audio playback with lip-sync alignment
- Backchannel modes: control, non-verbal, and verbal + non-verbal
- Optional webcam-based emotion mirroring

## Fallback Behavior

- Without an OpenAI API key, the app returns pre-authored warning/demo utterances.
- If speech synthesis fails, the frontend attempts browser speech synthesis.
- If backchannel TTS fails, the backend falls back to a bundled static `mhm` audio clip.

## External Services and Data Dependencies

- OpenAI is used for dialogue generation and optional TTS.
- ElevenLabs is optionally used for speech synthesis.
- Browser speech recognition depends on the client browser implementation.
- Webcam FER inference uses ONNX Runtime Web and currently loads WASM support from a CDN at runtime.
- Webcam FER inference also requires a manually provided ONNX model file placed alongside `emotion-ferplus-8.txt` in the app's `models` folder.

## Known Limitations

- Speech recognition support is browser-dependent and works best in Chromium-based browsers.
- Lip-sync generation depends on `ffmpeg` and a platform-compatible Rhubarb binary.
- Linux lip-sync support requires manual Rhubarb setup.
- Webcam emotion mirroring depends on camera permission, network availability for ONNX Runtime Web support files, and a manually provided `emotion-ferplus-8.onnx` model in the current implementation.
- This system is a research prototype and should not be interpreted as a clinical or diagnostic tool.

## Ethics and Privacy

- Microphone and camera streams should only be collected with informed user consent.
- The system is intended for research and prototyping in supportive dialogue, not for medical diagnosis, treatment, or emergency intervention.
