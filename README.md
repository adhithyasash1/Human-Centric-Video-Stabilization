# human-centric video stabilization

a sophisticated approach to video stabilization that keeps the human subject centered while allowing the background to drift naturally, creating smooth, professional-looking footage.

## what it does

instead of traditional stabilization that fixes the entire frame, this tool intelligently tracks human subjects and keeps them centered while the background moves organically. the result feels more natural and focuses attention on what matters most—the person in your video.

### key features

- **intelligent human tracking** using yolov8 pose detection
- **advanced background removal** with rmbg-2.0 model
- **kalman filtering** for smooth motion prediction
- **soft alpha blending** for seamless foreground/background composition
- **optional background blur** for cinematic depth of field
- **pose keypoint visualization** for debugging and analysis

## how it works

the magic happens in several stages:

1. **pose detection** - identifies key body points (shoulders, hips) to find the person's center
2. **background separation** - uses ai to cleanly separate subject from background
3. **motion smoothing** - applies kalman filtering to predict and smooth movements
4. **intelligent warping** - moves background opposite to subject motion while keeping subject fixed
5. **seamless blending** - combines foreground and background with soft alpha compositing

## getting started

### requirements

```bash
pip install torch>=2.0.0 torchvision>=0.15.0 transformers==4.44.2
pip install ultralytics>=8.0.0 opencv-python>=4.8.0 numpy>=1.24.0
pip install pillow>=9.0.0 accelerate>=0.20.0 huggingface-hub scipy>=1.10.0
```

### setup

you'll need a hugging face token for the background removal model:

```python
from huggingface_hub import login
login(token="your-hf-token-here")
```

### basic usage

```bash
python src/run.py --input your_video.mp4
```

this creates two outputs:
- `stabilized.mp4` - the stabilized video
- `comparison.mp4` - side-by-side original vs stabilized

### advanced options

```bash
python src/run.py \
  --input video.mp4 \
  --output-stab smooth_video.mp4 \
  --output-comp comparison.mp4 \
  --target-x 960 --target-y 540 \
  --device cuda \
  --blur-bg \
  --visualize
```

**parameters:**
- `--target-x/y` - where to center the subject (default: frame center)
- `--device` - use 'cpu', 'cuda', or 'auto' for automatic detection
- `--blur-bg` - adds gaussian blur to background for cinematic effect
- `--visualize` - shows keypoints and target point for debugging
- `--poses` - saves pose data as json for analysis

## technical approach

### pose tracking
uses yolov8's pose estimation to identify 17 body keypoints, focusing on shoulders and hips for stable center-of-mass calculation. only high-confidence keypoints (>0.5) are used for robustness.

### background separation
leverages briaai's rmbg-2.0 model, a state-of-the-art transformer for background removal. produces soft alpha mattes rather than hard cutouts for natural blending.

### motion smoothing
implements a 4-state kalman filter tracking position and velocity in 2d. tunable process and measurement noise parameters allow balancing responsiveness vs smoothness.

### rendering pipeline
applies inverse transforms to background (letting it drift) while forward-transforming the subject (keeping them centered). uses lanczos resampling for quality and replicate border mode to avoid black edges.

## output structure

```
your-project/
├── stabilized.mp4      # main stabilized output
├── comparison.mp4      # side-by-side comparison
└── poses.json         # frame-by-frame pose data
```

the pose data includes keypoint coordinates, confidence scores, and applied transforms for each frame—useful for analysis or further processing.

## performance notes

- **gpu acceleration** - automatically uses cuda if available, significantly faster for background removal
- **memory efficient** - processes frame-by-frame without loading entire video
- **real-time capable** - can achieve near real-time performance on modern hardware

typical performance: ~10-15 fps on cpu, ~25-30 fps on gpu (depends on resolution and hardware).

## use cases

perfect for:
- **interview footage** - keep speakers centered while handheld camera moves
- **dance videos** - maintain focus on performer while background stays dynamic  
- **sports analysis** - track athletes while preserving spatial context
- **content creation** - add professional polish to handheld footage
- **social media** - create smooth, engaging videos from shaky originals

## limitations

- works best with single person in frame (multi-person support planned)
- requires clear view of shoulders/hips for reliable tracking
- processing time depends on video length and hardware capabilities
- background model requires internet connection for first download

## contributing

this project welcomes contributions! areas for improvement:
- multi-person tracking and selection
- real-time processing optimizations  
- additional pose estimation models
- custom background replacement
- batch processing for multiple videos

---

*built with love by R Sashi Adhithya for creators who want their videos to look as smooth as their moves.*
