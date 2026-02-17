# Leaf Classifier Model

> **Hinweis: Dieses Modell befindet sich in einer Testphase. Die Ergebnisse sind nicht garantiert korrekt. Jede Nutzerin und jeder Nutzer ist selbst dafür verantwortlich, die Ergebnisse zu überprüfen. Keine Haftung für fehlerhafte Bestimmungen.**

CoreML model for classifying tree leaves and fruits. Covers 200+ tree species commonly found in Vienna.

## Download

Go to [Releases](../../releases) and download the latest `LeafClassifier.mlpackage.zip`.

## Usage

### In the Baumkataster Wien app

The app downloads the model automatically on first use from a CDN. No manual setup needed.

### Manual usage

1. Download `LeafClassifier.mlpackage.zip` from the latest release
2. Unzip and add it to your Xcode project
3. Load it with CoreML:

```swift
let config = MLModelConfiguration()
config.computeUnits = .cpuAndGPU
let model = try MLModel(contentsOf: modelURL, configuration: config)
let visionModel = try VNCoreMLModel(for: model)
```

## Model details

| Property | Value |
|---|---|
| Framework | CoreML + Vision |
| Input | 224x224 RGB image |
| Output | Classification probabilities |
| Species | 200+ |
| Size | ~380 MB |
| Training data | iNaturalist leaf images |

## Disclaimer

This model is provided **for testing and educational purposes only**. Classification results may be inaccurate. Users are solely responsible for verifying any species identification. No liability is accepted for incorrect classifications.

## Files

- `LeafClassifier.mlpackage/` - Full CoreML model package
  - `Data/com.apple.CoreML/model.mlmodel` - Model specification
  - `Data/com.apple.CoreML/weights/weight.bin` - Model weights (~379 MB)
  - `Manifest.json` - Package manifest

## License

All rights reserved.
