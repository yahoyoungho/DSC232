# DSC232
Link to dataset: https://github.com/smousavi05/STEAD

---

### 3. Fitting Analysis (4 points)

Answer the following questions:

- Where does your model fit in the fitting graph (underfitting vs. overfitting)?
- Build at least one model with **different hyperparameters** and compare results
- Which model performs best and why?
- What are the next models you are thinking of for Milestone 4 and why?

---

### 4. Conclusion Section (5 points)

Write a conclusion for your first model:

- What is the conclusion of your 1st model?
- What can be done to possibly improve it?
- How did distributed computing help with this task?

### 5. Speedup Analysis (5 points)

---

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
