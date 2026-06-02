# DSC 232R Final Project: Distributed Seismic Wave Arrival Prediction

**Dataset:** [STEAD: Stanford Earthquake Dataset](https://github.com/smousavi05/STEAD)  
**Primary objective:** Predict seismic wave arrival behavior from large-scale waveform data using distributed data processing.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Repository and Data Access](#repository-and-data-access)
3. [Computing Environment](#computing-environment)
4. [Methods](#methods)
   - [Data Exploration](#data-exploration)
   - [Preprocessing](#preprocessing)
   - [Model 1: Baseline Distributed Regression Models](#model-1-baseline-distributed-regression-models)
   - [Model 2: SVD Dimensionality Reduction and PhaseNet](#model-2-svd-dimensionality-reduction-and-phasenet)
5. [Results](#results)
   - [Exploratory Data Results](#exploratory-data-results)
   - [Model 1 Results](#model-1-results)
   - [Model 2 Results](#model-2-results)
   - [Prediction Examples](#prediction-examples)
   - [Speedup and Framework Comparison](#speedup-and-framework-comparison)
6. [Discussion](#discussion)
7. [Conclusion](#conclusion)
8. [Related Work](#related-work)
9. [Statement of Collaboration](#statement-of-collaboration)
10. [Extra Credit: Spark vs. Ray Framework Comparison](#extra-credit-spark-vs-ray-framework-comparison)

---

## Introduction

Earthquake waveform data is fundamentally different from ordinary tabular data because each observation contains a multi-channel time-series signal. We selected the STEAD dataset because it allows us to study a high-impact prediction problem: identifying seismic wave arrival behavior from large volumes of waveform data.

The project focuses on predicting S-wave arrival timing and analyzing waveform behavior. S-waves are especially important because they often produce stronger ground motion than P-waves, meaning that accurate prediction of S-wave arrival can support earlier warning systems and potentially reduce earthquake-related harm. In addition, distinguishing earthquake waveforms from noise is valuable for improving the reliability of automated seismic monitoring systems.

This project required big data and distributed computing because the dataset was too large to process efficiently on a single machine. The original HDF5 waveform data contains 1,268,314 events, where each waveform is shaped as `(6000, 3)`. Before missing-data handling, this corresponds to more than 7.6 billion waveform sample values. Without Spark or Ray, loading, preprocessing, reshaping, joining, and training on the dataset would have been impractical. Distributed computing allowed us to parallelize preprocessing and model training while managing memory constraints on SDSC Expanse.

---

## Repository and Data Access

### Original Dataset

The original dataset is available from the STEAD repository:

- <https://github.com/smousavi05/STEAD>

### Project Data Files

We used 5 main data files/directories:

| File/Directory | Description |
|---|---|
| `merge.csv` | Metadata file containing earthquake and noise trace information |
| `merge.hdf5` | Waveform file containing raw 3-channel seismic waveform arrays |
| `stead_combined.parquet` | Combined Parquet file generated from the CSV and HDF5 files |
| `stead_version4` | Parquet files containing waveform data, separating the 3 waveform channels into separate columns |
| `denoised_waveforms_parquets` | Parquet files containing SVD-applied waveform data with "noise" `trace_category` excluded |

### File Loading on SDSC Expanse

Before running notebooks on Expanse, copy the Parquet file into the active scratch job directory.

```bash
ls /scratch/$(whoami)
```

Then copy the data using one of the following commands, depending on the active scratch path:

```bash
cp ~/ysuh2/data/stead_combined.parquet /expanse/lustre/scratch/$(whoami)/{jobname}
```

or

```bash
cp ~/ysuh2/data/stead_combined.parquet /scratch/$(whoami)/{jobname}
```

### Notebooks

The following notebooks contain the main project code:

| Notebook | Purpose |
|---|---|
| `EDA.ipynb` | EDA using Spark with parquet file |
| `hdf5_2_parquet.ipynb` | Converts and combines the HDF5 waveform data and CSV metadata into Parquet format |
| `phasenet_no_svd_prediction_analysis.ipynb` | model evaluation on phasenet without SVD preprocessing on waveform data |
| `phasenet_no_svd.ipynb` | PhaseNet training using Ray with no SVD preprocessing |
| `phasenet_svd.ipynb` | PhaseNet training using Ray with SVD preprocessing  |
| `ray xgboost.ipynb` | Ray XGBoost implementation |
| `rayio_svd.ipynb` | Applied SVD into waveform data |
| `s_wave_pipeline_1_executor.ipynb` | Spark speedup comparsion for RandomForest Regressor |
| `s_wave_pipeline_7_executors.ipynb` | Spark speedup comparsion for RandomForest Regressor |
| `xgboost_test_7.30.ipynb` | Spark XGBoost implementation |


---

## Computing Environment

### SDSC Expanse Spark Environment

We used SDSC Expanse with Spark for distributed preprocessing and model training.

| Resource | Value |
|---|---:|
| Total cores | 8 |
| Total memory | 128 GB |
| Driver memory | 4 GB |

### Executor Planning for HDF5 / Parquet Processing

The initial executor estimate was:

```text
Executor instances = 8 - 1 = 7
Executor memory = (128 - 4) // 7 = 17 GB
```

In practice, we found that 6 executors with 20 GB per executor performed better than 7 executors with 17 GB per executor.

```python
# Spark configuration for converted Parquet processing
spark = (
    SparkSession.builder
    .config("spark.driver.memory", "4g")
    .config("spark.executor.instances", "6")
    .config("spark.executor.memory", "20g")
    .getOrCreate()
)
```

### Executor Planning for Metadata EDA

Because the metadata CSV file is approximately 400 MB, we used a smaller Spark configuration for metadata-only EDA.

```python
# Spark configuration for metadata EDA
spark = (
    SparkSession.builder
    .config("spark.driver.memory", "1g")
    .config("spark.executor.instances", "1")
    .config("spark.executor.memory", "500m")
    .getOrCreate()
)
```

### Spark UI Evidence

The following Spark UI screenshot shows evidence from the larger Spark configuration used for Parquet processing.

<img width="730" height="365" alt="Spark UI evidence for larger Spark job" src="https://github.com/user-attachments/assets/d6d319cd-1ffb-48d7-8f20-3e30f674840c" />

We initially used 2 GB of driver memory, but the large dataset required increasing the driver memory to 4 GB. We also observed that 6 executors with 20 GB each ran faster than 7 executors with 17 GB each. This result is referenced in the final cell of `hdf5_2_parquet.ipynb`.

The following Spark UI screenshot shows the smaller metadata EDA configuration.

<img width="725" height="360" alt="Spark UI evidence for metadata EDA" src="https://github.com/user-attachments/assets/9efb94fb-05d1-4c80-ad0c-37b8a89a770c" />

For metadata EDA, we allocated 1 GB to the driver and 500 MB to a single executor because the metadata file was much smaller than the waveform data.

---

# Methods

## Data Exploration

### Dataset Scale

The full project uses three dataset representations: `merge.csv`, `merge.hdf5`, and `stead_combined.parquet`.

| Dataset | Number of observations | Notes |
|---|---:|---|
| `merge.csv` | 1,268,314 | Metadata observations |
| `merge.hdf5` | 1,268,314 | Waveform observations corresponding to metadata traces |
| `stead_combined.parquet` | 1,265,657 | Combined dataset after joining and flattening waveform data |

Each HDF5 observation contains:

1. `trace_name`, the unique identifier for the waveform.
2. A waveform array with shape `(6000, 3)`.
3. A dictionary containing `p_arrival_sample`, `s_arrival_sample`, and `coda_end_sample`.

Before missing-data handling, the waveform data effectively contains:

```text
1,268,314 events × 6,000 waveform samples = 7,609,884,000 waveform sample rows
```

After dropping 5,314 rows with missing earthquake identifiers, we worked with approximately:

```text
7,578,000,000 waveform sample rows
```

### Numerical Metadata Summary

We computed summary statistics for metadata fields including receiver location, arrival samples, source depth, source magnitude, source distance, and back azimuth.

<details>
<summary>Click to expand numerical metadata summary</summary>

```text
RECORD 0-----------------------------------------------
 summary                          | count               
 receiver_latitude                | 1268314             
 receiver_longitude               | 1265657             
 receiver_elevation_m             | 1265657             
 p_arrival_sample                 | 1030231             
 p_weight                         | 1030057             
 p_travel_sec                     | 1030231             
 s_arrival_sample                 | 1030231             
 s_weight                         | 1030076             
 source_origin_uncertainty_sec    | 140294              
 source_latitude                  | 1030231             
 source_longitude                 | 1030231             
 source_error_sec                 | 459503              
 source_gap_deg                   | 380817              
 source_horizontal_uncertainty_km | 440738              
 source_depth_km                  | 1030182             
 source_depth_uncertainty_km      | 369423              
 source_magnitude                 | 1030231             
 source_distance_deg              | 1027574             
 source_distance_km               | 1027574             
 back_azimuth_deg                 | 1027574             
 coda_end_sample                  | 1027574             

RECORD 1-----------------------------------------------
 summary                          | mean                
 receiver_latitude                | 39.243258624484895  
 receiver_longitude               | -111.09354810763055 
 receiver_elevation_m             | 991.647229786382    
 p_arrival_sample                 | 661.043555468122    
 p_weight                         | 0.7128595893233322  
 p_travel_sec                     | 9.006534422262115   
 s_arrival_sample                 | 1335.1522172910325  
 s_weight                         | 0.6510318073614625  
 source_origin_uncertainty_sec    | 0.9475704591785838  
 source_latitude                  | 40.45908423705469   
 source_longitude                 | -116.71799423570586 
 source_error_sec                 | 0.41561243365113937 
 source_gap_deg                   | 106.82200183550917  
 source_horizontal_uncertainty_km | 1.3692877094101226  
 source_depth_km                  | 15.65473035832508   
 source_depth_uncertainty_km      | 1.3683928179891316  
 source_magnitude                 | 1.5260002465466593  
 source_distance_deg              | 0.45680988548757717 
 source_distance_km               | 50.79632535467068   
 back_azimuth_deg                 | 188.42516952744987  
 coda_end_sample                  | 2514.1423449795343  

RECORD 2-----------------------------------------------
 summary                          | stddev              
 receiver_latitude                | 18.02282428512557   
 receiver_longitude               | 51.29481347807843   
 receiver_elevation_m             | 685.8499845869212   
 p_arrival_sample                 | 175.8297530290482   
 p_weight                         | 0.20800895092003435 
 p_travel_sec                     | 7.441454748540921   
 s_arrival_sample                 | 605.4791890300081   
 s_weight                         | 0.24184524512983915 
 source_origin_uncertainty_sec    | 4.005288182513459   
 source_latitude                  | 13.798095105220117  
 source_longitude                 | 41.99185973383022   
 source_error_sec                 | 0.43653321182191107 
 source_gap_deg                   | 68.19063805544228   
 source_horizontal_uncertainty_km | 2.05842491199765    
 source_depth_km                  | 24.2123029616753    
 source_depth_uncertainty_km      | 2.0448316682153784  
 source_magnitude                 | 0.9757983146973724  
 source_distance_deg              | 0.4359214473594007  
 source_distance_km               | 48.43598571849436   
 back_azimuth_deg                 | 102.4214576404989   
 coda_end_sample                  | 1143.584651891823   
```

</details>

### Data Distribution

<img width="2488" height="1989" alt="Numerical data distribution" src="https://github.com/user-attachments/assets/1db3a568-7f5d-49e6-98ba-851c5b7d32ab" />

Most numerical metadata variables showed some degree of skew. This informed our decision to use median imputation rather than mean imputation for missing numerical values.

### Categorical Frequency Analysis

<img width="2388" height="1189" alt="Top 10 categorical values" src="https://github.com/user-attachments/assets/1b8305f9-4d2c-4378-9158-22c7baf929dd" />

### Waveform Scale Statistics

After reshaping waveform vectors back to `(6000, 3)`, we computed statistics for the three directional components.

```text
--- Waveform Scale Statistics ---
Component 1 (East):
  Mean: -2.302661
  Std Dev: 32890.795317
  Min: -7183533.000000
  Max: 7503966.000000

Component 2 (North):
  Mean: 1.713306
  Std Dev: 23582.819392
  Min: -3771832.750000
  Max: 2287881.500000

Component 3 (Vertical):
  Mean: -1.299870
  Std Dev: 10819.480801
  Min: -592841.187500
  Max: 651094.187500
```

The waveform mean centered near zero, which is expected for seismic amplitude data. The waveform column is quantitative and was flattened to shape `(1, 18000)` for the combined Parquet dataset.

### Target Variable

The target category column is `trace_category`.

| Label | Count |
|---|---:|
| `earthquake_local` | 1,027,574 |
| `noise` | 235,426 |
| `NULL` | 5,314 |

For binary classification framing, the positive class is `earthquake_local` and the negative class is `noise`. The 5,314 rows with missing category labels were dropped.

### Missing and Duplicate Values

The dataset did not contain duplicate `trace_name` values, which is expected because `trace_name` is a unique identifier. We also did not find duplicate rows. Duplicate values within other columns are expected because many traces can share station, source, or geographic metadata.

Missing values were handled according to the field type and modeling purpose:

- Rows with missing target labels were dropped.
- Numerical metadata variables with missing values were imputed using the median because the distributions were skewed.
- Identifier fields such as `trace_name` and `source_id` were treated as identifiers rather than predictive numerical features.
- Columns with redundancy or poor predictive utility were removed during preprocessing.

### Correlation and Redundancy Analysis

<img width="1558" height="1387" alt="Covariance matrix" src="https://github.com/user-attachments/assets/79994546-0ded-4a92-9c05-b606fcef796e" />

The covariance matrix suggested two cases of redundancy:

1. `source_distance_deg` and `source_distance_km` were nearly redundant because they represent distance in different units.
2. `receiver_latitude` and `source_latitude` showed a relationship that required further inspection.

Based on the distribution plots and covariance analysis, we dropped `source_distance_deg` and `receiver_latitude` from later modeling steps.

### Example Waveforms

Example earthquake waveform:

**Trace name:** `A16.CN_20150121053158_EV`

<img width="790" height="471" alt="Example earthquake waveform" src="https://github.com/user-attachments/assets/4e2a0850-d16b-4e0b-b6d1-fe904f7fb9b1" />

Example noise waveform:

**Trace name:** `109C.TA_201510210555_NO`

<img width="790" height="495" alt="Example noise waveform" src="https://github.com/user-attachments/assets/c9ea2eeb-8a88-493a-845b-0da3015f37df" />

The waveform plots provide visual evidence that earthquake and noise traces have distinguishable signal patterns that are not obvious from the raw flattened array values alone.

---

## Preprocessing

The preprocessing pipeline converted raw seismic files into a distributed modeling dataset.

### Main Preprocessing Steps

1. Loaded the metadata from `merge.csv` using Spark.
2. Loaded waveform data from `merge.hdf5`.
3. Joined waveform observations with metadata using trace identifiers.
4. Flattened each `(6000, 3)` waveform into a vector of length `18,000`.
5. Dropped rows with missing target labels.
6. Imputed missing numerical metadata values using median imputation.
7. Removed redundant or low-utility columns such as `source_distance_deg` and `receiver_latitude`.
8. Saved the combined result as `stead_combined.parquet` for faster downstream loading.

### Spark Configuration Used for Parquet Processing

```python
spark = (
    SparkSession.builder
    .config("spark.driver.memory", "4g")
    .config("spark.executor.instances", "6")
    .config("spark.executor.memory", "20g")
    .getOrCreate()
)
```

---

## Model 1: Baseline Distributed Regression Models

The first modeling stage predicted `s_arrival_sample` using distributed regression models.

### Models Tested

| Model | Key hyperparameters | Target |
|---|---|---|
| Random Forest Regressor | `num_trees=20`, `max_depth=5` | `s_arrival_sample` |
| Random Forest Regressor | `num_trees=40`, `max_depth=3` | `s_arrival_sample` |
| XGBoost Regressor | `max_depth=8`, `eta=0.1` | `s_arrival_sample` |

### Baseline Modeling Goal

The baseline model was used to determine whether distributed tree-based models could learn meaningful relationships from the waveform-derived feature representation. It also provided a comparison point for the final SVD and PhaseNet approach.

---

## Model 2: SVD Dimensionality Reduction and PhaseNet

The final modeling stage used dimensionality reduction followed by a seismic deep learning model.

### Dimensionality Reduction with SVD

We applied Singular Value Decomposition (SVD) to the three-channel waveform data. In seismology, applying SVD to 3-channel waveform data is related to polarization filtering. The first principal component captures the dominant direction of particle motion, while lower components can be interpreted as less dominant signal structure or background noise.

For each waveform matrix, SVD decomposes the data into directional and temporal components. The rank-1 reconstruction uses the dominant component:

```text
Waveform matrix ≈ U₁ S₁ V₁ᵀ
```

Where:

- `U₁` is a single column vector with shape `(6000, 1)` and represents the dominant or “master” waveform.
- `S₁` is the scalar singular value for the dominant component.
- `V₁ᵀ` is a single row vector with shape `(1, 3)` and represents directional weights across the three seismic channels.

<img width="304" height="48" alt="SVD equation" src="https://github.com/user-attachments/assets/64effe61-da45-4216-b42c-ea5795c20461" />

<img width="688" height="94" alt="Rank-one SVD reconstruction" src="https://github.com/user-attachments/assets/12b16e95-4052-4fdf-b913-493384758eea" />

This reconstruction projects the 3-dimensional seismic channel space onto a 1-dimensional motion direction.

<img width="1464" height="690" alt="SVD directional projection visualization" src="https://github.com/user-attachments/assets/47387fd6-3c7d-44a7-8782-cb07b21203ba" />

### SVD Visualization Interpretation

In the left plot, darker colors represent the beginning of the time window, green colors represent the approximate middle of the time window near the S-wave arrival, and yellow colors represent the later part of the time window.

In the right plot, darker colors represent motion before the main S-wave energy arrives, pink/orange colors represent the approximate S-wave arrival, and yellow colors represent later wave motion after the main arrival.

<img width="1389" height="790" alt="S-wave amplitude visualization" src="https://github.com/user-attachments/assets/63199fd0-8c0f-4a26-9dbb-dc58d8dd7120" />

The visualization shows that amplitude increases substantially after the initial wave arrival, supporting the interpretation that later S-wave motion is more impactful than the earlier P-wave arrival.

### PhaseNet Final Model

For the final model, we implemented PhaseNet, a deep neural network designed for seismic wave arrival picking.

We trained and evaluated two versions of the model:

1. PhaseNet using SVD-processed waveform data.
2. PhaseNet using waveform data without SVD preprocessing.

The model was trained on a subsample of 80,000 rows, equal to approximately 7.77% of the full dataset. Since each row contains 18,000 waveform features, this subset corresponds to approximately:

```text
80,000 rows × 18,000 waveform features = 1,440,000,000 waveform feature values
```

---

# Results

## Exploratory Data Results

The exploratory analysis showed that:

- The dataset is large enough to require distributed processing.
- Metadata features contain skewed distributions, motivating median imputation.
- The target labels are imbalanced, with more earthquake observations than noise observations.
- Some metadata fields are redundant, especially distance variables expressed in different units.
- Waveform plots show visible differences between earthquake and noise traces.

---

## Model 1 Results

### Baseline Model Performance

| Model | Train RMSE | Test RMSE | Interpretation |
|---|---:|---:|---|
| Random Forest Regressor, `num_trees=20`, `max_depth=5` | 84.71 | 87.47 | Best baseline result; train/test scores are close |
| Random Forest Regressor, `num_trees=40`, `max_depth=3` | Not reported | 93.55 | Higher test error; likely more underfit |
| XGBoost Regressor, `max_depth=8`, `eta=0.1` | Not reported | 431.36 | Severe overfitting or failed generalization |

The best baseline model was the Random Forest Regressor with `num_trees=20` and `max_depth=5`. Its train and test RMSE values were close, suggesting that it did not severely overfit. However, the RMSE was still large enough to indicate underfitting.

Because the data was downsampled to 20 Hz, each sample corresponds to 0.05 seconds. Therefore, the best baseline test RMSE corresponds to approximately:

```text
87.47 samples × 0.05 seconds/sample = 4.37 seconds
```

A 4.37-second average offset is meaningful for seismic arrival prediction, so the baseline model was not accurate enough for a high-quality arrival picking system.

### Baseline Model Figures

Random Forest Regressor with `num_trees=20` and `max_depth=5`:

<img width="1590" height="590" alt="Random Forest 20 trees max depth 5 test statistics" src="https://github.com/user-attachments/assets/12ced923-a516-4b79-a219-102353e7d5c2" />

Random Forest Regressor with `num_trees=40` and `max_depth=3`:

<img width="1590" height="590" alt="Random Forest 40 trees max depth 3 test statistics" src="https://github.com/user-attachments/assets/003329ba-648c-4477-8455-3e827f1f0085" />

The left graph in both figures shows two visible clusters. The `num_trees=20`, `max_depth=5` model produced more varied predictions, while the `num_trees=40`, `max_depth=3` model showed a narrower cluster around predicted S-wave sample values above 200. However, the second model had a higher test RMSE, suggesting that the narrower predictions did not improve accuracy.

---

## Model 2 Results

### PhaseNet Prediction Example

```text
Predicted P Wave Arrival Time: 8.05s (Index: 805)
Predicted S Wave Arrival Time: 10.02s (Index: 1002)
```

### PhaseNet Evaluation

| Model version | Final training loss | Final average test loss | Interpretation |
|---|---:|---:|---|
| PhaseNet with SVD | 0.01585 | 0.01640 | Slight generalization gap; SVD may have removed useful waveform detail |
| PhaseNet without SVD | 0.01285 | 0.01275 | Best final result; test loss is slightly lower than training loss |

The PhaseNet model without SVD achieved the best final test loss. This suggests that although SVD is useful for dimensionality reduction and denoising, the rank-reduced representation may remove information that PhaseNet can otherwise learn directly from the raw waveform channels.

---

#### Prediction Sample

```text
Predicted P Wave Arrival Time: 8.05s (Index: 805)
Predicted S Wave Arrival Time: 10.02s (Index: 1002)
```

---

# Discussion

The project began with the assumption that large-scale waveform data could support useful prediction of seismic wave arrival behavior.
The baseline Random Forest Regressor was a useful first model because it established a distributed machine learning benchmark for predicting `s_arrival_sample`. Its train and test RMSE values were close, which indicates that the model did not suffer from severe overfitting. However, the best baseline RMSE corresponded to an average timing error of approximately 4.37 seconds. For seismic arrival prediction, this is too large to be considered highly accurate. Therefore, the baseline model fits in the underfitting region of the fitting graph: it generalizes similarly across train and test sets, but its total error remains too high.

The XGBoost baseline performed much worse on the test set, with a test RMSE of 431.36. This suggests severe overfitting, failed generalization, or a mismatch between the model configuration and feature representation. In contrast, the Random Forest model was more stable but not expressive enough to capture the full structure of waveform signals.

The final model moved toward a more domain-specific approach. SVD was introduced to reduce dimensionality and isolate dominant directional waveform motion. This was useful for visualization and interpretation, and it provided a scientifically meaningful way to analyze three-channel waveform behavior. However, the PhaseNet comparison showed that the model without SVD achieved lower test loss than the model with SVD. This suggests that dimensionality reduction helped interpretation but may have removed high-frequency or multi-channel details that PhaseNet could use for arrival picking.

The results are believable because the best deep-learning result came from the representation that preserved the most waveform information. However, the reported PhaseNet evaluation currently uses a subsample of 80,000 rows rather than the full dataset. This means the final result should be interpreted as a promising experiment rather than a fully optimized production-level model.

Important shortcomings include:

- The final PhaseNet model was trained on only 7.77% of the dataset.
- The README does not yet include correct, false-positive, and false-negative prediction examples.
- The Spark vs. Ray framework comparison does not yet include exact runtime, memory, and lines-of-code measurements.
- The baseline model was affected by downsampling, which reduced feature dimensionality but also likely removed useful waveform detail.
- Additional seismic feature engineering, such as STA/LTA features, was not fully explored.

Future improvements should include training PhaseNet on a larger subset or the full dataset, reducing the amount of downsampling, adding seismic-specific engineered features, and reporting more detailed prediction examples and timing benchmarks.

---

# Conclusion

This project demonstrated that distributed computing is essential for large-scale seismic waveform analysis. Spark enabled us to load, join, preprocess, and store waveform data that would be impractical to process on a single machine. Ray provided a simpler interface for some model-training workflows and may be better suited for flexible hyperparameter search.

The best baseline model was the Random Forest Regressor with `num_trees=20` and `max_depth=5`, which achieved a test RMSE of 87.47 samples. Although this model generalized better than the XGBoost baseline, the corresponding timing error of approximately 4.37 seconds showed that the baseline was underfit.

The final PhaseNet model achieved stronger results than the baseline approach. Comparing PhaseNet with and without SVD showed that the model without SVD produced the lowest average test loss. This suggests that SVD was helpful for dimensionality reduction and interpretation, but the raw waveform representation preserved important signal details that the neural network could learn directly.

With more time and resources, we would train PhaseNet on a larger portion of the dataset, tune the architecture and learning parameters, evaluate predictions with correct/FP/FN examples, and complete a rigorous Spark vs. Ray timing comparison. We would also explore seismic feature engineering methods such as STA/LTA and compare those features against purely learned waveform representations.

Overall, this project showed how distributed computing changes the modeling workflow: instead of designing models around what can fit on one machine, we were able to design a pipeline around the full structure and scale of the seismic waveform data.

---

## Related Work

Zhu, Weiqiang, and Gregory C. Beroza. “PhaseNet: A Deep-Neural-Network-Based Seismic Arrival Time Picking Method.” *arXiv preprint arXiv:1803.03211* (2018).

Reference implementation used for model development:

- <https://github.com/AI4EPS/PhaseNet>

---

# Statement of Collaboration

> Replace the placeholder rows below with each group member’s actual contribution before final submission.

| Name | Title / Role | Contribution |
|---|---|---|
| TODO | TODO | TODO |
| TODO | TODO | TODO |
| TODO | TODO | TODO |

Required format from the assignment:

```text
Name: Title: Contribution
```

If a group member did not participate, write:

```text
Name: Did not participate in the project.
```

---

# Extra Credit: Spark vs. Ray Framework Comparison

We chose Option C: training an XGBoost model on a subset of the data using both Spark and Ray.

The Spark implementation is available in:

```text
xgboost_test_7.30.ipynb
```

The Ray implementation is available in:

```text
ray xgboost.ipynb
```

## Spark Implementation

```python
from pyspark.ml.feature import VectorAssembler
from pyspark.ml import Pipeline
from xgboost.spark import SparkXGBRegressor

# Assemble features
assembler = VectorAssembler(
    inputCols=["vec_N", "vec_E", "vec_Z"],
    outputCol="features"
)

# Distributed XGBoost
# num_workers should match the number of Spark executors.
xgb = SparkXGBRegressor(
    features_col="features",
    label_col=label_col,
    prediction_col="prediction",
    num_workers=3,
    max_depth=8,
    eta=0.1,
    objective="reg:squarederror",
    eval_metric="rmse"
)

pipeline = Pipeline(stages=[assembler, xgb])

# Train distributed model
model = pipeline.fit(train_df)
```

## Ray Implementation

```python
base_params = {
    "objective": "reg:squarederror",
    "tree_method": "hist",
    "eval_metric": "rmse",
}

max_depth_list = [3, 5, 7]
eta_list = [0.01, 0.05, 0.1]
subsample_list = [0.7, 1.0]

search_results = []

for max_depth in max_depth_list:
    for eta in eta_list:
        for subsample in subsample_list:
            params = {
                **base_params,
                "max_depth": max_depth,
                "eta": eta,
                "subsample": subsample,
            }

            print(
                f"Training with params: max_depth={max_depth}, "
                f"eta={eta}, subsample={subsample}"
            )

            trainer = XGBoostTrainer(
                label_column=LABEL_COL,
                params=params,
                scaling_config=ScalingConfig(num_workers=4, use_gpu=False),
                datasets={"train": train_ds, "valid": valid_ds},
                num_boost_round=200,
            )

            result = trainer.fit()
            valid_rmse = result.metrics.get("valid-rmse", None)

            search_results.append({
                "max_depth": max_depth,
                "eta": eta,
                "subsample": subsample,
                "valid_rmse": valid_rmse,
            })
```

## Performance Comparison

The assignment asks for average runtime over three runs, lines of code, and peak memory usage. The current project notes indicate that Ray appeared faster and easier to implement, but exact comparison values are not yet available.

| Framework | Execution time, average of 3 runs | Lines of code | Peak memory usage | Notes |
|---|---:|---:|---:|---|
| Spark | TODO | TODO | TODO | Distributed XGBoost using Spark pipeline |
| Ray | TODO | TODO | TODO | Ray XGBoostTrainer with hyperparameter search |

## Framework Analysis

Based on the current implementation experience, Ray was easier to implement because the training workflow required a lighter wrapper around XGBoost-style configuration, while Spark required more Spark-specific feature assembly and pipeline setup. For this project, Ray may be preferable for flexible model training and hyperparameter search, while Spark remains valuable for large-scale preprocessing and Parquet-based data handling.

To complete this section, add exact timing results from three repeated runs for both frameworks.














## Extra Credit
1. implementation
We chose option C. The XGBoost implementaion in Ray will be availble in `ray xgboost.ipynb` and Spark implementation will be available in `xgboost_test_7.30.ipynb`.

2.
Spark implementation
```python
from pyspark.ml.feature import VectorAssembler
from pyspark.ml import Pipeline
from xgboost.spark import SparkXGBRegressor

# ---------------------------
# Assemble features
# ---------------------------
assembler = VectorAssembler(
    inputCols=["vec_N", "vec_E", "vec_Z"],
    outputCol="features"
)

# ---------------------------
# Distributed XGBoost
# num_workers MUST match your Spark executors
# You have 3 executors → num_workers=3
# ---------------------------
xgb = SparkXGBRegressor(
    features_col="features",
    label_col=label_col,
    prediction_col="prediction",
    num_workers=3,
    max_depth=8,
    eta=0.1,
    objective="reg:squarederror",
    eval_metric="rmse"
)

pipeline = Pipeline(stages=[assembler, xgb])

# ---------------------------
# Train distributed model
# ---------------------------
model = pipeline.fit(train_df)```

Ray implementation

```python
# 9. HYPERPARAMETER SEARCH
base_params = {
    "objective": "reg:squarederror",
    "tree_method": "hist",
    "eval_metric": "rmse",
}

max_depth_list = [3, 5, 7]
eta_list = [0.01, 0.05, 0.1]
subsample_list = [0.7, 1.0]

search_results = []

for max_depth in max_depth_list:
    for eta in eta_list:
        for subsample in subsample_list:

            params = {
                **base_params,
                "max_depth": max_depth,
                "eta": eta,
                "subsample": subsample,
            }

            print(f"Training with params: max_depth={max_depth}, eta={eta}, subsample={subsample}")

            trainer = XGBoostTrainer(
                label_column=LABEL_COL,
                params=params,
                scaling_config=ScalingConfig(num_workers=4, use_gpu=False),
                datasets={"train": train_ds, "valid": valid_ds},
                num_boost_round=200,
            )

            result = trainer.fit()

            # Ray stores metrics like "valid-rmse"
            valid_rmse = result.metrics.get("valid-rmse", None)

            search_results.append({
                "max_depth": max_depth,
                "eta": eta,
                "subsample": subsample,
                "valid_rmse": valid_rmse,
            })
```
3.
- We argue that Ray was faster than Spark. We are not sure by how much due to missing comparision code.
- To us, ray was easier to implement since most of code just needed a simple wrapper compared spark where it needs its own spark code.
- For our specific use case, we will choose Ray due to easy implementation but with some configuration to suppress unnecessary print outs. 
