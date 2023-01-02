# MongoDB To Databricks
Data from MongoDB Atlas can be moved to Delta Lake in batch/real-time and can be aggregated with historical data and other data sources to run long-running analytics and complex machine learning pipelines to derive valuable insights and these insights can be moved back to MongoDB Atlas so that it reaches the right audience at the right time.

[MongoDB to Databricks](/images/mongodb-to-databricks.png)


The data from MongoDB Atlas can be movies to Delta Lake in the following ways:
- One-time data load
- Real-time data synchronization


## One-time data load
[One-time-data-load](/images/one-time-data-load.png)


### 1. Using Spark Connector
The MongoDB Connector for Apache Spark allows you to use MongoDB as a data source for Apache Spark. You can use the connector to read data from MongoDB and write it to Databricks using Spark's API and with the newly announced Databricks Notebooks integration, MongoDB developers now have an even easier and more intuitive interface to write complex transformation jobs. Refer to this documentation for more details.
 
 
 
### 2. Using $out Operator & Object Storage
This approach involves using the $out stage in the MongoDB aggregation pipeline to perform a one-time data load into object storage. Once the data is in object storage, it can be configured as the underlying storage for a Delta Lake. For more information on this approach, refer to the documentation.


## Real-Time Data Synchronization
Real-time data synchronization needs to happen immediately following the one-time load process. This can be achieved in multiple ways, as shown below.



[One-time-data-load](/images/one-time-data-load.png)


### 1. Using Spark streaming
MongoDB Connector for Apache Spark enables real-time micro-batch processing of data, enabling you to synchronize data from MongoDB to Databricks using Spark Streaming. This allows you to process data as it is generated, with the help of MongoDB's change data capture (CDC) feature to track all changes. By utilizing Spark Streaming, you can make timely and informed decisions based on the most up-to-date information available at Delta lake.
[Real-time-sync-using-spark](/images/real-time-sync-using-spark.png)


### 2. Using Apache Kafka and Object Storage 
Apache Kafka can be utilized as a buffer between MongoDB and Databricks. When new data is added to the MongoDB database, it is sent to the message queue using the MongoDB Source Connector for Apache Kafka. This data is then pushed to object storage using sink connectors, such as the Amazon S3 Sink connector. The data can then be transferred to Databricks Delta Lake using the Autoloader option, which allows for incremental data ingestion. This approach is highly scalable and fault-tolerant, as Kafka can process large volumes of data and recover from failures.
[Real-time-sync-using-kafka](/images/real-time-sync-using-kafka.png)