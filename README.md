# vocal-isolator-speaker
Raspberry Pi 4 speaker system using Facebook's Demucs AI model for real-time stem separation. Physical knobs control vocals, drums, bass, and melody independently via MCP3008 ADC.
# 🔊 AI Vocal Isolator Speaker

> A Bluetooth speaker that separates any song into its individual stems — vocals, drums, bass, and melody — in real time using Facebook's Demucs neural network. Five physical knobs control each stem's volume independently. Turn the instrumental knob to zero: only the singer's voice remains.

## 🚧 Status
`🔧 In Progress` — Target: July 2026

---

## Demo
> *Demo video coming — will show a Hindi film song playing with instrumental faded out leaving only isolated vocals*

---

## The Idea

Studio mixing desks separate instruments using individual recorded tracks. This speaker does it for any song, in real time, after it's already been mixed — using AI.

Play a Kishore Kumar song over Bluetooth. Fade out the instrumental knob. Hear only his voice, clean and isolated, no background music. No karaoke track required.

---

## How Demucs Works

```
Input audio (mixed song)
        │
        ▼
   STFT Transform
   (audio → spectrogram: time × frequency 2D map)
        │
        ▼
   U-Net Neural Network
   ┌────────────────────────┐
   │  Encoder               │  ← Compresses spectrogram, learns features
   │  Bottleneck            │  ← Hidden representation
   │  Decoder               │  ← Reconstructs 4 separate stem masks
   └────────────────────────┘
        │
        ├──▶ Vocals mask  ×  original spectrogram = Vocals stem
        ├──▶ Drums mask   ×  original spectrogram = Drums stem
        ├──▶ Bass mask    ×  original spectrogram = Bass stem
        └──▶ Other mask   ×  original spectrogram = Melody/Other stem
```

Demucs was trained on thousands of songs where the original separate studio tracks were known. It learned what a vocal spectrogram looks like vs. a bass spectrogram. The "masks" are its learned answer to: *"which frequencies, at which moments in time, belong to which instrument?"*

---

## System Architecture

```
Phone (Bluetooth audio source)
        │
        ▼
Raspberry Pi 4 (2GB)
  ├── bluez Bluetooth stack receives audio stream
  ├── Python chunks audio into ~5-second windows
  ├── demucs.separate() → 4 stem arrays
  ├── spidev reads MCP3008 → 5 knob positions (0.0 – 1.0)
  ├── Each stem × knob scalar = scaled stem
  ├── Sum all 4 scaled stems = mixed output
  └── python-sounddevice → ALSA → I2S

        │  I2S
        ▼
PCM5102A DAC  ──▶  PAM8403 Amplifier  ──▶  2× 3W Speakers

MCP3008 ADC (SPI)
  ├── CH0: Vocals knob
  ├── CH1: Drums knob
  ├── CH2: Bass knob
  ├── CH3: Melody/Other knob
  └── CH4: Master volume knob
```

---

## Components

| Component | Spec | Purpose |
|-----------|------|---------|
| Raspberry Pi 4 Model B | 2GB RAM minimum | AI inference + audio I/O |
| MicroSD card 32GB | Samsung Class 10 | OS + Demucs model (~300MB) |
| PCM5102A I2S DAC | 32-bit, high SNR | Digital to analog conversion |
| PAM8403 amplifier | 3W+3W stereo class D | Speaker drive |
| 3W 4Ω full-range speaker ×2 | Stereo pair | Audio output |
| MCP3008 SPI ADC | 8-channel | Reads all 5 potentiometers |
| 10kΩ linear pot ×5 | B10K panel-mount | Stem volume knobs |
| Aluminium knob caps ×5 | 6mm shaft, 20mm dia | Physical knob feel |
| 3.5mm stereo jack | PJ-307 | Headphone output option |
| USB-C 5V 3A power supply | Official Pi 4 spec | Power — underpowering causes crashes |
| 3mm acrylic front panel | A4 sheet, drilled | Knob mounting panel |

**Approximate build cost: ₹7,100 – ₹9,300 (Pi 4 version)**

> 💡 **Budget alternative:** Raspberry Pi Zero 2W + Spleeter model (₹2,200 – ₹3,500 total) with ~3 second processing latency. Same enclosure, same knobs, swap the brain.

---

## Software Stack

```bash
# Install Demucs
pip install demucs

# Audio I/O
pip install sounddevice

# SPI for knob reading
pip install spidev RPi.GPIO
```

**Python pipeline:**
```python
# Conceptual flow — full code in /src
audio_chunk = capture_bluetooth_audio(seconds=5)
stems = demucs.separate(audio_chunk)  # returns dict: vocals, drums, bass, other

knobs = read_mcp3008()  # returns 5 float values 0.0–1.0
mixed = sum(stems[k] * knobs[k] for k in stems)
play_audio(mixed)
```

---

## Build Timeline

| Week | Milestone |
|------|-----------|
| 1 | Pi 4 setup, Demucs installed, first stem separation from a file |
| 2 | MCP3008 knob interface working, Python mixing script live |
| 3 | Bluetooth audio input connected, full chain end-to-end |
| 4 | Enclosure built, clean wiring, demo video recorded |

---

## Skills Demonstrated

- Raspberry Pi 4 Linux audio pipeline (ALSA, I2S, bluez)
- I2S DAC interfacing (PCM5102A)
- SPI ADC reading (MCP3008) with Python
- Real-time AI inference on embedded Linux (Demucs / PyTorch)
- Audio DSP — chunked processing, stem mixing, latency management
- Physical hardware interface design (knob panel, enclosure)
- Bluetooth audio streaming (A2DP profile, bluez)

---

## Enclosure Design

5 knobs in a row on an acrylic or wooden front panel. Speaker grille on the front face. Retro mixer aesthetic — think small desktop studio monitor with a mixing console panel.

---

## License
MIT
