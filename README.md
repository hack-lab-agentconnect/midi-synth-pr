# midi-synth

Python MIDI synth. Built by Jilly @ Hackers In The Loop

A real-time Python MIDI synthesizer. It listens to **any** MIDI input device, plays
the notes it receives, and runs them through a dual-oscillator voice engine plus a
toggleable effects chain.

## Features

- MIDI input from **all connected input ports** (or a named port / single channel).
- Polyphony up to **12 simultaneous notes**.
- Waveforms: **sine, square, saw, triangle** (square/saw are band-limited with PolyBLEP).
- **Second oscillator is phase-modulated (FM) by the first oscillator's output.**
- Second oscillator **coarse tuning −12..+12 semitones** plus **fine tuning ±0.5 cents**.
- Toggleable effects: **Chorus, Delay, Reverb, Bitcrush**.
- Pitch-bend and velocity support; per-voice ADSR envelope.

## Install

```bash
python -m venv .venv && . .venv/bin/activate      # optional
pip install -r requirements.txt
```

On Linux the PortAudio system library is also required:

```bash
sudo apt install libportaudio2
```

(macOS/Windows wheels bundle PortAudio.)

## Run

```bash
python run.py --list            # list MIDI inputs and audio outputs
python run.py                   # listen on all MIDI ports, output to default device
python run.py --input "Launchkey" --channel 1
```

An interactive console starts alongside the audio. Type `help`. Commands:

```
fx <chorus|delay|reverb|bitcrush> <on|off|toggle>
wave1/wave2 <sine|square|saw|triangle>
level1/level2 <0-1>  # oscillator mix level (osc2 starts at 0)
fm <0-1>            # oscillator-2 FM amount
tune2 <-12..12>     # oscillator-2 coarse semitones
cents2 <-0.5..0.5>  # oscillator-2 fine cents
gain <0-1.2>
alloff | status | quit
```

> To hear the second oscillator, raise its level: `level2 0.5` (it defaults to 0
> so you get a pure osc-1 tone until you turn it up).

## Default MIDI CC map

| Control | Action |
|---|---|
| CC 1 | FM depth (0..1) |
| CC 7 | master volume |
| CC 20 | toggle Chorus |
| CC 21 | toggle Delay |
| CC 22 | toggle Reverb |
| CC 23 | toggle Bitcrush |
| CC 24 | osc 1 waveform (0-31 sine, 32-63 square, 64-95 saw, 96-127 triangle) |
| CC 25 | osc 2 waveform (same zones) |
| CC 26 | osc 2 coarse tune (−12..+12 semitones) |
| CC 27 | osc 2 fine tune (−0.5..+0.5 cents) |
| CC 28 | osc 2 level (0..1) |
| CC 29 | osc 1 level (0..1) |
| Pitch wheel | pitch bend (±2 semitones) |

Toggle CCs act on press (value ≥ 64) with edge detection.

## Signal flow

```
note ─▶ ADSR ─▶ osc1 ──┬────────────────────────────▶ Σ ─▶ chorus ─▶ delay ─▶ reverb ─▶ bitcrush ─▶ soft clip ─▶ out
                       └─(phase mod ×FM depth)─▶ osc2 ─┘
```

`osc2` pitch = note pitch × 2^((semitones + cents/100)/12).

## Offline render (no audio device)

```bash
python render_demo.py --out demo.wav --effects reverb,delay --wave1 saw --fm 0.4 --level2 0.6
```

Renders a 12-note chord to a WAV file using only NumPy and the standard library.
