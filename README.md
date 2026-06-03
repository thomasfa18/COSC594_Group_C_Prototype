# COSC594 Poultry Disease Detection Prototype

This repository contains the proof-of-concept experiment developed to support the COSC594 group C project on early AI-assisted disease detection in poultry.

The broader project is a PRISMA-style literature review examining non-invasive AI and machine learning approaches for poultry disease detection. The prototype component is intentionally small and is designed to validate one practical direction identified in the literature: image-based classification of poultry disease using public fecal image datasets.

This prototype is a research demonstration showing how several image classification models can be trained and compared on public poultry fecal image datasets.

## Prototype purpose

The prototype compares deep learning image classification models on public poultry fecal image datasets containing disease/health classes such as:

- Healthy
- Coccidiosis
- Newcastle disease / NCD
- Salmonella

The experiment reports:

- overall accuracy
- per-class precision
- per-class recall / sensitivity
- per-class specificity
- per-class F1-score
- confusion matrices
- comparison against literature baseline metrics where available

## Repository structure

Recommended repository layout:

```text
.
├── Group_C_Image_model_prototype.ipynb
├── README.md
├── requirements.txt
└── data/
    ├── poultry_fecal_dataset/
    │   ├── machuve-zenodo/
    │   │   ├── cocci/
    │   │   ├── healthy/
    │   │   ├── ncd/
    │   │   └── salmo/
    │   └── allandclive/
    │       └── chicken-disease-1/
    │           ├── train_data.csv
    │           └── Train/
    └── poultry_experiment_outputs/
        ├── machuve-zenodo/
        └── allandclive/
```

## Main notebook

The main notebook is:

```text
Group_C_Image_model_prototype.ipynb
```

The notebook performs the following steps:

1. Selects the dataset using `DATASET_NAME`
2. Loads image paths and labels
3. Applies a stratified train/validation/test split
4. Builds TensorFlow datasets
5. Trains selected models
6. Optionally fine-tunes transfer-learning models (not yet implemented)
7. Evaluates each model
8. Saves metrics, charts, and model outputs
9. Generates visual comparisons from in-memory results or saved CSVs

## Supported datasets

The prototype currently supports two public dataset configurations.

### 1. Machuve / Zenodo poultry fecal dataset

This is avialble from [zenodo](https://zenodo.org/records/5801834)

Set:

```python
DATASET_NAME = "machuve-zenodo"
IMG_DIR = Path("./data/poultry_fecal_dataset/machuve-zenodo")
```

Expected folder structure:

```text
data/poultry_fecal_dataset/machuve-zenodo/
  cocci/
  healthy/
  ncd/
  salmo/
```

This dataset is expected to be organised as one folder per class.

### 2. Kaggle AllanClive chicken disease dataset

This dataset is avialble from [kaggle](https://www.kaggle.com/datasets/allandclive/chicken-disease-1/data?suggestionBundleId=1679)

Set:

```python
DATASET_NAME = "allandclive"
IMG_DIR = Path("./data/poultry_fecal_dataset/allandclive")
```

Expected folder structure:

```text
data/poultry_fecal_dataset/allandclive/chicken-disease-1/
  train_data.csv
  Train/
```

The CSV file must contain:

```text
images,label
```

where:

- `images` is the filename
- `label` is one of:
  - `Salmonella`
  - `Coccidiosis`
  - `New Castle Disease`
  - `Healthy`

The notebook maps these dataset labels to canonical report labels.

## Dataset class mapping

The notebook standardises disease class names using:

```python
CLASS_ALIASES = {
    "Healthy": ["Healthy", "healthy", "Normal", "normal"],
    "Coccidiosis": ["Cocci", "Coccidiosis", "cocci", "coccidiosis", "Crcocci"],
    "Newcastle disease (NCD)": ["NCD", "ncd", "Newcastle", "Newcastle disease", "New Castle Disease", "newcastle", "newcastle disease", "new castle disease"],
    "Salmonella": ["Salmo", "salmo", "Salmonella", "salmonella"],
}
```

This is used so results from different public datasets can be compared using consistent labels.

## Models included

The notebook can train and compare several models, depending on what is selected in `MODELS_TO_RUN`.

Supported models include:

```python
MODELS_TO_RUN = [
    "Small_CNN",
    "MobileNetV2",
    "DenseNet121",
    "EfficientNetB0",
    "Xception",
    "ResNet50",
    "VGG19",
    "ViT_Tiny",
]
```

Recommended starting models:

```python
MODELS_TO_RUN = [
    "MobileNetV2",
    "DenseNet121",
    "Xception",
]
```

`VGG19` and `ViT_Tiny` are optional comparison models. They may take longer to train depending on the available hardware.

## Fine tuning

Fine tuning is not fully implemented or tested but is intended to be controlled by:

```python
FINE_TUNE = False
```

To enable fine-tuning:

```python
FINE_TUNE = True
```

Fine-tuning settings:

```python
FINE_TUNE_EPOCHS = 5
FINE_TUNE_LEARNING_RATE = 1e-5
FINE_TUNE_LAST_N_LAYERS = 30
```

Fine-tuning only applies to pretrained transfer-learning models. It does not apply to the small custom CNN or the lightweight custom ViT implementation unless separately implemented.

## Environment requirements

Recommended environment:

- Python 3.10+
- Jupyter Notebook or Google Colab
- TensorFlow / Keras
- pandas
- numpy
- scikit-learn
- matplotlib

Example installation:

```bash
pip install tensorflow pandas numpy scikit-learn matplotlib
```

For GPU acceleration, use a TensorFlow GPU-compatible environment. The notebook can run on CPU, but training will be slower.

## Running the prototype

1. Clone or download the repository.

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Download one of the supported public datasets.

4. Place the dataset in the expected folder structure under:

```text
data/poultry_fecal_dataset/
```

5. Open:

```text
Group_C_Image_model_prototype.ipynb
```

6. Set the dataset name:

```python
DATASET_NAME = "machuve-zenodo"
```

or:

```python
DATASET_NAME = "allandclive"
```

7. Select models:

```python
MODELS_TO_RUN = [
    "MobileNetV2",
    "DenseNet121",
    "Xception",
]
```

8. Run all notebook cells.

9. Review generated outputs in:

```text
data/poultry_experiment_outputs/<DATASET_NAME>_<date>_<time>/
```

## Output files

The notebook saves results suitable for the final report, including:

```text
prototype_per_class_metrics.csv
prototype_overall_model_metrics.csv
literature_overall_model_baselines.csv
literature_disease_level_baselines.csv
prototype_vs_literature_overall_accuracy.csv
prototype_vs_literature_disease_level_metrics.csv
```

TensorBoard logs are saved per model under:

```text
data/poultry_experiment_outputs/<DATASET_NAME>_<date>_<time>/logs/
```

To view logs:

```bash
tensorboard --logdir data/poultry_experiment_outputs/<DATASET_NAME>_<date>_<time>/logs
```

## Model saving

Model saving is optional and controlled by the `SAVE_MODEL` flag in the notebook globals.
When enabled, the best weights per model (by validation loss) are saved under:

```text
data/poultry_experiment_outputs/<DATASET_NAME>_<date>_<time>/models/
```

Example:

```text
data/poultry_experiment_outputs/allandclive_20260602_095856/models/MobileNetV2_best.weights.h5
```

## Visualizations from saved CSVs

The visualization cell in Section 7 can load saved CSV outputs directly by setting:

```python
CSV_OUTPUT_DIR = "./data/poultry_experiment_outputs/<DATASET_NAME>_<date>_<time>"
```

This enables plotting comparisons without re-running training.

## Deployment instructions for client review

This prototype is deployed as a notebook-based research artefact rather than a production web application.

To review or rerun the prototype:

1. Open the notebook in Jupyter or Google Colab.
2. Ensure the dataset is placed in the expected folder structure.
3. Select the dataset and model list in the globals section.
4. Run the notebook from top to bottom.
5. Review the generated metrics tables, charts, and saved model files.

The prototype demonstrates the feasibility of training and comparing image-based disease classifiers using public datasets. It should be treated as a proof of concept for future work, not as a validated clinical or commercial diagnostic tool.

## Handover notes for future project groups

A future group can extend this work by:

1. Re-running the notebook on the full public datasets rather than small subsets.
2. Adding additional public image datasets identified in the literature review.
3. Testing more models from the reviewed papers, such as EfficientNet, ResNet, YOLO classification variants, or larger Vision Transformers.
4. Developing fine tuning and performing hyperparameter tuning.
5. Adding stronger data augmentation.
6. Comparing cross-dataset generalisation.
7. Developing a simple Streamlit or web interface for image upload and prediction.
8. Validating performance with domain experts.
9. Extending the prototype to audio datasets by converting audio clips into spectrogram or waveform image slices.
10. Linking experiment results back to the PRISMA literature table and dataset/model mapping.

## Known limitations

- The prototype uses public datasets and may not represent commercial farm conditions.
- Dataset class distributions may be imbalanced, and no corrective action has been implemented to account for class imbalances.
- Results may vary depending on random seed, train/validation/test split, hardware, and dataset subset.
- Some literature baselines report only overall accuracy, which limits direct disease-specific comparison.
- The model has not been validated against unseen commercial farm data.