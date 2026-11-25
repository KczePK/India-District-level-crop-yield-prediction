# India District-Level Crop Yield Prediction

Predicting district-level rice yield in India from fertilizer use, rainfall, soil type and location, using 38 years of public agricultural data from ICRISAT.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/KczePK/India-District-level-crop-yield-prediction/blob/main/India%27s_Agriculture_crop_yield_prediction_model_using_ICRISAT_District_Level_Database.ipynb)
&nbsp;[View on nbviewer](https://nbviewer.org/github/KczePK/India-District-level-crop-yield-prediction/blob/main/India%27s_Agriculture_crop_yield_prediction_model_using_ICRISAT_District_Level_Database.ipynb) (use this if GitHub does not display the notebook)

---

## Business question

A policy team wants a **numeric prediction of each district's rice yield (kg/ha)**, to compare districts and decide which crops and districts are worth investing in.

The approach is designed to be adapted to Thailand, using soil data from the Land Development Department (กรมพัฒนาที่ดิน).

## Approach

| Step | What was done |
|---|---|
| Data integration | Joined 5 ICRISAT tables (yield, fertilizer, rainfall, soil, location) on state, district and year; 1980–2017 |
| Soil parsing | Custom scikit-learn transformer that turns free-text soil descriptions (`'ORTHIDS - 85%; VERTI SOLS - 15%'`) into percentage shares of 6 soil orders, including spelling variants in the source |
| Feature engineering | Soil × fertilizer interactions (different soils respond differently to nutrients) and N:P:K ratios |
| Preprocessing pipeline | Replacing `-1` placeholders, median imputation, log transforms for skewed features and scaling, all inside one `ColumnTransformer` |
| Train / test split | Time-based: train 1980–2010, test 2011–2017 (about 80/20) |
| Model comparison | 7 regression models compared with 5-fold cross-validation |
| Tuning | `RandomizedSearchCV` on the two best models |

## Results

Cross-validation RMSE (kg/ha, 5 folds, training data):

| Model | CV RMSE |
|---|---|
| Linear Regression | 785 |
| K-Nearest Neighbors | 630 |
| Support Vector Regression | 601 |
| Random Forest | 597 |
| XGBoost | 570 |
| HistGradientBoosting | 561 → **546 after tuning** |
| ExtraTrees | 558 → 559 after tuning |

Test RMSE on 2011–2017:

| Model | Test RMSE |
|---|---|
| HistGradientBoosting (selected) | 742 |
| ExtraTrees | 702 |

**Most important features (ExtraTrees):** nitrogen per hectare, latitude, phosphate, share of Vertisol soils, longitude and potash. Latitude and longitude most likely stand in for factors the data does not include, such as climate, irrigation access and farm management.

## Review notes and next steps

Looking back at this project, there are a few things I would do differently (also noted at the end of the notebook):

- I didn't compare the model against a simple baseline. Just using each district's yield from last year gives an RMSE of about 520 on the test years, which is better than my model (742).
- My cross-validation split the data by district rather than by year, so the CV score (546) looked better than the test result.
- I dropped the year column, so the model can't pick up that yields keep rising over time.
- The relative error should use the test-period mean, which makes it about 35% instead of 41%.

For the next iteration:
1. Add simple baselines to compare against.
2. Add past yields for each district and a time trend.
3. Use year-based cross-validation.
4. Add irrigation (`data/IrrigatedArea.csv`) and monsoon rainfall (June–September).
5. Adapt the pipeline to Thai data (Land Development Department soil data, Office of Agricultural Economics yields).

## How to run

1. Click **Open in Colab** above.
2. **Runtime → Run all.** All data loads directly from this repository; no uploads are needed.

To model another crop, change the target column in the notebook, for example `"WHEAT YIELD (Kg per ha)"`.

## Repository structure

```
India's_Agriculture_crop_yield_prediction_model_using_ICRISAT_District_Level_Database.ipynb
                                   Main notebook
data/
  AreaProductionYield.csv          Area, production and yield for 23 crops, 1966–2017
  FertilizerConsumption.csv        Nitrogen, phosphate and potash use
  RainFall.csv                     Monthly and annual rainfall
  SoilType.csv                     Soil composition per district (free text)
  Identifiers.csv                  District latitude, longitude and region codes
  IrrigatedArea.csv                Irrigated area by crop (for the next iteration)
DATASET_LICENSE.md                 Data license and attribution
```

## Data source and license

Data derived from the **ICRISAT District Level Database (DLD) for Indian agriculture and allied sectors**, licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). See [`DATASET_LICENSE.md`](DATASET_LICENSE.md).

> Swamikannu, N., Pingali, P. R., Padmanabhan, J., Ravula, P., Gumma, M. K., Mohammed, I. A., & Rathore, A. (2021). *District Level Data (DLD) and Dashboard for Agriculture and Allied-sectors in India*. ICRISAT Dataverse, V1. https://doi.org/10.21421/D2/XFB1BZ

## Author

**Nawa Penparkgoon**, Data Scientist
GitHub: [KczePK](https://github.com/KczePK)
