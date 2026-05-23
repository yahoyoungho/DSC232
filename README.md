# DSC232
Link to original dataset: https://github.com/smousavi05/STEAD

# Important TODO for file loading
- from terminal execute `ls /scratch/$(whoami)` and get the session job name
- copy data from network directory to scratch directory by executing `cp ~/ysuh2/data/stead_combined.parquet /expanse/lustre/scratch/$(whoami)/{jobname}` or `cp ~/ysuh2/data/stead_combined.parquet /scratch/$(whoami)/{jobname}`

<i>We have a parquet file generated form the given csv and hdf5 file. The generated data will be in `ysuh2/data/stead_combined.parquet`</i>

## SDSC Expanse Spark Environment

### Resource Request
- Total Cores: 8
- Total Memory: 128 GB
- Driver Memory: 4 GB

### Executor Calculation for hdf5 data
- Executor Instances = 8 − 1 = 7
- Executor Memory = (128 − 4) // 7 = 17 GB

### Executor Calculation for metadata file
Since the file size is too small, we decided that calculating the executor/driver memory is unnecessary.
- CSV file is around 400Mb

### Spark Configuration
# for memory of 128gb 8 cores
```python
# for converted parquet file configuration
spark = (
    SparkSession.builder
    .config("spark.driver.memory", "4g")
    .config("spark.executor.instances", "6")
    .config("spark.executor.memory", "20g")
    .getOrCreate()
)
```

```python
# for metadata EDA configuration
spark = (
    SparkSession.builder
    .config("spark.driver.memory", "1g")
    .config("spark.executor.instances", "1")
    .config("spark.executor.memory", "500m")
    .getOrCreate()
)
```



### Spark UI Evidence
<img width="730" height="365" alt="image" src="https://github.com/user-attachments/assets/d6d319cd-1ffb-48d7-8f20-3e30f674840c" />
Initial setup was to have the driver memory to be 2GB, but due to large dataset we had to increase the driver memory to be 4GB.
For the executor memory, we noticed that the execution for 6 executors with 20Gb configuration was faster than 7 executors with 17Gb.
- Refering to `hdf5_2_parquet.ipynb`'s last cell

<img width="725" height="360" alt="image" src="https://github.com/user-attachments/assets/9efb94fb-05d1-4c80-ad0c-37b8a89a770c" />
- For the metadata EDA, we only allocated 1gb to Driver and 500mb to the single executor for the approximately 0.5GB metadata file.


## How many observations does the dataset have?
We have three datasets in total: (merge.csv, merge.hdf5, stead_combined.parquet)\
The merge.csv metadata file contains total 1,268,314 observations, each representing a unique earthquake event.\
The merge.hdf5 file correspondingly contains 1,268,314 rows of tuples, each tuple containing three elements.
- The first element of the tuple is the `trace_name` of the corresponding earthquake.
- The second element is the earthquake's waveform. Each waveform is an array in the shape of (6000, 3).
- The third element is a dictionary of three key-value pairs. The keys are `p_arrival_sample`, `s_arrival_sample`, and `coda_end_sample`. The values are strings containing numerical values.
- Pre-missing data handling, we would effectively deal with 7,609,884,000 (over 7 billion) rows of data (1,268,314 earthquakes * 6,000 waveform samples = 7,609,884,000).
- After dropping 5,314 null earthquake ids, we are effectively working with 7,578,000,000 rows of data.

We combined the merge.csv and merge.hdf5 files to create the stead_combined.parquet dataset.
- It contains 1,265,657 observations and retains the same schema as the original files **except** the waveform data has been flattened.

## EDA

Scale of numerical variables from metadata file (merge.csv dataset)

```RECORD 0-----------------------------------------------
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
-RECORD 1-----------------------------------------------
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
-RECORD 2-----------------------------------------------
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
-RECORD 3-----------------------------------------------
 summary                          | min                 
 receiver_latitude                | -77.8492            
 receiver_longitude               | -178.8566           
 receiver_elevation_m             | -2920.5             
 p_arrival_sample                 | 12.0                
 p_weight                         | 0.0                 
 p_travel_sec                     | 0.0                 
 s_arrival_sample                 | 190.0               
 s_weight                         | 0.0                 
 source_origin_uncertainty_sec    | 0.0                 
 source_latitude                  | -43.8455            
 source_longitude                 | -179.9965           
 source_error_sec                 | 0.0                 
 source_gap_deg                   | 5.282               
 source_horizontal_uncertainty_km | 0.0                 
 source_depth_km                  | -3.49               
 source_depth_uncertainty_km      | 0.0                 
 source_magnitude                 | -0.5                
 source_distance_deg              | 0.0                 
 source_distance_km               | 0.0                 
 back_azimuth_deg                 | 0.0                 
 coda_end_sample                  | 621.0               
-RECORD 4-----------------------------------------------
 summary                          | 25%                 
 receiver_latitude                | 33.61157            
 receiver_longitude               | -122.795303         
 receiver_elevation_m             | 410.0               
 p_arrival_sample                 | 500.0               
 p_weight                         | 0.59                
 p_travel_sec                     | 3.569999933242798   
 s_arrival_sample                 | 924.0               
 s_weight                         | 0.55                
 source_origin_uncertainty_sec    | 0.6                 
 source_latitude                  | 33.7831667          
 source_longitude                 | -124.3933           
 source_error_sec                 | 0.11                
 source_gap_deg                   | 55.0                
 source_horizontal_uncertainty_km | 0.0                 
 source_depth_km                  | 4.13                
 source_depth_uncertainty_km      | 0.39                
 source_magnitude                 | 0.8                 
 source_distance_deg              | 0.151               
 source_distance_km               | 16.79               
 back_azimuth_deg                 | 107.3               
 coda_end_sample                  | 1691.0              
-RECORD 5-----------------------------------------------
 summary                          | 50%                 
 receiver_latitude                | 37.540531           
 receiver_longitude               | -118.7564           
 receiver_elevation_m             | 939.0               
 p_arrival_sample                 | 699.0               
 p_weight                         | 0.67                
 p_travel_sec                     | 7.269999980926514   
 s_arrival_sample                 | 1207.0              
 s_weight                         | 0.62                
 source_origin_uncertainty_sec    | 0.86                
 source_latitude                  | 37.5555             
 source_longitude                 | -118.84717          
 source_error_sec                 | 0.21                
 source_gap_deg                   | 91.0                
 source_horizontal_uncertainty_km | 0.36                
 source_depth_km                  | 8.47                
 source_depth_uncertainty_km      | 0.63                
 source_magnitude                 | 1.3                 
 source_distance_deg              | 0.35                
 source_distance_km               | 38.98               
 back_azimuth_deg                 | 182.0               
 coda_end_sample                  | 2273.0              
-RECORD 6-----------------------------------------------
 summary                          | 75%                 
 receiver_latitude                | 43.986              
 receiver_longitude               | -116.45637          
 receiver_elevation_m             | 1388.0              
 p_arrival_sample                 | 800.0               
 p_weight                         | 0.92                
 p_travel_sec                     | 12.529999732971191  
 s_arrival_sample                 | 1615.0              
 s_weight                         | 0.85                
 source_origin_uncertainty_sec    | 1.12                
 source_latitude                  | 45.6992             
 source_longitude                 | -116.503            
 source_error_sec                 | 0.62                
 source_gap_deg                   | 141.0               
 source_horizontal_uncertainty_km | 2.04142             
 source_depth_km                  | 14.22               
 source_depth_uncertainty_km      | 1.28                
 source_magnitude                 | 2.0                 
 source_distance_deg              | 0.636               
 source_distance_km               | 70.75               
 back_azimuth_deg                 | 284.2               
 coda_end_sample                  | 3100.0              
-RECORD 7-----------------------------------------------
 summary                          | max                 
 receiver_latitude                | 359.9               
 receiver_longitude               | 179.6277            
 receiver_elevation_m             | 4580.0              
 p_arrival_sample                 | 2877.0              
 p_weight                         | 1.0                 
 p_travel_sec                     | 57.18000030517578   
 s_arrival_sample                 | 5644.0              
 s_weight                         | 1.0                 
 source_origin_uncertainty_sec    | 999.0               
 source_latitude                  | 78.3804             
 source_longitude                 | 179.9972            
 source_error_sec                 | 29.33               
 source_gap_deg                   | 360.0               
 source_horizontal_uncertainty_km | 10.0                
 source_depth_km                  | 341.74              
 source_depth_uncertainty_km      | 15.0                
 source_magnitude                 | 7.9                 
 source_distance_deg              | 3.0                 
 source_distance_km               | 346.27              
 back_azimuth_deg                 | 360.0               
 coda_end_sample                  | 6000.0              
 ```


Numerical data distribution
<img width="2488" height="1989" alt="image" src="https://github.com/user-attachments/assets/1db3a568-7f5d-49e6-98ba-851c5b7d32ab" />

From the distribution tables, we can see that most have at least some degree of skew. This helped us decide to impute missing values with the median rather than the mean.

Categorical top 10 frequent values
<img width="2388" height="1189" alt="image" src="https://github.com/user-attachments/assets/1b8305f9-4d2c-4378-9158-22c7baf929dd" />


Sample Waveform Statistics

```
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
After reshaping back to (6000,3) to get directional vectors, we got these `waveform_data` statistics in the hdf5_2_parquet.ipynb. The `waveform_data` column is quantitative, and is in the shape of (1,18000).
The mean centers around 0 as expected from looking at the waveform amplitude plot (refer to data plot section of readme.md).

Our target column is `trace_category` from the merge.csv metadata file. The positive label will be "earthquake_local" and the negative label will be "noise". 5314 rows were missing and dropped.

```
+----------------+-------+
|  trace_category|  count|
+----------------+-------+
|earthquake_local|1027574|
|           noise| 235426|
|            NULL|   5314|
+----------------+-------+
```

## Missing and duplicate Values
In our dataset there are no duplicate value in `trace_name` as it is a unique identifier. To confirm we did a simple calculation on `trace_name` column. (Refer to cell 12)

Our dataset does not contain duplicate rows. In regards to duplicates values within the columns themselves,  source_id and trace_name are the only ones that should not have any since they are unique identifier fields. Any duplicate values that appear in the other variables are either expected or meaningful to the data (with the exception of `source_latitude` which we later decide to remove).

**Readme needs to be updated with missing values description**

## Data Plots

<img width="1558" height="1387" alt="image" src="https://github.com/user-attachments/assets/79994546-0ded-4a92-9c05-b606fcef796e" />

From this covariance matrix, we can see two possible instances of collinearity/redundant variables.\
First is between `source_distance_deg` and `source_distance_km`, since they are essentially the same thing represented in two different metrics.\
Second is between `receiver_latitude` and `source_latitude`. Upon further inspection and based on the data distribution graphs, `receiver_latitude` is not a useful predictor variable due to duplicate data values.\
The duplicate data values makes sense since the location(s) that was receiving the earthquake signals was likely the same.
Thus, we will drop `source_distance_deg` and `receiver_latitude` moving forward.


Example Earthquake waveform plot
trace name: A16.CN_20150121053158_EV
<img width="790" height="471" alt="image" src="https://github.com/user-attachments/assets/4e2a0850-d16b-4e0b-b6d1-fe904f7fb9b1" />

Example noise waveform plot
trace name: 109C.TA_201510210555_NO
<img width="790" height="495" alt="image" src="https://github.com/user-attachments/assets/c9ea2eeb-8a88-493a-845b-0da3015f37df" />

Just looking at the raw waveform array data does not garner any apparent information, so these waveform plots help visually distinguish the difference in values between "noise" waveforms and "earthquake" waveforms.

---

## Fitting Analysis

- Our RF Regressor model is underfitting, while our XGBoost Regressor was severely overfitting.
- We have one Random Forest Regressor model with 20 `num_trees` and `max_depth` 5.
- Another RF Regressor model with 40 `num_trees` and `max_depth` 3.
- For XGBoost Regressor we had the hyperparameter set as `max_depth` = 8 with `eta` = 0.1 .
- as a result, our baseline model with 20 trees with max_depth 5 had a better performance 
    - RF Regressor max_depth = 5; 20 trees: test RMSE: 87.47; train RMSE: 84.71
    - RF Regressor max_depth = 3; 40 trees: test RMSE: 93.55
    - XGBoost Regressor max_depth = 8; eta: 0.1; test RMSE: 431.35559602907716

- RF Regressor num_trees: 20; max_depth: 5 test statistics graph
<img width="1590" height="590" alt="image" src="https://github.com/user-attachments/assets/12ced923-a516-4b79-a219-102353e7d5c2" />

- RF Regressor num_trees: 40; max_depth: 3 test statistics graph
<img width="1590" height="590" alt="image" src="https://github.com/user-attachments/assets/003329ba-648c-4477-8455-3e827f1f0085" />

-  The left graph of both models reveals two clusters. The graph of RF Regressor num_trees: 20; max_depth: 5 has more spread out clusters, indicating that it's predictions were more varied than that of num_trees: 40; max_depth: 3. The cluster that hovers above predicted s-wave sample value of 200 is very narrow for the latter model, but that model has a slightly higher test RMSE, suggesting that it did not quite narrow in on the correct value.

    - **RMSE Interpretation**: The original data is collected in 100hz. However, we down sampled the data to 20hz instead. Due to do this, our evaluation for RF Regressor max_depth = 5; 20 trees will be having a offset of 87.47 * 0.05 = around 4.37 seconds from the actual `s_arrival time`.
- Which model performs best and why?
    - as of now, our Random Forest Regressor with max_depth = 5 and 20 trees is showing the best performance as it is showing the lowest test RMSE score while the training and testing has a similar score as well.
- What are the next models you are thinking of for Milestone 4 and why?
    - As our data is a time series heavy dataset, we might be looking into some time series applicable models such as GRU cells or a model named PhaseNet.
    - When we were still predicting `trace_category` rather than `s_arrival_sample`, we were running into a lot of data leakage issues with the model learning from intentional null values, and we were strongly considering Linear ODE for that scenario. We may look into if it still makes sense to use Linear ODE for our current setup.

---

## Conclusion Section

Our Random Forest Regressor model demonstrated the most reliable performance among all evaluated models, achieving test and train RMSEs of very close values, avoiding overfitting. However, the test RMSE of 87.47 samples, which is equivalent to an average time offset of 4.37 seconds (87.47 x 0.05s) from the true S-wave arrival time is indicative of underfitting. In seismology, especially considering the downsampling that we did, a 4.37 second window is not small, so our current model is not complex enough to fully capture the high-frequency characteristics embedded in the waveforms.

To improve it, we could have reduced how much we downsampled by. i.e. by 50Hz instead of 20Hz or keeping it at the original 100Hz. We only did this to try and reduce dimensionality. Additionally, we could have tried to increase the maxDepth and numTrees hyperparameters to capture more nuanced relationships within the data. Lastly, there are numerous types of feature engineering methods specific to seismic data that we found through research, such as STA/LTA (Short-Term Average / Long-Term Average).

The speedup analysis indicates that we weren't able to optimize well enough, thus not making full use of distributed computing to help is in this task. There is the possibility that the bottleneck we are clearly experiencing has something to do with the the way MLLib is performing this.
