# 👕 Virtual Try-On System

A real-time virtual shirt try-on application built with **Python**, **Streamlit**, **OpenCV**, **MediaPipe**, and **CVZone**. Uses live webcam pose detection to accurately overlay shirt images onto the user's body, scaled and positioned dynamically to match shoulder width.

---

## 📋 Table of Contents

- [Demo](#demo)
- [How It Works](#how-it-works)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the App](#running-the-app)
- [Usage Guide](#usage-guide)
- [Configuration & Tuning](#configuration--tuning)
- [Known Limitations](#known-limitations)
- [Contributing](#contributing)
- [License](#license)

---

## Demo

> Launch the app, allow webcam access, and select a shirt — it overlays in real time, scaling to your body.

---

## How It Works

The system uses **MediaPipe's BlazePose** (via CVZone's `PoseDetector`) to detect 33 body landmarks from the webcam feed. The key landmarks are:

- **Landmark 11** — Left shoulder
- **Landmark 12** — Right shoulder

The horizontal distance between these two points (`lm11[0] - lm12[0]`) drives all shirt scaling:

```
shirt_width  = shoulder_distance × fixedRatio        (fixedRatio = 262/190)
shirt_height = shirt_width × shirtRatioHeightWidth   (ratio = 581/440)
```

The shirt PNG (with transparency) is then overlaid using `cvzone.overlayPNG()`, offset upward and left from the right shoulder landmark to align with the chest.

Shirt navigation (database mode) is **gesture-controlled**: raising the right wrist (`lm16`) to the left triggers a left-to-right shirt change, and raising the left wrist (`lm15`) to the right triggers the reverse — using a circular progress indicator drawn with `cv2.ellipse()`.

---

## Features

- 📸 **Real-time webcam pose detection** via MediaPipe BlazePose
- 👕 **Dynamic shirt scaling** — auto-adjusts to shoulder width and body scale
- 🗂️ **Database mode** — browse a local folder of pre-loaded shirt PNGs
- 📤 **Upload mode** — try on any custom PNG shirt uploaded by the user
- 🤙 **Gesture-based navigation** — switch shirts with hand gestures (no click needed)
- 🖥️ **Streamlit web UI** — browser-based, no desktop GUI framework needed
- 🐧 **Linux/cloud compatible** — includes `libgl1-mesa-glx` for headless OpenCV

---

## Tech Stack

| Library | Version | Role |
|---|---|---|
| `streamlit` | latest | Web UI framework |
| `opencv-python` | 4.10.0.84 | Webcam capture & image processing |
| `mediapipe` | 0.10.14 | Pose landmark detection (BlazePose) |
| `cvzone` | 1.6.1 | Pose detector wrapper & PNG overlay |
| `Pillow` | 10.2.0 | Image loading for uploaded shirts |
| `numpy` | latest | Array operations for image conversion |
| `libgl1-mesa-glx` | system | OpenGL dependency for OpenCV on Linux |

---

## Project Structure

```
Virtual_Try_On_System/
│
├── app.py                      # Main Streamlit application
├── requirements.txt            # Python dependencies
├── packages.txt                # System-level dependencies (for Streamlit Cloud)
├── README.md
│
└── Resources/
    ├── Shirts/                 # Pre-loaded shirt PNG files (transparent background)
    │   ├── shirt1.png
    │   ├── shirt2.png
    │   └── ...
    └── button.png              # Navigation button overlay image
```

> **Note:** The `Resources/` folder is not included in the repo. You must create it and populate it with shirt PNGs before running locally.

---

## Getting Started

### Prerequisites

- Python **3.8 – 3.11** (MediaPipe 0.10.14 does not support Python 3.12+)
- A working **webcam**
- On Linux: `libgl1-mesa-glx` system package

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/RajendraPIssar/Virtual_Try_On_System.git
cd Virtual_Try_On_System

# 2. (Recommended) Create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install system dependency (Linux only)
sudo apt-get install -y libgl1-mesa-glx

# 4. Install Python dependencies
pip install -r requirements.txt
```

### Set Up Resources

```bash
# Create the required directory structure
mkdir -p Resources/Shirts

# Add shirt PNGs (must have transparent backgrounds - RGBA)
# Example: copy your PNG files into Resources/Shirts/
cp ~/my_shirts/*.png Resources/Shirts/

# Add a navigation button image
# Example: a circular arrow PNG at Resources/button.png
```

### Running the App

```bash
streamlit run app.py
```

The app will open at `http://localhost:8501` in your browser.

---

## Usage Guide

1. **Launch** the app via `streamlit run app.py`.
2. Choose a mode:
   - **"Try on Normal Database"** — Activates the webcam and loads shirts from `Resources/Shirts/`.
   - **"Upload the Shirt You Want to Try On"** — Upload a custom PNG (with transparent background), then click **Start Webcam**.
3. **Stand in frame** so your full upper body is visible.
4. **Navigate shirts** (database mode only) using gestures:
   - Raise your **right hand to the left** side → next shirt
   - Raise your **left hand to the right** side → previous shirt
   - A circular progress arc confirms the gesture selection
5. Click **"Stop Webcam"** to end the session.

---

## Configuration & Tuning

Key constants in `app.py` that you may want to adjust:

| Constant | Default | Purpose |
|---|---|---|
| `fixedRatio` | `262 / 190 ≈ 1.38` | Scales shirt width relative to shoulder distance |
| `shirtRatioHeightWidth` | `581 / 440 ≈ 1.32` | Maintains shirt aspect ratio |
| `selectionSpeed` | `10` | Degrees per frame for gesture progress arc |
| `offset` | `(44, 48) × scale` | Fine-tunes shirt vertical/horizontal alignment |
| Webcam resolution | `1420 × 1080` | Set via `cap.set()` — adjust for your camera |
| Gesture zones | `lm16[0] < 300`, `lm15[0] > 900` | Pixel thresholds for left/right gesture detection |

For best results, ensure your **shirt PNGs**:
- Have a **transparent background** (RGBA, not RGB)
- Are captured/designed at a consistent aspect ratio matching `shirtRatioHeightWidth`
- Are high resolution to avoid pixelation after upscaling

---

## Known Limitations

- **Single person only** — designed for one user in frame; multiple people will cause unpredictable overlay behavior.
- **Gesture zones are resolution-dependent** — the hardcoded pixel thresholds (`< 300`, `> 900`) assume a ~1420px-wide frame; recalibrate if using a different resolution.
- **No depth/3D awareness** — shirt scale is derived purely from 2D shoulder distance; leaning forward/back or rotating will cause misalignment.
- **Streamlit webcam loop** — `st.empty()` frame streaming is not optimized for production; latency will increase on lower-end hardware.
- **Shirt PNG quality** — poorly masked or non-RGBA PNGs will render with black/white backgrounds instead of transparency.
- **Streamlit Cloud deployment** — live webcam access requires `cv2.VideoCapture(0)`, which is not supported on most cloud hosting platforms. For cloud deployment, consider migrating to a browser-based WebRTC solution.

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push and open a Pull Request

Please ensure shirt PNGs in any PR are free to use / properly licensed.

---

## License

This project is open source and available under the [MIT License](LICENSE).

---

## Author

**Rajendra P. Issar**
[GitHub Profile](https://github.com/RajendraPIssar)
