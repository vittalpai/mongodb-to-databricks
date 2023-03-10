# MongoDB To Databricks
Data from MongoDB Atlas can be moved to Delta Lake in batch/real-time and can be aggregated with historical data and other data sources to run long-running analytics and complex machine learning pipelines to derive valuable insights and these insights can be moved back to MongoDB Atlas so that it reaches the right audience at the right time.

The data from MongoDB Atlas can be movies to Delta Lake in the following ways:
- One-time data load
    - [Using Spark Connector](#1-using-spark-connector)
    - [Using $out Operator & Object Storage](#2-using-out-operator--object-storage)
- Real-time data synchronization
    - [Using Spark Streaming](#1-using-spark-streaming)
    - [Using Apache Kafka and Object Storage](#2-using-apache-kafka-and-object-storage)

## Pre-requisites
- [MongoDB Atlas Cluster](https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/) loaded with data
- [Databricks Delta Lake Cluster](https://www.databricks.com/product/delta-lake-on-databricks)
- [AWS S3 bucket](https://aws.amazon.com/s3/)
- [MongoDB Shell](https://www.mongodb.com/try/download/shell)
- [Apache Kafka Cluster](https://kafka.apache.org/)
- Good understanding of MongoDB Atlas, Databricks, AWS Services, Kafka and Application services


## One-time data load
![One-time-data-load](/images/one-time-data-load.png)

### 1. Using Spark Connector
The MongoDB Connector for Apache Spark allows you to use MongoDB as a data source for Apache Spark. You can use the connector to read data from MongoDB and write it to Databricks using Spark's API and with the newly announced Databricks Notebooks integration, MongoDB developers now have an even easier and more intuitive interface to write complex transformation jobs.

- Login to Databricks cluster, Click on `New` > `Data`.
    ![Data](/images/new-data.jpeg)

- Click on `MongoDB` which is available under Native Integrations tab. This loads the pyspark notebook which provides a top-level introduction in using Spark with MongoDB.
    ![MongoDB Notebook](/images/mongodb-notebook.jpeg)

- Follow the instructions in the notebook to learn how to load the data from MongoDB to Databricks deltalake using Spark.
 
 
### 2. Using $out Operator & Object Storage
This approach involves using the $out stage in the MongoDB aggregation pipeline to perform a one-time data load into object storage. Once the data is in object storage, it can be configured as the underlying storage for a Delta Lake.

We need to set up a Federated Database Instance to copy our MongoDB data and utilize MongoDB Atlas Data Federation's $out to S3 to copy MongoDB Data and land it in an S3 bucket. 

- The first thing you'll need to do is navigate to "Data Federation" on the left-hand side of your Atlas Dashboard and then click "Create Federated Database Instance" or "Configure a New Federated Database Instance."

    ![Data Federation](/images/data_federation.jpeg)

- Connect your S3 bucket to your Federated Database Instance. This is where we will write the MongoDB data. The setup wizard should guide you through this pretty quickly, but you will need access to your credentials for AWS.

    ![Add data source](/images/add_data_source.jpeg)

- Select an AWS IAM role for Atlas.
    - If you created a role that Atlas is already authorized to read and write to your S3 bucket, select this user.
    - If you are authorizing Atlas for an existing role or are creating a new role, be sure to [refer to the documentation](https://docs.mongodb.com/datalake/deployment/deploy-s3/?_ga=2.93289885.12980670.1672725304-988028800.1667488630#select-an-iam-role-for) for how to do this.

- Enter the S3 bucket information.
    - Enter the name of your S3 bucket.
    - Choose Read and write, to be able to write documents to your S3 bucket.

- Assign an access policy to your AWS IAM role.
    - Follow the steps in the Atlas user interface to assign an access policy to your AWS IAM role.
    - Your role policy for read-only or read and write access should look similar to the following:
        ```
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                        "Effect": "Allow",
                        "Action": [
                        "s3:ListBucket",
                        "s3:GetObject",
                        "s3:GetObjectVersion",
                        "s3:GetBucketLocation"
                        ],
                        "Resource": [
                        <role arn>
                        ]
                }
            ]
        }
        ```

- Define the path structure for your files in the S3 bucket and click Next. Now you've successfully configured S3 bucket with Atlas Data Federation.

- Connect to your MongoDB instance using the MongoDB shell. This command prompts to enter the password.
    ```
    mongosh "mongodb+srv://server.example.mongodb.net" --username username
    ```
    Replace MongoDB connection string and username with appropriate values.

- Specify the database and collection that you want to export data from using the following commands.
    ```
    use db_name;
    db.collection_name.find()
    ```
    Replace `db_name` & `collection_name` with actual values & verify the data exists by running the above command.

- Use the `$out` operator to export the data to an S3 bucket. 
    ```
    db.[collection_name].aggregate([{$out: "s3://[bucket_name]/[folder_name]"}])
    ```
    Make sure to replace [collection_name], [bucket_name] and [folder_name] with the appropriate values for your S3 bucket and desired destination folder.

**Note**: The `$out` operator will overwrite any existing data in the specified S3 location, so make sure to use a unique destination folder or bucket to avoid unintended data loss.

## Real-Time Data Synchronization
Real-time data synchronization needs to happen immediately following the one-time load process. This can be achieved in multiple ways, as shown below.


### 1. Using Spark streaming
MongoDB has released a [version](https://www.mongodb.com/docs/spark-connector/current/) of the MongoDB Connector for Apache Spark that leverages the new Spark Data Sources API V2 with support for Spark Structured Streaming. MongoDB Connector for Apache Spark enables real-time micro-batch processing of data, enabling you to synchronize data from MongoDB to Databricks using Spark Streaming. This allows you to process data as it is generated, with the help of MongoDB's change data capture (CDC) feature to track all changes. By utilizing Spark Streaming, you can make timely and informed decisions based on the most up-to-date information available at Delta lake. More details about the streaming functionality can be found [here](https://www.mongodb.com/developer/languages/python/streaming-data-apache-spark-mongodb/).

![Real-time-sync-using-spark](/images/real-time-sync-using-spark.png)

- Login to Databricks cluster, Click on `New` > `Notebook`.
    ![Data](/images/new-notebook.png)

- In create a notebook dialog, enter a `name`, select `Python` as the default language, and choose the Databricks cluster. Then click on `Create`.
    
- Obtain the Spark streaming Python notebook from [here](https://github.com/vittalpai/mongodb-to-databricks/blob/main/data-streaming/mdb-spark-streaming.ipynb).

- Go to `File` > `Import`, and navigate to the notebook you downloaded in the previous step.

- Click on `Import` to add the data streaming notebook to your workspace.
    ![Data](/images/import-notebook.png)

- Follow the instructions in the notebook to learn how to stream the data from MongoDB to Databricks Delta Lake using Spark connector for MongoDB.


### 2. Using Apache Kafka and Object Storage 
Apache Kafka can be utilized as a buffer between MongoDB and Databricks. When new data is added to the MongoDB database, it is sent to the message queue using the MongoDB Source Connector for Apache Kafka. This data is then pushed to object storage using sink connectors, such as the Amazon S3 Sink connector. The data can then be transferred to Databricks Delta Lake using the Autoloader option, which allows for incremental data ingestion. This approach is highly scalable and fault-tolerant, as Kafka can process large volumes of data and recover from failures.

![Real-time-sync-using-kafka](/images/real-time-sync-using-kafka.png)

- Download & Install the MongoDB Source & AWS Sink Connector Plugin in your Kafka Cluster 
    - https://www.confluent.io/hub/mongodb/kafka-connect-mongodb
    - https://www.confluent.io/hub/confluentinc/kafka-connect-s3

- Update the following in the [mongodb-source-connector.properties](/connection-properties/mongodb-source-connector.properties) connector configuration file.
    - `CONNECTION-STRING` - MongoDB Cluster Connection String
    - `DB-NAME` - Database Name
    - `COLLECTION-NAME` - Collection Name

- Update the following in the [s3-sink-connector.properties](/connection-properties/s3-sink-connector.properties) connector configuration file.
    - `TOPIC-NAME` - Kafka Topic Name (i.e DB.COLLECTION name)
    - `S3-REGION` - AWS S3 Region Name
    - `S3-BUCKET-NAME` - AWS S3 Bucket Name where you wish to push the data.

- Deploy the connector configuration files in your Kafka Cluster. This will enable real time data synchronization from MongoDB to AWS S3 Buckets.

    **Note**: The above connector push the data to the S3 bucket at a regular interval of time, these configuration can be modified based on the use case. Refer to the following documentation for more details.
    - [MongoDB Souce Configuration](https://www.mongodb.com/docs/kafka-connector/current/source-connector/configuration-properties/)
    - [AWS S3 Sink Configuration](https://docs.confluent.io/kafka-connectors/s3-sink/current/configuration_options.html)

- Load the data from S3 buckets to Databricks Delta lake using Databricks [Autoloader](https://docs.databricks.com/ingestion/auto-loader/index.html) feature. Refer [this](https://docs.databricks.com/ingestion/auto-loader/unity-catalog.html) documentation for more details.