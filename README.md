# DSC232
Link to dataset: https://github.com/smousavi05/STEAD

# Distributed XGBoost Regression on STEAD Waveforms (PySpark)

Train a **distributed XGBoost regressor** with **PySpark + xgboost.spark** to predict **S-wave arrival sample (`s_arrival_sample`)** from three-component seismic waveform data (`waveform_N`, `waveform_Z`, `waveform_E`) from the **STEAD v4** dataset. 
---

## Project Overview

This project loads STEAD waveform traces from a Parquet directory, filters to `trace_category == 'earthquake_local'`, converts waveform arrays into Spark ML vectors, assembles them into a single feature vector, and trains a distributed **SparkXGBRegressor** inside a Spark ML pipeline.
### Key steps
- Start a Spark session with custom driver/executor memory and 3 executors 
- Read STEAD Parquet data and filter local earthquakes 
- Cast label column `s_arrival_sample` to `double`  
- Convert waveform arrays → vectors (`array_to_vector`)   
- Sample a small subset and split into train/val/test 
- Train distributed XGBoost regression with `num_workers=3` 
- Evaluate RMSE and MAE + print example predictions 

---

## Data

Expected Parquet directory:

```python
parquet_path = "/expanse/lustre/projects/uci157/ysuh2/data/stead_version4"
