# unity-catalog-x-table

![Screenshot 2024-07-03 at 10 03 13 PM](https://github.com/soumilshah1995/unity-catalog-x-table/assets/39345855/c08d7e3d-df8e-465c-8098-f205988d75ae)

# Create Hudi Tables 
```
from pyspark.sql import SparkSession
from pyspark.sql.types import *

# Initialize Spark session
spark = SparkSession.builder \
    .appName("HudiExample") \
    .config("spark.serializer", "org.apache.spark.serializer.KryoSerializer") \
    .config("spark.sql.hive.convertMetastoreParquet", "false") \
    .getOrCreate()

# Initialize the bucket
table_name = "people"
base_path = "s3://<BUCKET>/hudi-dataset"

# Define the records
records = [
   (1, 'John', 25, 'NYC', '2023-09-28 00:00:00'),
   (2, 'Emily', 30, 'SFO', '2023-09-28 00:00:00'),
   (3, 'Michael', 35, 'ORD', '2023-09-28 00:00:00'),
   (4, 'Andrew', 40, 'NYC', '2023-10-28 00:00:00'),
   (5, 'Bob', 28, 'SEA', '2023-09-23 00:00:00'),
   (6, 'Charlie', 31, 'DFW', '2023-08-29 00:00:00')
]

# Define the schema
schema = StructType([
   StructField("id", IntegerType(), False),
   StructField("name", StringType(), True),
   StructField("age", IntegerType(), True),
   StructField("city", StringType(), True),
   StructField("create_ts", StringType(), True)
])

# Create a DataFrame
df = spark.createDataFrame(records, schema)

# Define Hudi options
hudi_options = {
   'hoodie.table.name': table_name,
   'hoodie.datasource.write.recordkey.field': 'id',
   'hoodie.datasource.write.precombine.field': 'create_ts',
   'hoodie.datasource.write.partitionpath.field': 'city',
   'hoodie.datasource.write.hive_style_partitioning': 'true'
}

# Write the DataFrame to Hudi
(
   df.write
   .format("hudi")
   .options(**hudi_options)
   .mode("overwrite")  # Use overwrite mode if you want to replace the existing table
   .save(f"{base_path}/{table_name}")
)
```

# Download jar files
* utilities-0.1.0-beta1-bundled.jar  https://github.com/apache/incubator-xtable/packages/1986830

Define my_config.yaml
```
sourceFormat: HUDI

targetFormats:
  - ICEBERG
  - DELTA
datasets:
  -
    tableBasePath: s3://<BUCKET>/hudi-dataset/people/
    tableName: people
    partitionSpec: city:VALUE

```
# Run sync Command
```
%%bash 
java -jar  ./utilities-0.1.0-beta1-bundled.jar --dataset ./my_config.yaml

```


## Download UC 
```
git clone https://github.com/unitycatalog/unitycatalog.git

change /etc/conf/server.properties

server.env=false
s3.bucketPath.0="s3://<BUCKET>"
s3.accessKey.0="AWS ACCESS KEY"
s3.secretKey.0="AWS SECRET KEY"
s3.sessionToken.0=

THEN
build/sbt package


Start Server
bin/start-uc-server
```

## Register tables in UC
```
bin/uc table create \
    --full_name unity.default.hudi_table \
    --columns "id INT, name STRING, age INT, city STRING, create_ts STRING, _hoodie_commit_time STRING, _hoodie_commit_seqno STRING, _hoodie_record_key STRING, _hoodie_partition_path STRING, _hoodie_file_name STRING" \
    --storage_location s3://XXX/hudi-dataset/people \
    --format DELTA

bin/uc table read -full_name unity.default.hudi_table
bin/uc table delete --full_name unity.default.hudi_table
```
