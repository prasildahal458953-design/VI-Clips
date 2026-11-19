# YouTube Clip Studio

Made by Rash · Licensed under the [MIT License](./LICENSE).

A native Python desktop tool for creating short clips from a YouTube video or livestream range. It can preserve and enhance the source voice, keep the original audio clean, or replace it with Edge TTS narration, then transcribe the chosen audio into burned-in captions and export an MP4 at the selected dimensions (1080×1920 by default).

Run `install.bat` on Windows to install the Python requirements and create a Desktop shortcut. The installer shortcut description is **YT Clip Studio - Made by Rash**.

## Setup

1. Install Python 3.10 or later.
2. Install [yt-dlp](https://github.com/yt-dlp/yt-dlp) and FFmpeg and make sure both commands are available in `PATH`.
3. For CUDA transcription, install a CUDA-compatible `ctranslate2`/`faster-whisper` setup for your NVIDIA driver. If CUDA cannot be used, Whisper falls back to CPU `int8`.
4. From this directory, install the Python packages:

   ```powershell
   python -m pip install -r requirements.txt
   ```

5. Launch the native desktop UI:

   ```powershell
   python main.py
   ```

   On Linux, install the system Tk package if the Python distribution does not include Tk.

## Workflow

- Paste a YouTube URL, choose **Fetch / Preview** to look up metadata and thumbnail, and enter start/end times as `MM:SS` or `HH:MM:SS` (for example `02:56` normalizes to `00:02:56`). Minutes and seconds must be from 00 to 59.
- Choose an audio mode: **Enhanced Original Voice** keeps the clip audio and applies pitch-preserving shift plus commentary DSP; **Original Audio** keeps it with loudness normalization; **AI Voiceover** replaces it with narration from the script box. **None** leaves original audio untouched. A script and a selected TTS voice are required only for AI voiceover mode.
- **Caption Text Script** defaults to **Romanized (English Letters)** for Hindi/Nepali mixed with English; **Native Script** preserves Whisper's original script. Romanization uses optional `indic-transliteration` when installed and an offline built-in fallback otherwise.
- Choose one of the vertical, horizontal, custom-size, or default-size output options. Custom width/height must be even values between 160 and 3840 pixels. Crop X/Y percentages move the crop window; enter **None** to center that axis.
- **Animation look** offers **Soft Animation** (smooth color with a gentle sharpen) and **Bold Toon** (more contrast, saturation, and detected linework). These are lightweight local FFmpeg filters, not AI generation; keep **None** for the unfiltered source look.
- Caption presets include several display fonts; use **None** to skip transcription and caption burn-in. The independent caption font control can override the selected preset's font.
- The source-preview action opens the fetched YouTube page before generation. Once a clip is ready, use **Preview Export** to watch it and **Export As…** to copy it to a chosen location.
- Stages run sequentially on a worker thread. The UI remains responsive, shows stage progress/log messages, and can cancel a running external command or frame-processing batch. Whisper transcribes the selected source audio with one worker and releases model/GPU resources before FFmpeg starts.
- The studio uses a dark, compact workspace with a larger preview canvas and eased progress updates.
- The **Timeline** tab previews local/exported MP4s using background OpenCV decoding, a frame-accurate seek bar, play/pause, and frame stepping. Drag on the displayed video to set an aspect-locked crop. Add, update, and delete camera keyframes on the timeline, choose linear or smoothstep interpolation, then use **Export Edits** to render that video. **Exports** lists completed MP4s with refresh and double-click-to-load behavior.
- **Punchy Blur Fill** uses a dark, blurred background behind a large crop. **Center Crop (Full 9:16)** fills the frame; **Blur Fill (Keep 16:9 Frame)** places a fitted foreground over a blurred background. **None** fits the source inside the requested dimensions without cropping.
- Captions use timed ASS word/phrase chunks with a bold face, thick outline, lower-middle placement, and selectable accent-color presets.
- Final videos are saved in `workspace/exports/`. Per-job intermediate files are stored under `workspace/temp/`; rotating logs are in `workspace/logs/`.

The app does not download the full video as a separate step: `yt-dlp` requests the selected timestamp range with a 1080p height cap. Livestream and keyframe-at-cuts options are available in the UI.

## Keyboard shortcuts

- **Ctrl+L** focuses the YouTube URL field.
- **Ctrl+Enter** starts clip generation.
- **Esc** requests cancellation while a job is running.

## Architecture

- `main.py` creates workspace folders, checks external tools/CUDA, and starts the desktop window.
- `config/settings.py` holds workspace paths, encoding/model limits, voices, framing names, output bounds, and caption presets.
- `core/` contains independent environment checks, yt-dlp metadata/download handling, Edge TTS, Whisper/ASS captions, FFmpeg rendering, cancellable subprocess handling, and the sequential background pipeline.
- `ui/` contains the settings/script widgets, video metadata preview, threaded exported-video library, OpenCV/Pillow preview timeline, and the native application window. Worker-thread events are queued back to Tk so UI updates remain on the main thread.
- `core/keyframe_camera.py` validates camera keys and builds FFmpeg expressions for smooth crop/zoom interpolation.
