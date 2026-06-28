# Hardware Setup

## Bill of materials (~£230 total)

| Component | Part | Cost |
|---|---|---|
| Compute | Raspberry Pi 5 (4GB) | £55 |
| Camera | Raspberry Pi Camera Module 3 | £25 |
| AI accelerator | Raspberry Pi AI HAT+ | £70 |
| Microphone | M-305 USB condenser mic | £20 |
| Pan/tilt | PCA9685 + MG996R + SG90 | £15 |
| Power | USB-C power bank 27W PD | £35 |
| Storage | microSD 64GB (A2 rated) | £10 |

---

## Pi setup

### 1. Flash

Use [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
- OS: Raspberry Pi OS Lite (64-bit, Bookworm)
- Hostname: `pistation` · Username: `pistation`
- Enable SSH · Configure Wi-Fi

### 2. Verify audio

```bash
arecord -l
# M-305 USB mic appears as card 2 (verify on your unit)

arecord -D plughw:2,0 -f S16_LE -r 16000 -c 1 -d 5 test.wav
aplay test.wav
```

Set `AUDIO_DEVICE=plughw:2,0` in `.env`.

### 3. Verify camera

```bash
libcamera-hello --list-cameras
libcamera-still -o test.jpg
```

### 4. AI HAT+ (Hailo-8)

Follow the [official AI HAT+ guide](https://www.raspberrypi.com/documentation/accessories/ai-hat-plus.html), then:

```bash
hailotools device-inspect   # confirm detected
```

Set `FACE_DETECTION=hailo` in `.env`.

### 5. PCA9685 pan/tilt wiring

```
PCA9685 → Pi GPIO
VCC     → 5V  (pin 2)
GND     → GND (pin 6)
SDA     → GPIO 2 / SDA (pin 3)
SCL     → GPIO 3 / SCL (pin 5)
```

Channel 0 → MG996R (pan) · Channel 1 → SG90 (tilt)

**⚠️ Servo power must come from the power bank — not from Pi GPIO (3.3V, insufficient current).**

```bash
sudo raspi-config   # Interface Options → I2C → Enable
sudo i2cdetect -y 1   # should show 0x40
```

Set `PAN_TILT=pca9685` in `.env`.

---

## Provision + deploy

```bash
bash scripts/provision-pi.sh pistation@pistation.local
bash scripts/build-frontend.sh
bash scripts/deploy-pi.sh pistation.local
```

---

## Troubleshooting

**Can't connect to pistation.local** — Pi and Mac must be on the same network.
```bash
arp -a | grep -i raspberry   # find the IP directly
```

**initramfs on boot** — SD card corruption. At the prompt:
```bash
fsck /dev/mmcblk0p2
exit
```

**Audio not capturing** — check `arecord -l` and update `AUDIO_DEVICE` in `.env`.
