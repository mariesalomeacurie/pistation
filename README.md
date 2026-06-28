# Pi-Station

**The room keeps recording. Even when the internet doesn't.**

Pi-Station is an edge intelligence platform running on Raspberry Pi 5. It captures audio and video continuously at live events, transcribes locally, syncs automatically when connectivity returns — and wraps every session in a cryptographic provenance stack that makes deepfake fabrication practically impossible to pass off as genuine.

Built at the [Agents in the Wild](https://www.londoncalling.guide/agents-in-the-wild) hackathon, London, June 2026.

---

## The problem

Two unsolved problems in live event recording:

**Reliability.** Venue Wi-Fi fails. Cloud-dependent recorders stop. Sessions are lost. Pi-Station records to local storage first — the network is optional.

**Authenticity.** Deepfake technology makes it trivially easy to fabricate convincing audio and video of real people. Broadcasters and journalists have no systematic way to prove that what they captured is what actually happened.

Pi-Station solves both — simultaneously.

---

## The three guarantees

Regardless of network state, a Pi-Station session always produces:

1. **Audio** — WAV, gapless, 30-second rolling chunks, always
2. **Video** — H.264 MP4, 30-second rolling chunks, always
3. **Transcript** — faster-whisper STT, local, private, always

These are written to local storage first. Cloud sync happens automatically when connectivity returns — resumable, idempotent, in four phases.

---

## Deepfake prevention — the six-layer provenance stack

Pi-Station embeds a cryptographic authenticity chain into every session. Six independent layers, each adding a different kind of proof:

```
Layer 1 — SHA-256 hash                          ✅ Active
  Every 30-second chunk hashed at close.
  Any byte changed after capture breaks the hash.
  Verifiable offline by anyone. sha256sum.

Layer 2 — Session manifest (Ed25519 signed)     ✅ Active
  Ordered list of every chunk hash + device ID + timestamps.
  Signed at session stop. Proves complete session is intact.

Layer 3 — DV token per chunk                    Planned
  Server-issued revocable token per chunk.
  Audit log of every verification. ML-DSA-65 quantum-resistant.

Layer 4 — AV watermark                          Planned
  AudioSeal (Meta FAIR) + VideoSeal ChunkySeal per chunk.
  Fragile + reversible. Sample/frame-level tamper localisation.
  LAVA (2026): detects audio/video decoupling — the deepfake tell.

Layer 5 — Coupling mark + HDMI torch            Planned
  StegaStamp steganographic mark in every video frame.
  Invisible to viewers. Decoded only by a licensed HDMI device.
  Human-visible authentication — no technical knowledge needed.

Layer 6 — Controlled degradation                Planned
  Highest-information bytes withheld before distribution.
  Full quality requires authorised server call — logged, auditable.
  Every use of broadcast-quality material is tracked.
```

See [DEEPFAKE.md](DEEPFAKE.md) for the full architecture.

---

## Hardware

| Component | Part |
|---|---|
| Compute | Raspberry Pi 5 (4GB) |
| Camera | Camera Module 3 (imx708, 12MP) |
| AI accelerator | AI HAT+ (Hailo-8, 26 TOPS) |
| Microphone | M-305 USB condenser mic |
| Pan/tilt | PCA9685 + MG996R (pan) + SG90 (tilt) |
| Power | USB-C power bank (27W PD, pass-through) |

The AI HAT+ runs face detection at 30fps with zero CPU overhead, driving pan/tilt servos to track the active speaker automatically.

---

## Architecture

```
Pi-Station (edge)
├── VoiceComponent     USB mic → ElevenLabs live STT or faster-whisper
│                      WAV chunks written every 30 seconds
│                      SHA-256 hash at chunk close
├── VideoComponent     libcamera → H.264 MP4 chunks every 30 seconds
│                      AI HAT+ face detection → pan/tilt speaker tracking
├── SyncService        4-phase resumable cloud sync
│                      1: session manifest  2: transcript segments
│                      3: audio + video chunks (S3 multipart)
│                      4: sync-complete signal
├── State machine      IDLE → PAIRING → READY → RECORDING
│                      → OFFLINE_BUFFERING → SYNCING → REPORT_READY
└── Operator UI        TypeScript frontend at :3456
                       🛡 Authenticity guardrails — Preventing DeepFake
```

---

## Quick start (mock mode — no Pi needed)

```bash
git clone https://github.com/mariesalomeacurie/pistation.git
cd pistation
npm install
cp .env.example .env
npm run dev
# Open http://localhost:3456
```

In mock mode, audio and video sources are simulated. The full state machine, offline buffering, sync phases, and authenticity guardrails panel all work without real hardware.

---

## Deploy to Raspberry Pi

```bash
# First-time provisioning
bash scripts/provision-pi.sh pistation@pistation.local

# Deploy
bash scripts/build-frontend.sh
bash scripts/deploy-pi.sh pistation.local
```

Open `http://pistation.local:3456` on any device on the same network.

---

## Operator UI — four screens

**Pair** → enter session code to pair the station to an event.

**Ready** → confirm Audio and Video components are healthy. Press Start.

**Recording** → live timer, audio level, live transcript, and the **🛡 Authenticity guardrails — Preventing DeepFake** panel showing real-time status of each provenance layer. Network drop → amber "OFFLINE · Audio safe" banner. Reconnect → automatic sync.

**Report Ready** → duration, chunk counts, session report link.

---

## Project structure

```
apps/meet-station/     MeetStation — the first Pi-Station app
  src/capture/         Audio capture, WAV writer, STT providers
  src/components/      VoiceComponent, VideoComponent, SpeakerTracker
  src/control/         HTTP server, routes, operator UI
  src/public/          TypeScript frontend (app.ts)
  src/relay/           Ingest client, queue, relay service
core/                  State machine, sync engine, config, SQLite
hardware/              Hailo face detector, PCA9685 servo, GPIO
scripts/               provision-pi.sh, deploy-pi.sh, build-frontend.sh
docs/                  Architecture, components, sync protocol, hardware
```

---

## Hackathon proof of deployment

Session `VI-2026-06-28-7999fcf9` — recorded at 15:03 BST on 28 June 2026:
- Real Pi 5 hardware · M-305 USB mic · Camera Module 3
- ElevenLabs live transcript active · 4 video chunks captured
- `temp=57.6°C · throttled=0x0` — hardware healthy throughout

---

## Roadmap

- [ ] AudioSeal audio watermark per chunk (Meta FAIR, MIT)
- [ ] VideoSeal ChunkySeal video watermark per chunk (Meta FAIR, MIT)
- [ ] DV token per chunk — revocable, ML-DSA-65 quantum-resistant
- [ ] StegaStamp coupling mark + HDMI decoder torch
- [ ] Controlled degradation — server-gated full quality
- [ ] LAVA joint AV watermark — deepfake decoupling detection
- [ ] Live video preview in recording screen
- [ ] Bluetooth BLE control (no Wi-Fi needed to start recording)

---

## Licence

MIT

---

*Pi-Station is part of the MeetPaper / Foundry365 ecosystem. The MeetPaper cloud backend is not open source.*
