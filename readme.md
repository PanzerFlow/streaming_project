# API Streaming Project (Nifi,Debezium,Kafka,Spark,Hudi)

- API Streaming Project (Nifi,Debezium,Kafka,Spark,Hudi)
  - [1. Introduction](#1-introduction)
  - [2. Design](#2-design)
  - [3. Output](#3-output)

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



