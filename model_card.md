# Model card: MobileNetV2 breast mass classifier (benign vs malignant) for low-resource settings

**Research and education only. Not for clinical use.**

## Model details

| | |
|---|---|
| Developed by | [Team name and members], AI Research Foundations for Higher Education, Nigeria Cohort 1 |
| Date | November 2026 |
| Model type | Convolutional neural network image classifier |
| Architecture | MobileNetV2 (ImageNet weights), global average pooling, 30% dropout, one output (malignancy score) |
| Input | One grey mammogram patch centred on an already-outlined mass, resized to 224×224 (copied to 3 channels, pixel values 0–255) |
| Output | Probability that the mass is malignant (after temperature scaling), and a benign/malignant decision at a fixed threshold |
| Versions | Full model (Keras, 32-bit) and an 8-bit TensorFlow Lite model for CPU-only computers |
| Framework | TensorFlow 2.21 / Keras 3; TensorFlow Lite (LiteRT) for the 8-bit model |
| Code | Notebooks 01–05 in this folder |
| Licence of training data | CBIS-DDSM, CC BY 3.0 (see `data_card.md`) |

**Training** (notebook 03): head trained first with the network frozen (learning rate 0.001, up to 15 epochs),
then layers from `block_13_expand` upward unfrozen with batch-normalisation layers kept frozen (learning rate
0.00002, up to 30 epochs). Augmentation: flips, rotation up to about 11°, contrast ×0.85–1.15, brightness ±15.
Class-weighted loss. Early stopping on validation AUROC (patience 6). One run, seed 42, no hyperparameter search.
The best model came from stage 2, epoch 3.

**Calibration and decision threshold** (fixed on the validation set before testing):
temperature 1.208; threshold 0.279 (the highest probability that still flagged at least 90% of malignant
validation masses). The 8-bit model uses its own temperature, 1.326, also fitted on validation.

**Baseline for comparison:** the same MobileNetV2 left untrained, its 1,280 features fed to a logistic regression.

## Intended use

- **Research** on how a small image classifier and its explanations behave when images are degraded and only a
  CPU is available.
- **Teaching** about classification, calibration, robustness testing and checking explanation methods in medical imaging.

**Intended users:** researchers and students. Not patients, and not clinicians making decisions about patients.

## Out-of-scope and prohibited uses

- Diagnosing, screening or triaging real patients, alone or as a second reader.
- Telling anyone that a mass is benign or that they do not need follow-up. At the chosen threshold the model
  still misses about 8% of malignant masses.
- Finding masses in a whole mammogram. The model only classifies a patch already centred on a known mass.
- Use on digital mammograms, other populations, or other lesion types (calcifications, architectural distortion)
  without new validation on that data.
- Treating the heatmaps as evidence of where a lesion is or why the model decided. Section "Explanations" shows
  they are not reliable for this.

## Factors

- **Image quality** is the main factor studied: JPEG compression, lower resolution, noise, contrast, brightness,
  and a simulated phone photograph of film.
- **Population and equipment:** trained and tested only on scanned film from the United States (1990s).
  Performance on Nigerian patients and on digital mammography is unknown.
- **Not available for analysis:** patient age and ethnicity are not in the dataset.

## Metrics

| Metric | Why |
|---|---|
| AUROC | Ranking ability, independent of threshold (0.5 = guessing) |
| Sensitivity, specificity | What happens at the locked threshold: cancers caught, benign masses cleared |
| Expected calibration error (ECE) | Whether the probability shown can be taken at face value (0 = perfect) |
| Size and time per image | Feasibility on ordinary hardware |

95% confidence intervals come from 1,000 bootstrap resamples **by patient**, because most masses appear in two views.

## Data

- **Training:** 1,121 patches (587 patients); 589 benign, 532 malignant.
- **Validation:** 197 patches (104 patients); used only for early stopping, temperature and threshold.
- **Test:** 378 patches (201 patients); 231 benign, 147 malignant. Used once, after everything was fixed.
- Benign includes 141 benign-without-callback masses (not biopsied). See `data_card.md`.

## Results: clean test images (notebook 04)

| Model | AUROC (95% CI) | Sensitivity | Specificity | ECE |
|---|---|---|---|---|
| Fine-tuned (full) | 0.778 (0.721–0.835) | 0.918 | 0.312 | 0.074 |
| 8-bit (TensorFlow Lite) | 0.775 (0.717–0.832) | 0.959 | 0.242 | 0.095 |
| Baseline (frozen features + logistic regression) | 0.719 (0.663–0.774) | 0.905 | 0.312 | 0.195 |

| Paired comparison (same bootstrap resamples) | AUROC difference (95% CI) |
|---|---|
| Fine-tuned − baseline | +0.059 (+0.009 to +0.111) |
| 8-bit − fine-tuned | −0.003 (−0.029 to +0.022) |

- Fine-tuning gave a modest gain over generic features that is unlikely to be chance, and much better calibration.
- The test AUROC (0.778) is consistent with validation (0.767). It is slightly below published fine-tuned CNNs on
  CBIS-DDSM mass patches (around 0.80–0.84), at a smaller input size and with a single untuned run.
- At the locked threshold, catching about 92% of malignant masses means flagging about 69% of benign masses.
- Quantising to 8 bits cost no measurable AUROC, but shifted the probabilities so that the 8-bit model flags more
  masses (sensitivity 0.959, specificity 0.242) at the same threshold.

## Results: degraded test images (notebook 04)

The worst condition was JPEG quality 10, where AUROC fell to **0.549**, close to guessing.

[To complete from `table_degraded.csv`: AUROC, sensitivity, specificity and ECE for each of the 15 degradations,
for the fine-tuned, 8-bit and baseline models.]

## Size and speed

| Model | Size | Kaggle server CPU, 1 thread (median) | Laptop, 1 thread (median / 95th percentile) |
|---|---|---|---|
| 32-bit TensorFlow Lite | 8.9 MB | 8.0 ms | 7.2 ms / 8.7 ms |
| 8-bit TensorFlow Lite | 2.7 MB | 11.8 ms | **4.3 ms / 4.8 ms** |

Laptop: HP Pavilion x360 14, Intel Core i3-1125G4 (4 cores, 8 threads, 2.0 GHz), 8 GB RAM, integrated
Intel UHD Graphics only (no separate GPU), Windows 11, Python 3.10, LiteRT 2.4.0. One patch at a time; times
cover the model only, not loading or preparing the image.

On the laptop the 8-bit model was about 1.7 times faster than the 32-bit one and 3.3 times smaller. On Kaggle's
server CPU it was slower than the 32-bit model, so the benefit depends on the processor and should be measured
on the target machine.

## Explanations (notebook 05)

Heatmaps were made for all 378 test patches with Grad-CAM, Grad-CAM++ and occlusion, and compared with the
radiologist's outline. Because every patch is centred on its mass, each method is compared with a model-free
"centre blob". The outline covers on average 24.7% of the patch.

| Method | Hottest point inside outline (95% CI) | Share of heat inside outline (95% CI) | Similarity after scrambling final layer | Similarity after scrambling all layers |
|---|---|---|---|---|
| Centre blob (no model) | 0.976 (0.958–0.994) | 0.493 (0.485–0.502) | — | — |
| Grad-CAM | 0.574 (0.524–0.623) | 0.439 (0.416–0.463) | 0.33 | −0.31 |
| Grad-CAM++ | 0.751 (0.706–0.793) | 0.501 (0.491–0.513) | **0.98** | −0.57 |
| Occlusion | 0.339 (0.288–0.391) | 0.194 (0.169–0.221) | −0.03 | −0.02 |

Similarity is the Spearman correlation between the original and scrambled heatmaps (1 = unchanged), on 100 test
patches. A method that explains the model should change when the model is scrambled.

- **No method located the mass better than the model-free centre blob.** Grad-CAM++ only tied with it on share
  of heat (+0.008, 95% CI 0.000 to 0.015) and was worse on the hottest point.
- **Grad-CAM++ failed the final-layer randomisation check:** its heatmaps barely changed when the final layer was
  scrambled, so they mostly reflect image structure rather than the model's decision.
- **Grad-CAM and occlusion passed the randomisation checks but localised poorly.** Occlusion put less heat inside
  the outline than its area alone would give, suggesting the model's score depends more on tissue around or at the
  edge of the mass.
- **Explanations were unstable under degradation:** Grad-CAM similarity to the clean-image heatmap fell to 0.29 at
  JPEG quality 10 and 0.43 for a strong simulated phone photo.

Caution: the outlines are hand-drawn and approximate, and spiculated malignant masses extend beyond them, so
heat just outside an outline is not necessarily wrong.

## Ethical considerations

- **Risk of harm if misused:** a false "benign" could delay cancer diagnosis; false alarms cause anxiety and
  unnecessary procedures. Hence the prohibited uses above.
- **Misleading explanations:** heatmaps can look convincing while not reflecting the model's reasoning. This
  model's heatmaps should not be shown to clinicians or patients as justification for a prediction.
- **Representation:** the training data contains no Nigerian or other African patients. Deploying it in Nigeria
  without local validation could perform worse for exactly the women it is meant to help.
- **Privacy:** only public, de-identified data were used; no patient data were collected.

## Caveats and limitations

- One training run with one seed and no hyperparameter search; run-to-run variation is not measured.
- Small validation set (197 patches), so the threshold and temperature are themselves uncertain.
- 224×224 input loses fine detail of mass margins in larger patches.
- Degradations are simulated, not real images from Nigerian clinics.
- Benign-without-callback masses (141) have no biopsy confirmation.
- The case mix (about 46% malignant) is far richer in cancer than a screening population.

## About this card

This card follows the format proposed by Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., Spitzer, E., Raji, I. D., &
Gebru, T. (2019), *Model cards for model reporting*, in *Proceedings of the Conference on Fairness,
Accountability, and Transparency*, 220–229. https://doi.org/10.1145/3287560.3287596. Anyone using this work
should cite the dataset as listed in `data_card.md`.
