# Soil Grain-Size Distribution Prediction from Images

A small-data computer-vision and machine-learning project that predicts cumulative soil
grain-size distribution curves from photographs using handcrafted image features and
multi-output Ridge regression.

## Project summary

The workflow was developed in Google Colab using a dataset containing 24 labeled soil
samples. The recorded run processed 127 training photographs and 35 test photographs
grouped into 10 test sites.

Rather than training a deep neural network on a very small labeled dataset, the project
extracts interpretable, handcrafted image statistics and uses a regularized regression model.



## Prediction target

The model predicts cumulative percentages at 11 grain-size support points (millimeters):

`0.002, 0.0063, 0.02, 0.063, 0.2, 0.63, 2, 6.3, 20, 63, 200`

Predicted curves are constrained to:
- lie between 0 and 100;
- be non-decreasing across grain sizes;
- end at 100 at 200 mm.

## Image features

The 44 features extracted per photograph include:
- RGB channel means and standard deviations;
- HSV channel means and standard deviations;
- grayscale percentiles (10th, 25th, 50th, 75th, 90th) and standard deviation;
- Canny edge density;
- Laplacian variance;
- normalized 8-bin RGB histograms for each channel.

Multiple photographs of a labeled sample are aggregated by taking the mean feature vector.

## Model and validation

The selected model is a scikit-learn pipeline:

`StandardScaler() -> Ridge(alpha=10.0)`

Evaluation uses Leave-One-Out Cross-Validation (LOOCV), with a DummyRegressor mean
baseline. Metrics recorded in the project:

| Method | Log-weighted curve-error proxy | MAE |
|---|---:|---:|
| Dummy mean baseline | 17.821 | 16.343 |
| Ridge (alpha=10), feature averaging | **7.633** | **7.057** |
| Ridge (alpha=10), prediction averaging | 8.145 | 7.523 |

Lower values are better. The curve-error metric is the normalized area between cumulative
curves on the logarithmic grain-size axis. It is a proxy metric and should not be described
as the official competition score unless independently confirmed.

## Run in Google Colab

1. Open `notebooks/soil_grain_size_prediction.ipynb` in Google Colab.
2. Upload `Training_labels_updated.csv` and `ppm_updated.csv` to `/content`.
3. Place the training and test image folders in Google Drive.
4. Check and update `TRAIN_IMG_DIR` and `TEST_IMG_DIR` in the first setup cell.
5. Run the notebook from top to bottom.
6. The final cells create and validate `/content/submission.csv`, then offer it for download.


## Important limitations

- Only 24 labeled soil samples were available. This is a very small dataset for learning
  a general image-to-grain-size relationship.
- LOOCV evaluates held-out samples, but does not fully reproduce the camera distribution
  shift between training and test photographs.
- Training photographs primarily came from Motorola and Samsung devices, whereas test
  photographs came from iPhone 14 / iPhone 16 devices. Color, sharpening, exposure, and
  image processing differences may affect the handcrafted features.
- The features describe image appearance and texture; they do not explicitly segment or
  measure individual soil particles.
- Low within-site prediction variation indicates consistency among photos, not accuracy.
- Some predicted curves reached 100% at relatively small grain sizes or contained flat
  sections. These should be interpreted cautiously.

## Data and usage

This Kaggle competition is hosted as part of Geotechnical Resilience via Intelligent Design (GRID), an EU research initiative bringing data-driven methods to geotechnical engineering — from soil characterisation and foundation design to climate-adapted infrastructure.

Learn more at: https://grid.boku.ac.at/

train.csv — the training labels: cumulative grain size distributions of the 24 training soil samples Each row represents one soil sample; each sample is also represented by multiple photographs in the image folder, where the file naming links the images to the sample ID in this CSV.

test.csv — the test set: list of sample IDs for which predictions must be submitted. The corresponding grain size distributions are not disclosed.

sample_submission.csv — a sample submission file in the correct format. It contains all required test sample IDs and a placeholder distribution.

All_Photos_training/test —two folder containing the photographs of each soil sample. The image file names link each photograph to its sample_id. The camera with which the photo was taken is indicated in the name of each photo

ppm.csv — contains the pixel-per-millimeter (PPM) resolution and technical specifications of each camera used in the dataset. The camera used for each photo is indicated in the photo name itself

The grain size distribution is described as a cumulative curve at 11 fixed grain diameters defined by DIN EN ISO 14688-1. For each diameter, the value indicates the percentage of the soil mass that consists of particles smaller than that diameter. By definition, the curve is monotonically non-decreasing, bounded between 0 % and 100 %, and equals 100 % at the largest diameter (200 mm).



