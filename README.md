# Explainable breast mass classification under low-resource conditions

A small image model (MobileNetV2) classifies breast masses on mammogram patches as benign or malignant. The
project then asks whether it still works, and whether its heatmap explanations still point at the mass, under
conditions an under-resourced clinic may have: compressed, low-resolution or photographed images, and a
laptop with no GPU.

**Team:** [names and affiliations]
**Programme:** AI Research Foundations for Higher Education, Nigeria Cohort 1, capstone project

**Research and education only. Not for clinical use.**

## Files

| File | What it is |
|---|---|
| `01-data-preparation.ipynb` | Reads the CBIS-DDSM mass case lists, sets labels, splits by patient |
| `02-patch-extraction.ipynb` | Downloads the original DICOM images and cuts a patch and outline around each mass |
| `03-training-and-calibration.ipynb` | Baseline and fine-tuned MobileNetV2; temperature and threshold locked on validation |
| `04-testing-and-robustness.ipynb` | Test results on clean and degraded images; 8-bit low-resource model; speed |
| `05-heatmaps.ipynb` | Grad-CAM, Grad-CAM++ and occlusion, checked against the outlines and with weight randomisation |
| `data_card.md` | The dataset: source, licence, labels, splits, processing, limitations |
| `model_card.md` | The model: intended use, training, results, limitations, prohibited uses |

Each notebook is saved **with its outputs**, so the results can be read without rerunning anything.

## How to run

The notebooks run on **Kaggle** (not Colab): the input paths are Kaggle's `/kaggle/input`. Run them in order.
Each one uses the previous notebook's output as an input (*Add Input* on Kaggle).

| Notebook | Inputs to attach | GPU | Internet | Approx. time |
|---|---|---|---|---|
| 01 | CBIS-DDSM mass case lists (two CSV files) | No | No | < 1 min |
| 02 | Output of 01 | No | **Yes** (downloads from TCIA) | ~11 min |
| 03 | Output of 02 | Yes | Yes (ImageNet weights) | ~15 min |
| 04 | Outputs of 02 and 03 | Yes | Yes (ImageNet weights) | ~20–40 min |
| 05 | Outputs of 02 and 03 | Yes | No | ~10–20 min |

Run each with *Save Version → Save & Run All* so its outputs are kept for the next notebook.

The two case-list CSV files (`mass_case_description_train_set.csv` and `mass_case_description_test_set.csv`)
are available from the CBIS-DDSM page of The Cancer Imaging Archive.

Main libraries: TensorFlow 2.21 / Keras 3, scikit-learn, pandas, NumPy, OpenCV, pydicom, SciPy, Matplotlib.
Random seed 42 throughout.

## Method in brief

1. **Data:** 1,696 masses from 892 patients (CBIS-DDSM). Official train/test split kept; 15% of training
   patients held out for validation. No patient in more than one set.
2. **Patches:** a square around each radiologist-drawn outline, from the original DICOM, saved losslessly.
3. **Models:** a baseline (frozen ImageNet MobileNetV2 features + logistic regression) and a fine-tuned
   MobileNetV2 (head first, then upper layers), 224×224 input.
4. **Locked on validation:** temperature scaling for calibration; a threshold giving at least 90% sensitivity.
5. **Testing, once:** AUROC, sensitivity, specificity and calibration error with 95% confidence intervals
   (bootstrap by patient), on clean images and under 15 degradations; an 8-bit TensorFlow Lite version timed on a CPU.
6. **Explanations:** heatmaps compared with the outlines and with a model-free centre blob, plus a
   weight-randomisation check.

## Data and licence

CBIS-DDSM is distributed by The Cancer Imaging Archive under **CC BY 3.0**. No images are included in this
folder; notebooks 01 and 02 download them. Please cite:

- Sawyer-Lee, R., Gimenez, F., Hoogi, A., & Rubin, D. (2016). *Curated Breast Imaging Subset of Digital Database
  for Screening Mammography (CBIS-DDSM)* (Version 1) [Data set]. The Cancer Imaging Archive.
  https://doi.org/10.7937/K9/TCIA.2016.7O02S9CY
- Lee, R. S., Gimenez, F., Hoogi, A., Miyake, K. K., Gorovoy, M., & Rubin, D. L. (2017). A curated mammography
  data set for use in computer-aided detection and diagnosis research. *Scientific Data, 4*, 170177.
  https://doi.org/10.1038/sdata.2017.177
- Clark, K., et al. (2013). The Cancer Imaging Archive (TCIA): Maintaining and operating a public information
  repository. *Journal of Digital Imaging, 26*(6), 1045–1057. https://doi.org/10.1007/s10278-013-9622-7

Other methods cited:

- Sandler, M., Howard, A., Zhu, M., Zhmoginov, A., & Chen, L.-C. (2018). MobileNetV2: Inverted residuals and
  linear bottlenecks. *Proceedings of CVPR*, 4510–4520.
- Guo, C., Pleiss, G., Sun, Y., & Weinberger, K. Q. (2017). On calibration of modern neural networks.
  *Proceedings of ICML*, 1321–1330.
- Selvaraju, R. R., et al. (2017). Grad-CAM: Visual explanations from deep networks via gradient-based
  localization. *Proceedings of ICCV*, 618–626.
- Chattopadhay, A., Sarkar, A., Howlader, P., & Balasubramanian, V. N. (2018). Grad-CAM++: Generalized
  gradient-based visual explanations for deep convolutional networks. *Proceedings of WACV*, 839–847.
- Zeiler, M. D., & Fergus, R. (2014). Visualizing and understanding convolutional networks. *Proceedings of ECCV*, 818–833.
- Adebayo, J., Gilmer, J., Muelly, M., Goodfellow, I., Hardt, M., & Kim, B. (2018). Sanity checks for saliency
  maps. *Advances in Neural Information Processing Systems, 31*.
