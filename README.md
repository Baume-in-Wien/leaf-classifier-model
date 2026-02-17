# 🍃 Leaf Classifier Model

> **⚠️ Hinweis / Disclaimer:** Dieses Modell befindet sich in einer Testphase. Die Ergebnisse sind nicht garantiert korrekt. Jede:r Nutzer:in ist selbst dafür verantwortlich, die Ergebnisse zu überprüfen. Keine Haftung für fehlerhafte Bestimmungen.
>
> **This model is experimental and provided for testing purposes only. Classification results are not guaranteed to be correct. Every user is solely responsible for verifying the results. No liability is accepted for incorrect identifications.**

A CoreML and PyTorch model for classifying tree leaves and fruits. Covers **200+ tree species** commonly found in Vienna, Austria.

## ⚡ Known Limitations

- The model is trained **only on leaf images**. It does not know what "not a leaf" looks like.
- If you feed it a random image (a face, a car, food, etc.), it will still output a tree species with potentially high confidence. This is a fundamental limitation of the model architecture, not a bug.
- Accuracy varies significantly between species. Common species tend to be recognized more reliably than rare ones.
- The model works best with a **single leaf on a plain background**.

## 📥 Download

Go to [Releases](../../releases) and download:

- **`LeafClassifier.mlpackage.zip`** (~380 MB) — CoreML model for iOS/macOS
- **`model_weights.pth`** (~757 MB) — PyTorch weights (ConvNeXt Large) for all other platforms
- **`class_names.json`** — mapping of class indices to species names

## 🔧 Usage

### In the Baumkataster Wien App

The app downloads the model automatically on first use from a CDN. No manual setup needed.

### CoreML (iOS / macOS)

1. Download `LeafClassifier.mlpackage.zip` from the latest release
2. Unzip and add it to your Xcode project
3. Load with CoreML + Vision:

```swift
let config = MLModelConfiguration()
config.computeUnits = .cpuAndGPU
let model = try MLModel(contentsOf: modelURL, configuration: config)
let visionModel = try VNCoreMLModel(for: model)
```

### PyTorch (all platforms)

1. Download `model_weights.pth` and `class_names.json`
2. Load with PyTorch:

```python
import torch
import json
from torchvision.models import convnext_large

# Load class names
with open("class_names.json") as f:
    class_names = json.load(f)

# Load model
model = convnext_large(weights=None)
model.classifier[2] = torch.nn.Linear(model.classifier[2].in_features, len(class_names))
model.load_state_dict(torch.load("model_weights.pth", map_location="cpu"))
model.eval()

# Inference
from torchvision import transforms
from PIL import Image

transform = transforms.Compose([
    transforms.Resize(384),
    transforms.CenterCrop(384),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
])

image = Image.open("leaf.jpg").convert("RGB")
input_tensor = transform(image).unsqueeze(0)

with torch.no_grad():
    output = model(input_tensor)
    probabilities = torch.softmax(output, dim=1)
    top5 = torch.topk(probabilities, 5)

for i in range(5):
    idx = top5.indices[0][i].item()
    conf = top5.values[0][i].item()
    print(f"{class_names[str(idx)]}: {conf:.1%}")
```

### ONNX (Android)

1. Export with `export_android_model.py` or download from releases
2. Use with ONNX Runtime:

```kotlin
val session = OrtEnvironment.getEnvironment()
    .createSession(modelPath)
val inputTensor = OnnxTensor.createTensor(env, imageBuffer, shape)
val output = session.run(mapOf("image" to inputTensor))
```

## 📊 Model Details

| Property | Value |
|---|---|
| Architecture | ConvNeXt Large (197M parameters) |
| Input | 384×384 RGB image |
| Output | Classification probabilities (366 classes) |
| Species | 200+ tree species |
| Top-1 accuracy | 43.9% |
| Top-5 accuracy | 88.9% |
| Training data | iNaturalist leaf images |
| Training hardware | NVIDIA H100 80GB |
| Precision | BF16 mixed precision |

### Training Details

- 4-phase progressive training (100 epochs total)
- Progressive resizing: 224 → 288 → 384
- Mixup + CutMix augmentation
- Label smoothing
- EMA (decay=0.9999)
- Cosine annealing with warmup
- Layer-wise learning rate decay
- 5-fold test-time augmentation

## 📁 Files

| File | Size | Description |
|---|---|---|
| `LeafClassifier.mlpackage.zip` | ~380 MB | CoreML model for iOS/macOS |
| `model_weights.pth` | ~757 MB | PyTorch weights (ConvNeXt Large) |
| `class_names.json` | ~50 KB | Class index → species name mapping |
| `IMAGE_CREDITS.md` | — | Full credits for all training images |

## 📸 Image Credits / Bildnachweis

The model was trained on **530,675 images** from **376 tree species**, photographed by **50,615 photographers** on [iNaturalist](https://www.inaturalist.org) under Creative Commons licenses.

**Full credits: → [`IMAGE_CREDITS.md`](IMAGE_CREDITS.md)**

### License Breakdown

| License | Images |
|---|---|
| CC BY-NC 4.0 | 420,793 |
| CC BY 4.0 | 65,441 |
| CC0 (Public Domain) | 21,711 |
| CC BY-NC-SA 4.0 | 8,974 |
| CC BY-NC-ND 4.0 | 6,569 |
| CC BY-SA 4.0 | 5,419 |
| CC BY-ND 4.0 | 1,768 |

All images were used in compliance with their respective Creative Commons licenses for training this classification model. Each photographer is individually credited in `IMAGE_CREDITS.md`.

We are grateful to all iNaturalist contributors whose observations made this model possible. 🙏

## ⚖️ Disclaimer

This model is provided **for testing and educational purposes only**. It is trained exclusively on leaf images and has no concept of "not a leaf" — it will classify any input image as some tree species regardless of content. Users are solely responsible for verifying any species identification. No liability is accepted for incorrect classifications.

## License

All rights reserved.
