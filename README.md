# No DOM, No Cry: Human-Inspired CAPTCHA Solving via YOLO Reflexes and VLM Teachers

  Vision-language model (VLM) agents can now operate computers on behalf of users, and recent work shows they can solve visual CAPTCHAs without task-specific training. What they lack is the ability to learn from the experience. Humans, by contrast, leverage prior experience naturally. The first CAPTCHA requires deliberate effort to read the instructions, examine the grid, and reason about each tile, but after a few encounters the entire process collapses into reflex. VLM agents never make this transition. Every CAPTCHA is treated as a novel reasoning problem, even after hundreds of encounters with visually identical challenges at significant latency and cost (e.g., 21.8s and \$2.40 per 100 puzzles). Fast specialized classifiers avoid this cost but fail entirely on unseen categories, and neither approach improves over time with exposure.

This paper is not about building yet another CAPTCHA solver --- it is about what CAPTCHA solving looks like in the agentic era, where solvers can reason, learn, and adapt. We present a DOM-free hybrid architecture that mirrors human dual-process cognition: a fine-tuned YOLOv8 provides fast reflexes from screenshots while an open-weight Qwen-7B VLM supplies deliberate reasoning, with a confidence-based cascade dispatching 70\% of challenges at reflex speed (4ms) and invoking VLM (225ms) only when needed. The hybrid achieves 85.4\% overall and 84.2\% macro accuracy across 16 classes (+10.0pp and +24.2pp over a YOLO-only baseline). By design, every challenge the VLM solves becomes a labeled training example for YOLO, enabling the system to autonomously expand its capabilities with each encounter. We demonstrate that previously unsupported object classes can be learned from only a few VLM-labeled puzzle encounters, and that PGD adversarial attacks reducing YOLO to 0\% accuracy are autonomously recovered through VLM-guided retraining. Counterintuitively, under iterative grey-box escalation, VLM's imperfect labels cause the solver to diverge from the defender's surrogate, yielding a 53pp robustness advantage that clean labels cannot achieve --- a property typically seen as a limitation becomes an implicit defense. These results challenge a core assumption of visual CAPTCHA defenses: that bot failures remain persistent. When a solver reasons, adapts, and hardens with each encounter, temporary failures become permanent reflexes, undermining the security model that visual CAPTCHAs depend on.

## Key Strengths

**State-of-the-Art Performance**

- Handles all three reCAPTCHA v2 puzzle types with rare class support (classification, segmentation, dynamic)
- 86.85% macro-averaged recall (+24.2% over YOLO-only baselines)

**Cross-Platform Compatible**

- Works on Linux, macOS and Windows
- No platform-specific dependencies or modifications required

**Cross-Browser Compatible**

- **Works with ANY browser** (Chrome, Firefox, Edge, Safari, etc.)
- **No DOM access required** - Screenshot-based interaction
- **No browser automation frameworks** No automation framework or browser needed (Selenium, Playwright, Puppeteer)
- **Evades WebDriver detection** - No DOM manipulation signatures

**Open-Weight Models**

- Uses Qwen-7B-VL (open-weight) instead of proprietary APIs
- No API costs for inference
- Fully reproducible on-premise

## System Requirements

### Hardware Requirements

- **Recommended (Hybrid mode)**: 32GB RAM, 16GB+ GPU VRAM (NVIDIA RTX 3090 or equivalent), 100GB disk space

### Software Requirements

- **Operating Systems**: Linux, macOS or Windows (tested on all three)
- **Python**: 3.11
- **Browsers**: Any modern browser with screenshot capability
- **Key Libraries**: PyTorch, Transformers, Ultralytics (YOLOv8), PyAutoGUI, MSS, OpenCV
- **Display Resolution**: The system performs optimally at 1920x1080 resolution. For other display resolutions, browser zoom adjustment may be necessary to ensure reliable UI element localization

### Platform-Specific Requirements

**Linux**:

```bash
sudo apt install gnome-screenshot
```

**macOS**:

- No additional requirements (uses built-in `screencapture`)

**Windows**:

- No additional requirements (uses Windows API)

## Installation

1. Ensure Python 3.11 is installed on your system
2. Create a virtual environment:

```bash
# Linux/Mac
python3.11 -m venv venv
source venv/bin/activate

# Windows
python -m venv venv
venv\Scripts\activate
```

3. Install dependencies:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

4. Required model files:
   - `detection_model.pt` (39 MB) - Detection model
   - `classification_model.pt` (2.9 MB) - Classification model
   - `yolov8x-seg.pt` (140 MB) - Segmentation model (auto-download due to size limitation)

### Command-line Arguments

- `-v, --verbose`: Enable verbose output for debugging
- `--no-mouse`: Disable mouse movement animations
- `--backend {yolo,llm,hybrid}`: Choose the solver backend
- `yolo`: Full YOLO-based detection and classification (default)
- `llm`: YOLO detection with LLM-based classification
- `hybrid`: Best backend per object type

### Examples

```bash
# Run with YOLO only backend
python main.py

# Run with LLM only backend
python main.py --backend llm

# Run with hybrid mode
python main.py --backend hybrid
```

## Configuration

- `config.json`: Contains the API URL for LLM backend communication (if using LLM/hybrid modes)
- `config.py`: Global configuration for mouse movement behavior

## Architecture Overview

Our system implements a human-inspired dual-process approach:

- **YOLO (Fast Reflexes)**: Handles frequent, routine patterns with millisecond inference
- **VLM (Slow Reasoning)**: Intervenes for novel or semantically complex challenges

This hybrid architecture achieves superior performance by combining:

1. Fine-tuned YOLOv8 for real-time UI element detection
2. Specialized YOLO classifiers for common object classes
3. Open-weight Qwen-7B-VL for zero-shot reasoning on rare categories
4. Finite-state machine controller for robust puzzle solving

## Project Structure

```
.
├── main.py                          # Main entry point
├── captcha_fsm.py                   # State machine for CAPTCHA solving
├── unified_captcha_processor.py     # Core processor with multiple backends
├── captcha_solver_bridge.py         # Interface bridge
├── universal_object_analyzer.py     # LLM analysis utility
├── logger.py                        # Logging utility
├── config.py                        # Global configuration
├── config.json                      # API configuration
├── requirements.txt                 # Python dependencies
├── detection_model.pt               # Detection model checkpoint
├── classification_model.pt          # Classification model
├── yolov8x-seg.pt                   # Segmentation model (auto-download model if missing)
├── captcha_detector_trainer/        # Detection dataset generation
├── complete_classification/         # Classification experiments
├── segmentation_metrics/            # Segmentation experiments (auto-download models if missing)
└── llm_backend/                     # VLM server backend
```

## How It Works

1. **Detection**: Continuously monitors the screen for reCAPTCHA checkboxes using YOLOv8
2. **UI Localization**: Detects CAPTCHA area, grid cells, verify/reload buttons via screenshot analysis
3. **Puzzle Analysis**: OCR extracts target object; determines puzzle type (classification vs segmentation)
4. **Solving**: Routes to YOLO (fast) or VLM (accurate on rare classes) based on target object
5. **Verification**: Checks success and handles dynamic puzzles (tiles refresh after clicks)

## Ethical Considerations

This research is conducted for academic purposes to advance understanding of vision-language model capabilities and GUI automation systems. As with any security research, our techniques could potentially be misused for unauthorized automation. We do not endorse such applications and emphasize that circumventing security measures without authorization is unethical and often illegal. However, advancing scientific understanding of AI capabilities requires studying both strengths and limitations of deployed systems.
