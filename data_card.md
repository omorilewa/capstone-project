# Data card: CBIS-DDSM mass patches

## Summary

| | |
|---|---|
| Dataset | CBIS-DDSM (Curated Breast Imaging Subset of the Digital Database for Screening Mammography), **mass** cases only |
| Data type | Mammogram image patches around each outlined mass, with matching outline masks |
| Task | Binary classification of each mass: benign (0) or malignant (1) |
| Size used | 1,696 masses from 892 patients, on 1,592 mammograms |
| Source | The Cancer Imaging Archive (TCIA), original DICOM images |
| Licence | CC BY 3.0; TCIA Data Usage Policy applies |
| Collected by this project | Nothing. No new data was collected; this is a public, de-identified dataset |

## Source and citation

CBIS-DDSM is a curated, standardised subset of the Digital Database for Screening Mammography (DDSM), selected
and curated by a trained mammographer, with images decompressed and converted to DICOM and updated outlines
of each lesion. It is distributed by The Cancer Imaging Archive under a CC BY 3.0 licence.

Required citations:

- Sawyer-Lee, R., Gimenez, F., Hoogi, A., & Rubin, D. (2016). *Curated Breast Imaging Subset of Digital Database
  for Screening Mammography (CBIS-DDSM)* (Version 1) [Data set]. The Cancer Imaging Archive.
  https://doi.org/10.7937/K9/TCIA.2016.7O02S9CY
- Lee, R. S., Gimenez, F., Hoogi, A., Miyake, K. K., Gorovoy, M., & Rubin, D. L. (2017). A curated mammography
  data set for use in computer-aided detection and diagnosis research. *Scientific Data, 4*, 170177.
  https://doi.org/10.1038/sdata.2017.177
- Clark, K., et al. (2013). The Cancer Imaging Archive (TCIA): Maintaining and operating a public information
  repository. *Journal of Digital Imaging, 26*(6), 1045–1057. https://doi.org/10.1007/s10278-013-9622-7

## What the data is

- **Images:** scanned (digitised) screen-film mammograms from the United States, collected in the 1990s.
  Two standard views per breast: CC (top-down) and MLO (angled side view).
- **Outlines:** a hand-drawn outline (region of interest) of each mass, as a binary mask the same size as the mammogram.
- **Case lists:** two CSV files, `mass_case_description_train_set.csv` and `mass_case_description_test_set.csv`,
  with one row per outlined mass: patient ID, breast, view, breast density, mass shape and margins, BI-RADS
  assessment, subtlety, pathology, and the paths of the image files.
- **Not included:** patient age, ethnicity, or any information about where each image was taken. These cannot
  be analysed.

## Labels

| Pathology in the case list | Count (training list / test list) | Label used |
|---|---|---|
| MALIGNANT | 637 / 147 | 1 |
| BENIGN | 577 / 194 | 0 |
| BENIGN_WITHOUT_CALLBACK | 104 / 37 | 0 |

BENIGN_WITHOUT_CALLBACK masses were judged benign on imaging and the patient was not recalled for biopsy.
They are counted as benign, so the labels are **pathology-verified or radiologist-assessed**, not all
biopsy-confirmed.

## Splits

The official CBIS-DDSM training/test split was kept. 15% of the patients in the official training list were
held out as a validation set (`GroupShuffleSplit`, seed 42). No patient appears in more than one set; this
was checked in notebooks 01, 03 and 04.

| Split | Masses | Patients | Benign | Malignant | Used for |
|---|---|---|---|---|---|
| Training | 1,121 | 587 | 589 | 532 | Training the models |
| Validation | 197 | 104 | 92 | 105 | Choosing the stopping epoch, temperature and threshold |
| Test | 378 | 201 | 231 | 147 | Final results only, once |

## Processing (notebooks 01–02)

1. For each mass, the full mammogram and the outline were downloaded from TCIA as original DICOM.
2. A square patch was cut around the outline: the larger of the mass's height and width, plus 25% on each
   side; at least 64 pixels; shrunk to 1,024 pixels if larger. The same square was cut from the outline.
3. Patches were saved as **16-bit PNG** (lossless, original pixel values); outlines as 8-bit PNG.
4. Result: 1,696 patches, **none excluded**. Eight examples were checked visually; the outline sat on the mass in all.
5. For the models (notebooks 03–05), each patch is converted to 8-bit by dividing by 257 and resized to
   224×224 pixels.

The patches themselves are **not redistributed** with this project. They can be regenerated from TCIA by
running notebooks 01 and 02.

## Known limitations and who is missing

- **Population.** CBIS-DDSM comes from the United States in the 1990s. Nigerian and other African women are
  not represented. Breast density and age at presentation differ between populations. Results cannot be assumed to transfer to Nigerian patients; that needs a
  separate, ethically approved study on local images.
- **Imaging technology.** The images are scanned film. Most Nigerian centres that offer mammography use
  digital systems, which produce different-looking images.
- **Case mix.** DDSM cases were selected for a research database. Roughly half the masses here are malignant,
  far more than in a screening population. Sensitivity and specificity are affected less than predictive values,
  which would be much lower in screening.
- **Patches, not whole mammograms.** Every patch is centred on a known mass. The task is to classify a mass
  that has already been found, not to find it.
- **Outlines are approximate.** Each outline is one reader's drawing; spiculated malignant masses can extend
  beyond it.
- **Label noise.** Benign-without-callback masses have no biopsy confirmation.
- **Two views per mass.** Most masses appear in both CC and MLO views. They are kept as separate patches, and
  confidence intervals are computed by resampling patients, not images.

## Ethics and privacy

- The data are public and de-identified. No patient was contacted and no new data was collected.
- Use follows the CC BY 3.0 licence and the TCIA Data Usage Policy, with the citations above.
- The data and the models trained on it are for research and education only, not for clinical use.
