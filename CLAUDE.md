# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a computer vision project that uses MediaPipe for real-time hand tracking and finger detection via webcam. The project is implemented entirely in a Jupyter notebook (`finger_tracking.ipynb`) for interactive development and visualization.

## Environment Setup

The project uses Python 3.12 (venv312) with the following key dependencies:
- mediapipe==0.10.21 (hand landmark detection)
- opencv-python==4.11.0.86 (video capture and image processing)
- numpy==1.26.4 (numerical operations)
- matplotlib==3.10.8 (visualization in notebook)
- jupyter/ipython (notebook environment)
- scipy==1.16.3 (scientific computing)
- sounddevice==0.5.3 (audio I/O, possibly for future features)

**Activate the environment:**
```bash
source venv312/bin/activate
```

**Install dependencies:**
```bash
pip install mediapipe opencv-python numpy matplotlib ipython jupyter scipy sounddevice
```

## Running the Project

This project runs entirely in Jupyter notebooks. To start:

```bash
source venv312/bin/activate
jupyter notebook finger_tracking.ipynb
```

Execute cells sequentially. The main tracking loop captures 300 frames and displays real-time hand tracking with a green dot on the index fingertip.

**Important:** macOS camera permissions must be granted to Python/Jupyter.

## Architecture

### Core Components

1. **MediaPipe Hands Pipeline** (cell 4)
   - Configured with `max_num_hands=1`, `model_complexity=1`
   - Detection/tracking confidence thresholds set to 0.6
   - Uses GPU acceleration via Metal on macOS (see GL context logs)

2. **Smoother Class** (cell 5)
   - Exponential moving average (EMA) filter with alpha=0.75
   - Smooths fingertip coordinates to reduce jitter
   - Stateful: maintains previous x/y position

3. **Tracking Loop** (cell 6)
   - 300-frame capture cycle
   - Mirrors webcam feed (horizontal flip)
   - Tracks landmark #8 (index fingertip) from MediaPipe's 21-point hand model
   - Draws smoothed position and full hand skeleton
   - Uses matplotlib for inline frame display

### Data Flow

```
Webcam → OpenCV capture → BGR→RGB → MediaPipe.process() →
Hand landmarks → Index tip (ID=8) → Smoother → Draw overlay → Display
```

### Key Variables

- `TIP_ID = 8`: MediaPipe landmark index for index fingertip
- `smoother.alpha`: Controls smoothing aggressiveness (higher = more smoothing)
- `mp_hands.Hands()`: Main detection/tracking object
- `result.multi_hand_landmarks[0]`: First detected hand's 21 landmarks

## Development Notes

- **Notebook-based workflow**: All code is in `finger_tracking.ipynb`. There are no separate .py modules.
- **Camera resource management**: Always run cleanup cell (`cap.release()`, `hands.close()`) to avoid camera lock
- **Performance**: The loop runs at webcam framerate. For production, consider implementing FPS limiting or asynchronous processing.
- **MediaPipe version**: Uses legacy solutions API (mp.solutions.hands). Newer versions may use different APIs.
- **Coordinate system**: MediaPipe returns normalized coords (0-1), converted to pixel space using frame dimensions

## Common Issues

- **"Could not open webcam"**: Check camera permissions in System Settings → Privacy & Security → Camera
- **GL context warnings**: Normal on macOS - MediaPipe uses Metal acceleration
- **"Feedback manager" warnings**: Expected with this MediaPipe model, can be ignored
- **Camera lock**: Run the cleanup cell if camera appears in use after stopping the loop
