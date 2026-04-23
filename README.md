# DSC232
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
<insert SparkSession code>
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

