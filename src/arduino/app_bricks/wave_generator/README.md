# Wave Generator Brick

The Wave Generator Brick provides continuous wave generation for real-time audio synthesis. It supports multiple waveform types with smooth transitions, making it ideal for creating alarms, musical tones, or audio feedback in your Arduino projects.

## Overview
The Wave Generator brick allows you to:

- Generate continuous audio waveforms in real-time
- Select between different waveform types (sine, square, sawtooth, triangle)
- Control frequency and amplitude dynamically during playback
- Configure smooth transitions with attack, release, and glide (portamento) parameters
- Stream audio to USB speakers with minimal latency

It runs continuously in a background thread, producing audio blocks at a steady rate with configurable envelope parameters for professional-sounding synthesis.

## Features

- **Multiple Waveforms**: Select between sine, square, sawtooth, and triangle waves.
- **Real-Time Control**: Change frequency and amplitude continuously during playback.
- **Smooth Transitions**: Configurable attack, release, and glide parameters prevent audio glitches.
- **Hardware Volume**: Direct control over the USB audio device volume.
- **Low Latency**: Optimized buffering for responsive audio generation.
- **Flexible Output**: Works with an internal auto-configured speaker or an externally provided `Speaker` instance.

## Prerequisites

- **USB-C® Hub** with external power delivery (recommended for stable audio).
- **USB Audio Device**: A USB speaker, headset, or USB-C® to 3.5mm adapter connected to the hub.
- Arduino UNO Q running in **Network Mode** or **SBC Mode** (USB-C port needed for the hub)


## Code Example and Usage

### Basic Tone Generation

This example generates a simple 440 Hz sine wave (A4 note).

```python
from arduino.app_bricks.wave_generator import WaveGenerator
from arduino.app_utils import App

# Initialize generator (defaults to Sine wave)
wave_gen = WaveGenerator()

# Start the audio stream
App.start_brick(wave_gen)

# Set frequency to A4 (440 Hz) and volume amplitude to 80%
wave_gen.set_frequency(440.0)
wave_gen.set_amplitude(0.8)

# Run the app (audio continues in background)
App.run()
```

### Dynamic Waveform & Envelope Control

This example demonstrates how to change waveforms and envelope parameters to create different sound effects.

```python
import time
from arduino.app_bricks.wave_generator import WaveGenerator
from arduino.app_utils import App

# Initialize with specific envelope settings
wave_gen = WaveGenerator(
    wave_type="square",
    attack=0.1,   # Slow fade-in
    release=0.1,  # Slow fade-out
    glide=0.05    # Slide between frequencies (portamento)
)

App.start_brick(wave_gen)

def loop():
    # Play low tone
    wave_gen.set_frequency(220.0)
    wave_gen.set_amplitude(1.0)
    time.sleep(1)
    
    # Slide to high tone
    wave_gen.set_frequency(880.0)
    time.sleep(1)
    
    # Change timbre
    wave_gen.set_wave_type("triangle")
    
    # Fade out
    wave_gen.set_amplitude(0.0)
    time.sleep(1)

App.run(loop)
```

### Using an External Speaker Instance

For advanced use cases where you need to share a `Speaker` instance or configure specific hardware parameters, you can pass it to the brick.

```python
from arduino.app_bricks.wave_generator import WaveGenerator
from arduino.app_peripherals.speaker import Speaker
from arduino.app_utils import App

# Configure Speaker manually for low latency
speaker = Speaker(
    device=Speaker.USB_SPEAKER_1,
    sample_rate=16000,
    channels=1,
    format="FLOAT_LE",
    periodsize=480,  # Match block size (16000 * 0.03s)
    queue_maxsize=10
)

# You must manage the lifecycle of external speakers
speaker.start()

# Pass the speaker to the generator
wave_gen = WaveGenerator(sample_rate=16000, speaker=speaker)
App.start_brick(wave_gen)

wave_gen.set_frequency(440.0)
wave_gen.set_amplitude(0.5)

App.run()

# Cleanup
speaker.stop()
```

## Configuration

The Brick is initialized with the following parameters:

| Parameter        | Type      | Default  | Description                                                                    |
| :--------------- | :-------- | :------- | :----------------------------------------------------------------------------- |
| `sample_rate`    | `int`     | `16000`  | Audio sample rate in Hz.                                                       |
| `wave_type`      | `str`     | `"sine"` | Initial waveform: `"sine"`, `"square"`, `"sawtooth"`, `"triangle"`.            |
| `block_duration` | `float`   | `0.01`   | Duration of each audio processing block in seconds (lower = lower latency).    |
| `attack`         | `float`   | `0.01`   | Time in seconds for amplitude to rise to target value.                         |
| `release`        | `float`   | `0.03`   | Time in seconds for amplitude to fall to target value.                         |
| `glide`          | `float`   | `0.02`   | Time in seconds for frequency to slide to target value (portamento).           |
| `speaker`        | `Speaker` | `None`   | Optional external `Speaker` instance. If `None`, one is created automatically. |

## Methods

- **`start()`**: Starts the background audio generation thread.
- **`stop()`**: Stops audio generation and releases resources.
- **`set_frequency(hz)`**: Sets the target frequency.
- **`set_amplitude(0.0-1.0)`**: Sets the target amplitude (volume).
- **`set_wave_type(type)`**: Switches the waveform type.
- **`set_volume(0-100)`**: Sets the system hardware volume for the speaker.
- **`set_envelope_params(attack, release, glide)`**: Updates smoothing parameters at runtime.

## Understanding Wave Generation

The Wave Generator produces sound by mathematically calculating samples for a given waveform shape.

- **Frequency (Hz)**: Controls the pitch. Audible range is roughly 20 Hz (low rumble) to 20,000 Hz (high piercing), though 16,000 Hz is the limit for the default sample rate.
- **Amplitude (0.0 - 1.0)**: Controls the signal strength (loudness).
- **Glide (Portamento)**: When you change frequency, `glide` determines how long it takes to slide to the new note. A value of `0` jumps instantly (which can cause clicks), while higher values create a "swooping" effect.
