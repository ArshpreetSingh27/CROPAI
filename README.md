# CropVision AI

CropVision AI is a FastAPI web app for classifying crop leaf images. Its interface displays a disease prediction, class information, and a Grad-CAM heatmap. The 17 classes cover tomato, potato, and corn, including healthy leaves. This is a project prototype; check predictions and treatment suggestions independently before field use.

## What is in this repository

```text
app.py                    FastAPI routes and web app entry point
inference.py              EfficientNet-B3 inference and Grad-CAM
class_symptoms.json       Labels, symptoms, and treatments for 17 classes
class_weights.json        Training class weights
static/                   HTML, CSS, and browser JavaScript
scripts/                  Training and diagnostic programs
notebooks/                Colab training notebooks
examples/                 Saved example heatmaps
requirements.txt          Web app dependencies
requirements-training.txt Optional training and analysis dependencies
VIVA_GUIDE.md             Project study notes
```

The app files are at the repository root. Run the commands below from that directory.

## Setup

```bash
git clone https://github.com/ArshpreetSingh27/CROPAI.git
cd CROPAI
python -m venv .venv
```

Activate the environment in PowerShell with `.venv\Scripts\Activate.ps1`, or on macOS/Linux with `source .venv/bin/activate`. Then install dependencies:

```bash
python -m pip install -r requirements.txt
```

### Model checkpoint

Place a trained EfficientNet-B3 checkpoint named `best_model_vision_autosave.pth` in the repository root. It must match the 17 labels in `class_symptoms.json`.

**The checkpoint is not included.** The interface can load without it, but `/predict` returns HTTP 503 until a valid checkpoint is available. Model files and datasets are excluded by `.gitignore`.

### Optional API keys

Create a root-level `.env` file to enable external services:

```dotenv
GROQ_API_KEY=your_groq_key
SARVAM_API_KEY=your_sarvam_key
GEMINI_API_KEY=your_gemini_key
```

Groq provides optional text advice; Sarvam provides optional speech, with a browser speech fallback. The current `/predict` implementation may use Gemini to override the model label and display a generated confidence value when a key is configured. That value is **not** a calibrated model probability. Keep `.env` private.

For the optional dataset-image fast path in `app.py`, set `CROP_TEST_DATA_DIR` to the local test-image directory. Leave it unset for normal predictions.

## Run

```bash
uvicorn app:app --host 127.0.0.1 --port 8090
```

Open [http://127.0.0.1:8090](http://127.0.0.1:8090). API documentation is at [http://127.0.0.1:8090/docs](http://127.0.0.1:8090/docs).

| Route | Method | Purpose |
| --- | --- | --- |
| `/` | GET | Serve the interface |
| `/predict` | POST | Predict from multipart image field `file` |
| `/ask_ai` | POST | Request optional text advice |
| `/tts` | POST | Request optional speech audio |

Example:

```bash
curl -X POST -F "file=@path/to/leaf.jpg" http://127.0.0.1:8090/predict
```

## Training and diagnostics

For standalone scripts, install the additional dependencies and run modules from the repository root:

```bash
python -m pip install -r requirements-training.txt
python -m scripts.check_sorting
python -m scripts.model_training_vision_only
```

The scripts expect local ImageFolder-style datasets such as `dataset_final/train`, `dataset_final/val`, and `dataset_final/test`. Several diagnostics also expect `best_model_vision.pth`; provide that checkpoint before using them. The notebooks in `notebooks/` target Google Colab and have Drive-specific paths that need adjustment for your dataset.

## Completeness and limitations

- Source code, frontend assets, training scripts, notebooks, class metadata, and example heatmaps are present.
- Trained checkpoints and datasets are absent, so end-to-end prediction accuracy cannot be reproduced from this repository alone.
- Optional external API features require your own keys.
- The generated Grad-CAM image is written to a shared `static/gradcam.jpg` path; simultaneous predictions can overwrite each other's heatmap.
- Some diagnostic scripts contain machine-specific paths or assume local datasets. Adjust those before running them on another computer.
