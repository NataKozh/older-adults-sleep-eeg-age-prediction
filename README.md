# Sleep EEG age prediction in older adults

This directory provides a linear model that estimates chronological age (in years) from six features of central EEG during N2 and N3 sleep. It uses Luna’s [`PREDICT` command](https://zzz-luna.org/luna/ref/predict/) and follows the same three-file layout as the SUN2019 example.

| File | Purpose |
| --- | --- |
| [`m3-old-age-features.txt`](m3-old-age-features.txt) | Defines the six predictors, training means and standard deviations, regression coefficients, model intercept, and age-bias correction. |
| [`m3-old-age-data.txt`](m3-old-age-data.txt) | Contains 3,958 rows of standardised reference values for the six predictors. Luna uses these values for 10-nearest-neighbour imputation. The model requires at least four observed features. |
| [`m3-old-age-luna.txt`](m3-old-age-luna.txt) | Resamples the EEG to 125 Hz, converts it to µV, applies a 0.3–30 Hz band-pass filter, masks artefacts, and retains N2 and N3 epochs. The script computes and caches the required spectral and spindle measures, then runs `PREDICT`. |

## Model development

We trained the model on 70% of Sleep Heart Health Study (SHHS) Visit 1 and reserved the other 30% for held-out testing. Starting with 195 candidate features, we excluded those that differed most strongly between the SHHS training sample and a Cleveland Family Study (CFS) subset restricted to the SHHS age range (39–89 years). This screen used linear regression adjusted for age, sex, and self-reported race and ethnicity (`p < 10⁻³⁰`) to favour features that transfer across recording protocols.

Among the remaining features, we retained those most strongly associated with age in the SHHS training sample after adjustment for sex and self-reported race and ethnicity (`p < 10⁻⁵⁰`). For highly correlated features (Pearson’s `r > 0.70`), we prioritised the one more strongly associated with age. We then removed two of the eight remaining features whose coefficients did not differ significantly from zero in the joint model. The final linear regression used these six features:

- Fast sigma power (13–15 Hz)
- Fast spindle integrated activity per spindle
- Time from fast spindle onset to the maximum wavelet coefficient
- Fast spindle overlap with slow oscillations (SOs)
- Normalised phase consistency of slow spindle–SO coupling
- Normalised slow spindle overlap with SOs

We evaluated the final model in the held-out SHHS sample and in MESA, WSC, CFS, and MrOS.

## Running the model

From this directory, supply a Luna sample list with an EDF and sleep-stage annotations. Set `cen` to a single central EEG channel already referenced to the contralateral mastoid. The supplied script assumes this reference is already in place. Set `age` to the individual’s chronological age in years for bias correction.

```sh
luna sample.lst cen=C4_M1 age=69 mpath=. -o predictions.db < m3-old-age-luna.txt
```

This example assumes one recording. For a sample list with multiple participants, supply individual ages using [Luna variables](https://zzz-luna.org/luna/ref/predict/#using-variables).

Luna returns the raw predicted age (`Y`) and the bias-adjusted predicted age (`Y1`). The raw predicted age difference (PAD) is `Y − chronological age`. The correction subtracts the expected PAD at that age, estimated from the slope and intercept of the PAD–age regression in the SHHS training sample. For held-out or external samples, calculate age-adjusted PAD as `Y1 − chronological age`.
