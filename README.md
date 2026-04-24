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
Numerical data distribution
<img width="2488" height="1989" alt="image" src="https://github.com/user-attachments/assets/1db3a568-7f5d-49e6-98ba-851c5b7d32ab" />

Categorical top 10 frequent values
<img width="2388" height="1189" alt="image" src="https://github.com/user-attachments/assets/1b8305f9-4d2c-4378-9158-22c7baf929dd" />


## Do you have missing and duplicate values in your dataset?

## For image data: describe number of classes, image sizes, uniformity, cropping/normalization needs.
