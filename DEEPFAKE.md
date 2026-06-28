# Pi-Station — Deepfake Prevention Architecture

## The problem

Deepfake technology has reached the point where fabricating convincing audio and video of real people is accessible to motivated individuals with commodity hardware. For broadcasters and journalists, this creates a fundamental trust problem: how do you prove that footage captured at a real event is authentic?

Existing solutions address individual photos and short clips. None address continuous long-form event capture — a two-hour panel discussion, a keynote, a press briefing. None have a concept of a *session* as a provenance unit. None combine audio and video authenticity simultaneously. None are designed for offline-resilient deployment on edge hardware.

Pi-Station is designed from the ground up for this use case.

---

## The six-layer provenance stack

### Layer 1 — SHA-256 chunk hashing

Every 30-second WAV and MP4 chunk is hashed with SHA-256 the moment it closes.

**What it proves:** any byte modified after capture breaks the hash.
**Verifiable by:** anyone, offline, using `sha256sum`.
**Status:** ✅ Active.

### Layer 2 — Session manifest (Ed25519 signed)

At session stop, Pi-Station generates a manifest: an ordered list of every chunk's hash, device ID, session ID, start/stop time — signed with Ed25519.

**What it proves:** the complete session is intact and ordered correctly. Removing or reordering any chunk breaks the signature.
**Verifiable by:** anyone with the public key, offline.
**Status:** ✅ Active.
**Planned upgrade:** ML-DSA-65 (NIST FIPS 204) — quantum-resistant.

### Layer 3 — DV token per chunk (planned)

Each chunk receives a server-issued cryptographic token — revocable, auditable, independently verifiable. Crypto-agile: algorithm is a field in the token, not hardcoded.

**What it proves:** authenticated origin, revocable provenance, mandatory audit trail.
**Status:** Planned.

### Layer 4 — Audio and video watermark (planned)

**AudioSeal** (Meta FAIR, MIT): inaudible watermark per WAV chunk. Fragile + reversible. Sample-level tamper localisation.

**VideoSeal ChunkySeal** (Meta FAIR, MIT): invisible watermark per MP4 chunk. Frame-level tamper localisation.

**LAVA** (2026): joint audio-visual watermark. Detects audio-video decoupling — the most common deepfake technique. AP = 0.999.

**What it proves:** which region was edited, at sample/frame precision. Whether audio and video were decoupled after capture.
**Status:** Planned (P-W1, P-W2, P-W3).

### Layer 5 — Coupling mark + physical decoder (planned)

**StegaStamp** (Stanford, CVPR 2020): steganographic mark in every video frame. Invisible to viewers. Encodes 96-bit payload per frame: session ID, chunk index, hash, device ID, timestamp. Survives H.264 recompression, social media upload, even rephotography from a screen.

**The decoder ("the torch"):** a proprietary HDMI passthrough device. Point any HDMI source through it — the mark is decoded in real time and overlaid as an authentication badge:

```
🔒 MeetPaper Station
Session  VI-2026-06-28-7999fcf9
Status   ✅ AUTHENTIC
```

If edited:
```
🔒 MeetPaper Station
Status   ⚠️ MARK CORRUPTED · Frames 1,340–1,680 (seconds 44–56)
```

**What it proves:** physical coupling — footage came from a real Pi-Station device. Human-visible — no software required.
**Status:** Planned (P5).

### Layer 6 — Controlled degradation (planned)

The highest-information bytes of each video chunk (DCT high-energy coefficients) are withheld before storage. Withheld bytes live only at the authorisation server — not on the Pi, not in S3.

The distributed video is watchable but not broadcast-quality. Full quality requires `POST /reconstruct` — logged, auditable, revocable.

**What it proves:** every use of broadcast-quality material is tracked. Revocation makes all existing degraded copies permanently unusable.
**Status:** Planned (P6).

---

## The competitive landscape

| Tool | Long-form | Chunk signing | Session aggregate | Audio | Deepfake detection |
|---|---|---|---|---|---|
| Truepic | ✗ | ✗ | ✗ | ✗ | ✗ |
| ProofMode | ✗ | ✗ | ✗ | ✗ | ✗ |
| eyeWitness | ✗ | ✗ | ✗ | ✅ | ✗ |
| Sony/Nikon/Leica | ✗ | ✗ | ✗ | ✗ | ✗ |
| Starling Lab | Partial | ✗ | ✗ | ✅ | ✗ |
| C2PA v2.3 | ✅ | ✅ | ✗ | ✅ | ✗ |
| **Pi-Station** | **✅** | **✅** | **✅** | **✅** | **✅ (planned)** |

---

## For broadcasters — the verification chain

1. **Ingest** — mark verified + DV token checked + manifest verified. Logged.
2. **Edit suite** — HDMI decoder always on. Mark disappears on any edit. Editor sees exactly which frames were modified.
3. **Pre-transmission QC** — final cut through decoder before broadcast.
4. **Random post-broadcast spot-checks** — unpredictable, mandatory, logged.
5. **Third-party verification** — regulator or court with a licensed decoder. No broadcaster cooperation required.

---

## References

- AudioSeal — [github.com/facebookresearch/audioseal](https://github.com/facebookresearch/audioseal)
- VideoSeal — [github.com/facebookresearch/videoseal](https://github.com/facebookresearch/videoseal)
- StegaStamp — [github.com/tancik/StegaStamp](https://github.com/tancik/StegaStamp)
- NIST FIPS 204 (ML-DSA) — Post-quantum digital signature standard, August 2024
- C2PA v2.3 — Content Credentials for live video streaming
- LAVA: Layered Audio-Visual Anti-tampering Watermarking (2026)
