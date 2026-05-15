# IQ Viewer WLX — HackRF IQ Data Viewer for Total Commander

A Total Commander WLX64 Lister plugin that displays HackRF IQ data files
with real-time PSD spectrum and waterfall spectrogram visualization.

## Features

- **PSD Spectrum**: FFT-based power spectral density with Hann window and segment averaging
- **Waterfall Spectrogram**: Scrolling waterfall display with Viridis colormap
- **CUDA Acceleration**: Automatic GPU acceleration via cuFFT with CPU fallback
- **OpenGL Rendering**: Hardware-accelerated rendering with MSAA anti-aliasing
- **Playback Controls**: Play/Stop, Reset, position slider for scrubbing through files
- **Settings Dialog**: Adjustable sample rate, center frequency, FFT sizes, dB range
- **Metadata Support**: Reads `.txt` sidecar files for sample rate, center frequency, signal annotations
- **Thumbnail Preview**: Quick PSD+waterfall preview in TC file panels
- **Multiple Formats**: `.iq`, `.raw`, `.cs8` (int8), `.cs16` (int16), `.cf32` (float32)

## Building

### Requirements

- **Lazarus IDE** 2.2+ with Free Pascal Compiler 3.2+
- **Target**: x86_64-win64 (64-bit Windows)
- No LCL required — pure Win32 API + OpenGL

### Build Steps

1. Open `iq_viewer.lpi` in Lazarus IDE
2. Select build mode **Release** (or **Debug** for development)
3. Build → `Compile` (Ctrl+F9)
4. Output: `iq_viewer.wlx64`

### Command Line Build

```bash
lazbuild --build-mode=Release iq_viewer.lpi
```

Or directly with FPC:

```bash
fpc -MObjFPC -dRELEASE -O3 -Xs -XX -CX \
    -Twin64 -Px86_64 \
    iq_viewer.lpr
```

Rename the output `.dll` to `.wlx64`.

## Installation

### Automatic

1. Double-click `pluginst.inf` in Total Commander
2. TC will install the plugin automatically

### Manual

1. Copy `iq_viewer.wlx64` to a permanent location
2. In TC: Configuration → Options → Plugins → Lister Plugins
3. Click "Add" and select `iq_viewer.wlx64`
4. Detection string: `EXT="IQ" | EXT="RAW" | EXT="CF32" | EXT="CS8" | EXT="CS16"`

## Usage

1. Select an IQ file in Total Commander
2. Press **F3** (Quick View) or **Ctrl+Q** (Lister panel)
3. Use the toolbar buttons:
   - **▶ Play**: Start/stop playback through the file
   - **↺ Reset**: Jump back to the beginning
   - **⚙ Settings**: Open the settings dialog
4. Drag the position slider to scrub through the file

## Settings Dialog

| Parameter | Default | Description |
|-----------|---------|-------------|
| Sample Rate | 2 MHz | Sample rate in Hz |
| Center Freq | 0 | Center frequency in Hz (0 = baseband) |
| PSD NFFT | 2048 | FFT size for PSD display |
| WF NFFT | 256 | FFT size for waterfall rows |
| WF History | 150 | Number of waterfall rows visible |
| WF Overlap | 0.0 | Overlap between waterfall windows (0..0.95) |
| Min dB | -10 | Minimum of color/magnitude scale |
| Max dB | 45 | Maximum of color/magnitude scale |
| Chunk (sec) | 0.2 | Duration of each playback chunk |
| Avg Segments | 1 | Number of FFT segments averaged for PSD |

## Metadata Sidecar Files

Place a `.txt` file next to the IQ file with the same base name:

```
sample_rate_hz=2000000
center_freq_hz=433920000
signals=
  [0] modulation=fm offset_hz=0 amplitude=0.5 params={"deviation": 5000}
  [1] modulation=am offset_hz=50000 amplitude=0.3 params={}
```

The plugin reads `sample_rate_hz` and `center_freq_hz` automatically and
shifts the frequency axis accordingly.

## CUDA Support

The plugin dynamically loads NVIDIA CUDA libraries at startup:
- `cudart64_12.dll` (fallback to versions 11, 10)
- `cufft64_11.dll` (fallback to version 10)

If CUDA is unavailable, all DSP computation runs on CPU using a
Cooley-Tukey radix-2 FFT implementation. The status bar shows `[GPU:...]`
or `[CPU]` to indicate which backend is active.

## Project Structure

```
iq_viewer.lpr       — DLL entry point with WLX64 exports
iq_viewer.lpi       — Lazarus project file
uGlobals.pas        — Constants, types, dark theme colors
uMetadata.pas       — Sidecar .txt metadata parser
uIQData.pas         — IQ file loading (int8/int16/float32)
uFFT.pas            — CPU FFT engine (Cooley-Tukey radix-2)
uCudaFFT.pas        — CUDA cuFFT wrapper with dynamic loading
uColormap.pas       — Viridis colormap implementation
uWinGL.pas          — OpenGL/WGL bindings (no LCL)
uGLRenderer.pas     — OpenGL spectrum + waterfall renderer
uViewerForm.pas     — Main viewer window (pure Win32)
uThumbnail.pas      — Thumbnail bitmap generator
pluginst.inf        — TC plugin installation descriptor
```

## License

This project is provided as-is for personal use.
