# No DOM, No Cry: Human-Inspired CAPTCHA Solving via YOLO Reflexes and VLM Teachers


  Traditional CAPTCHA solvers require Document Object Model (DOM) access to detect UI elements, making them incompatible with fully visual GUI agents. While Vision-Language Models (VLMs) offer superior reasoning, they are too expensive and slow for real-time CAPTCHA solutions. We introduce a DOM-free hybrid architecture inspired by how humans develop reflexes for familiar patterns to avoid the latency of decision process. In this architecture, YOLO model handles frequent and routine tasks like human reflexes while VLM reasoning intervenes only for novel or hard problems like a slow cognitive process. Our system utilizes a fine-tuned YOLOv8 backbone for real-time UI localization, dynamically routing challenges either to specialized YOLO classifiers for rapid pattern recognition or to an open-weight Qwen-7B VLM for semantic reasoning on novel or hard-to-distinguish categories. Combining these complementary strengths, our hybrid method achieves 86.85% macro-averaged recall, +24.2% over YOLO-only baselines. This macro-averaged performance is critical as CAPTCHA designers can prioritize the distributions of classes where known YOLO-only solvers fail. Beyond current performance gains, this work establishes a paradigm shift in efficient automation where VLM acts as a "teacher" to programmatically label novel challenges encountered during operation. We propose that as the agent accumulates sufficient samples for new categories, it can autonomously train and deploy specialized YOLO-based solvers as reflexes. Until such data is sufficient, VLM continues to intervene to provide accurate solutions. This approach allows future AI agents to achieve maximum efficiency by distilling expensive visual reasoning into millisecond-scale reflexes, using VLM for the discovery of novel patterns while specialized classifiers execute rapid responses required for CAPTCHA solution.


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