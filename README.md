# DSC232

## GitHub Repository Setup (2 points)
Link to dataset: https://github.com/smousavi05/STEAD
## SDSC Expanse Spark Environment

### Resource Request
- Total Cores: 8
- Total Memory: 128 GB
- Driver Memory: 2 GB

### Executor Calculation
- Executor Instances = 8 − 1 = 7
- Executor Memory = (128 − 2) / 7 = 18 GB

### Spark Configuration
# for memory of 128gb 8 cores
spark = (
    SparkSession.builder
    .appName("young-job")
    .config("spark.driver.memory", "2g")
    .config("spark.executor.instances", "7")
    .config("spark.executor.memory", "18g")
    .getOrCreate()
)

df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("data/merge.csv")
)


### Spark UI Evidence
<insert screenshot showing multiple executors> <img width="408" height="291" alt="setup" src="https://github.com/user-attachments/assets/543e353b-ece7-4fa1-a2f4-85affa25cddc" />
This was recommended in the best practices files. Therefore, we decided to stick with the 128 GB. 

## How many observations does your dataset have?
The merge.csv metadata file contains total 1268314 observations.\
The merge.hdf5 file contains the waveforms of each earthquake observation.\
Each waveform is in an array in the shape of (6000, 3).\
Pre-missing data handling, we would effectively deal with 7609884000 (over 7 billion) rows of data.\
After dropping 5314 null earthquake ids, we are effectively working with 7578000000 rows of data.

## Describe all columns in your dataset: their scales and data distributions. Describe categorical and continuous variables. Describe your target column.

Scale of numerical variables from metadata file

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

Categorical top 10 frequent values
<img width="2388" height="1189" alt="image" src="https://github.com/user-attachments/assets/1b8305f9-4d2c-4378-9158-22c7baf929dd" />

Our target column is `trace_category`. The positive label will be `earthquake_local` and the negative label will be `noise`. 5314 rows were missing and dropped.

```
+----------------+-------+
|  trace_category|  count|
+----------------+-------+
|earthquake_local|1027574|
|           noise| 235426|
|            NULL|   5314|
+----------------+-------+
```

## Do you have missing and duplicate values in your dataset?
In our dataset there are no duplicate value in `trace_name` as it is a unique identifier. To confirm we did a simple calculation on `trace_name` column. (Refer to cell 12)

Our dataset does not contain duplicate rows. In regards to duplicates values within the columns themselves,  source_id and trace_name are the only ones that should not have any since they are unique identifier fields. Any duplicate values that appear in the other variables are either expected or meaningful to the data.

#### for missing values and duplicates update the notebook and edit the readme


## Preprocessing Plan (3 points)

- How will you handle missing values?
For samples in metadata that has missing values in either `trace_name` or `trace_category` will be dropped as they are part of unique identifiers. 

For quantitative variables from the metadata, we will be imputing them in medians as a lot them are skewed.

For qualitative variables from the metadata, we will be trying out different types of imputers to see the best performing imputer (ex: KNNImputer, Mode imputing, IterativeImputer, etc.)

- How will you handle data imbalance (if applicable)?
We will be using proportion of the positive label data to make the ratio between `earthquake_local` and `noise` to be similar.


- What transformations will you apply (scaling, encoding, feature engineering)?
We will apply the MinMaxScaler to hdf5 file containing waveform data. As the data is a frequency data we will be exploring differnt types of filters to filter out the noise.

In terms of cateogorical variables we are going to apply one-hot-encoding so that we can feed in the numerical representation.

We can feature engineering the displacement, velocity, accleration columns derived from the waveform data.

- What Spark operations will you use for preprocessing?

