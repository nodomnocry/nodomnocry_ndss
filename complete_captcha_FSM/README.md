# No DOM, No Cry: Human-Inspired CAPTCHA Solving via YOLO Reflexes and VLM Teachers

Vision-language model (VLM) agents can now operate computers on behalf of users, and recent work shows they can solve visual CAPTCHAs without task-specific training. What they lack is the ability to learn from the experience. Humans, by contrast, leverage prior experience naturally. The first CAPTCHA requires deliberate effort to read the instructions, examine the grid, and reason about each tile, but after a few encounters the entire process collapses into reflex. VLM agents never make this transition. Every CAPTCHA is treated as a novel reasoning problem, even after hundreds of encounters with visually identical challenges at significant latency and cost (e.g., 21.8s and $2.40 per 100 puzzles). Fast specialized classifiers avoid this cost but fail entirely on unseen categories, and neither approach improves over time with exposure.

In this paper, we present a DOM-free hybrid architecture that resolves both limitations by mirroring human dual-process cognition, distilling expensive VLM reasoning into fast YOLO reflexes without relying on browser automation frameworks. A fine-tuned YOLOv8 localizes UI elements directly from screenshots, and a class-wise router dispatches each challenge to YOLO or an open-weight Qwen-7B VLM. The hybrid achieves 85.4% overall accuracy and 84.2% macro recall across 16 classes (+10.0pp overall and +24.2pp macro recall over a YOLO-only baseline), surpassing prior DOM-dependent and VLM-only solvers while using VLM for only 30% of images. By design, every challenge the VLM solves becomes a labeled training example for YOLO, creating a built-in mechanism for continuous improvement. We demonstrate this through two adaptive scenarios: learning unsupported object classes from only a few VLM-labeled puzzle encounters, and recovering from PGD adversarial attacks that reduce YOLO to 0% accuracy. Counterintuitively, under iterative grey-box escalation where both solver and defender retrain each round, VLM's imperfect labels cause the solver to diverge from the defender's surrogate model, yielding a 53pp robustness advantage that clean labels cannot achieve. A property typically seen as a limitation, noisy teacher labels, becomes an implicit defense under adversarial pressure, progressively transforming expensive reasoning into permanent, hardened reflexes that strengthen with each encounter.


## Key Strengths

**State-of-the-Art Performance**
- Handles all three reCAPTCHA v2 puzzle types with rare class support (classification, segmentation, dynamic)
- 85.4% overall accuracy (+10.0pp over YOLO-only baseline)
- 84.2% macro-averaged accuracy (+24.2pp over YOLO-only baseline)
- VLM invoked for only 30% of images via confidence-based cascade routing

**Adaptive Learning**
- Teacher-student distillation: VLM labels unsupported classes, YOLO learns them as reflexes
- Taxi: 96% recall from ~2 puzzle encounters, Tractor: 88% from ~3 puzzles
- Autonomous recovery from adversarial attacks via VLM-guided retraining

**Cross-Platform Compatible**
- Works on Linux, macOS and Windows
- No platform-specific dependencies or modifications required

**Cross-Browser Compatible**
- **Works with ANY browser** (Chrome, Firefox, Edge, Safari, etc.)
- **No DOM access required** - Screenshot-based interaction
- **No browser automation frameworks** - No Selenium, Playwright, or Puppeteer needed
- **Evades WebDriver detection** - No DOM manipulation signatures

**Open-Weight Models**
- Uses Qwen-7B-VL (open-weight) instead of proprietary APIs
- No API costs for inference
- Fully reproducible on-premise

## System Requirements

### Hardware Requirements
- **Minimum**: 16GB RAM, 20GB disk space. YOLO-only solver can run on CPU without GPU, though inference will be significantly slower
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
   - `detection_model.pt` (39 MB) - UI element detection model
   - `classification_model.pt` (2.9 MB) - Classification model
   - `yolov8x-seg.pt` (140 MB) - Segmentation model (auto-download on first run)

### Command-line Arguments

- `-v, --verbose`: Enable verbose output for debugging
- `--no-mouse`: Disable mouse movement animations
- `--backend {yolo,llm,hybrid}`: Choose the solver backend
  - `yolo`: Full YOLO-based detection and classification (default, fastest)
  - `llm`: YOLO UI detection with VLM-based classification (requires VLM backend)
  - `hybrid`: Confidence-based routing between YOLO and VLM (requires VLM backend)

### Examples

```bash
# Run with YOLO only backend (default)
python main.py

# Run with VLM backend
python main.py --backend llm

# Run with hybrid mode
python main.py --backend hybrid
```

## Configuration

- `config.json`: Contains the API URL for VLM backend communication (if using llm/hybrid modes)
- `config.py`: Global configuration for mouse movement behavior

## Architecture Overview

Our system implements a human-inspired dual-process approach:
- **YOLO (Fast Reflexes)**: Handles frequent, routine patterns with millisecond inference
- **VLM (Slow Reasoning)**: Intervenes for novel or semantically complex challenges

This hybrid architecture achieves superior performance by combining:
1. Fine-tuned YOLOv8 for real-time UI element detection
2. Specialized YOLO classifiers for common object classes
3. Open-weight Qwen-7B-VL for zero-shot reasoning on rare categories
4. Confidence-based cascade routing (threshold = 0.70)
5. Finite-state machine controller for robust puzzle solving

## Project Structure

```
.
├── complete_captcha_FSM/                # CAPTCHA solver application
│   ├── main.py                          # Main entry point
│   ├── captcha_fsm.py                   # State machine for CAPTCHA solving
│   ├── unified_captcha_processor.py     # Core processor with multiple backends
│   ├── config.py                        # Global configuration
│   ├── config.json                      # API configuration
│   ├── detection_model.pt               # UI element detection model
│   ├── classification_model.pt          # Classification model
│   └── yolov8x-seg.pt                   # Segmentation model (auto-download)
├── experiments/                         # Evaluation and experiment notebooks
│   ├── classification_metrics/          # Classification evaluation
│   │   ├── classification_metrics.ipynb # Per-class metrics and confusion matrices
│   │   └── cascade_routing_analysis.ipynb # Cascade threshold analysis
│   ├── segmentation_metrics/            # Segmentation evaluation
│   ├── distillation/                    # Experience Replay fine-tuning
│   │   ├── taxi_finetune_er.ipynb
│   │   ├── tractor_finetune_er.ipynb
│   │   └── boat_finetune_er.ipynb
│   └── adversarial/                     # Adversarial robustness experiments
│       ├── 01_generate_pgd_attacks.ipynb
│       ├── 02_vlm_inference_on_adversarial.ipynb
│       ├── 03_sample_efficiency.ipynb
│       └── 04_greybox_escalation.ipynb
├── llm_backend/                         # VLM server backend
├── captcha_detector_trainer/            # UI detection dataset generation
└── requirements.txt                     # Python dependencies
```

## How It Works

1. **Detection**: Continuously monitors the screen for reCAPTCHA checkboxes using YOLOv8
2. **UI Localization**: Detects CAPTCHA area, grid cells, verify/reload buttons via screenshot analysis
3. **Puzzle Analysis**: OCR extracts target object; determines puzzle type (classification vs segmentation)
4. **Routing**: For unsupported classes, routes directly to VLM. For supported classes, YOLO classifies first; if confidence < 0.70, falls back to VLM
5. **Verification**: Checks success and handles dynamic puzzles (tiles refresh after clicks)

## Experiments

### Teacher-Student Distillation
VLM predictions serve as labeled training data to fine-tune YOLO on unsupported classes via Experience Replay. Results (10 seeds per N value):
- **Taxi**: 96% recall at N=10 (~2 puzzle encounters)
- **Tractor**: 88% recall at N=15 (~3 puzzle encounters)
- **Boat**: 48% recall at N=6 (~1 puzzle encounter)

### Adversarial Robustness
PGD attacks (epsilon=4/255, 10 steps) reduce YOLO accuracy from 83.1% to 0%, but VLM accuracy drops only 3.7pp (78.7% to 75.0%). The solver autonomously recovers by retraining on VLM-labeled adversarial samples, reaching 65% adversarial accuracy from ~2,000 samples.

### Grey-Box Escalation
Over 6 rounds of iterative attacks, VLM label noise causes the solver to diverge from the defender's surrogate model, yielding a 53pp robustness advantage. The noisy labels function as an implicit defense against grey-box attacks.

## Ethical Considerations

This research is conducted for academic purposes to advance understanding of vision-language model capabilities and GUI automation systems. As with any security research, our techniques could potentially be misused for unauthorized automation. We do not endorse such applications and emphasize that circumventing security measures without authorization is unethical and often illegal. However, advancing scientific understanding of AI capabilities requires studying both strengths and limitations of deployed systems.
