# API Streaming Project (Nifi,Debezium,Kafka,Spark,Hudi)

- API Streaming Project (Nifi,Debezium,Kafka,Spark,Hudi)
  - [1. Introduction](#1-introduction)
  - [2. Design](#2-design)
  - [3. Output](#3-output)
  - [4. Setup](#4-setup)
## 1. Introduction
The goal of this project is to build a streaming data pipeline to report on live TTC bus/streetcar information. The final output of this project is an interactive PowerBI dashboard, which provides an overview of the route 7 Bathurst Street. 

The motivation of the project was based on an interest in learning different aspect of streaming data pipeline.

This project also provides an excellent opportunity to develop skills and experience in a range of tools. As such, the project is more complex than required, utilizing nifi, mysql, debezium, docker, and a series of AWS technology (S3, MSK,EMR,Athena).

An additional ppt file with similar content can also be found here [Streaming Project](https://github.com/PanzerFlow/streaming_project/blob/main/ppt/Streaming_Project.pptx)


## 2. Design
### Architecture
<img src="https://github.com/PanzerFlow/streaming_project/blob/main/images/Arch.PNG" width=110% height=110%>

Components
- Data Publisher
  - The architecture of this project is designed to be run continuously streaming API data of TTC bus and streetcar into S3.
  - Currently, the data source is public API that report on the lat, long of TTC buses on the Bathurst route.


- Ingestion and CDC
  - Ingestion is completed using NIFI via docker container. Data is ingested into MYSQL database as an append only table.
  - Debezium is used to perform change data capture on the MYSQL table, data is streaming into a topic on a managed kafka service cluster on aws


- Data Processing and Storage
  - Spark structured streaming is used to consume the kafka topic and transform the data as it arrives.
  - Data is then upserted into a hudi table on S3.

- Compute resources
  - EC2 is used to host nifi ,MySQL and Debezium connect via docker.
  - MSK is used to support the debezium CDC process, the topic is then consumed via EMR.
  - Athena are used as the compute engine which powers the dashboard.


- Visualization
  - PowerBI is used for dashboarding


## 3. Output
<img src="https://github.com/PanzerFlow/streaming_project/blob/main/images/PowerBI_MAP.png" width=110% height=110%>



## 4. Setup

Create an t2.xlarge EC2 instance with AWS and install docker

Create the MySQL Container for storing the NiFi Result
```bash
docker run -dit --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:1.6
```

Create the Nifi Container for pulling the API data
```bash
    docker run --name nifi -p 8080:8080 -p 8443:8443 --link mysql:mysql -d apache/nifi:1.12.0
```

Create your table inside of Mysql docker container 
```bash
	docker exec -it mysql bash
	mysql -u root -p
```

Use the Nifi Template in the repo and ingest data from your API to mysql table

Create an MSK cluster with auto.create.topics.enable=true. Make sure it is in the same VPC as your EC2 instance

Create Debezium Connect Container and point it to your MSK End Point

	
Start the CDC process and point it to your MSK End Point
```bash
    curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "root", "database.password": "debezium", "database.server.id": "184054", "database.server.name": "dbserver1", "database.include.list": "demo", "database.history.kafka.bootstrap.servers": "b-2.panzerstreaming.crgei7.c25.kafka.us-east-1.amazonaws.com:9092,b-3.panzerstreaming.crgei7.c25.kafka.us-east-1.amazonaws.com:9092,b-1.panzerstreaming.crgei7.c25.kafka.us-east-1.amazonaws.com:9092", "database.history.kafka.topic": "dbhistory.demo" } }'
```

Create an jar file with your scala code using sbt

Create an EMR instance and run Spark-submit
```bash
    spark-submit --master yarn --deploy-mode client --name panzerflow-stremaing-app --jars /usr/lib/hudi/hudi-spark-bundle.jar,/usr/lib/spark/external/lib/spark-avro.jar,/usr/lib/spark/jars/httpclient-4.5.9.jar,/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client-* --conf "spark.serializer=org.apache.spark.serializer.KryoSerializer" --conf "spark.sql.hive.convertMetastoreParquet=false" --class weclouddata.streaming.StreamingJob s3://panzerflow-streaming/jars/spark-streaming.jar
```


