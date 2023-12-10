# Build a pipeline from S3 to Redshift

Build an ETL service pipeline to load data incrementally from Amazon S3 to Amazon Redshift using AWS Glue

## Table of Contents

- [Environment](#environment)
- [Summary](#summary)
- [Prerequisites and Limitations](#prerequisites-and-limitations)
- [Architecture](#architecture)
- [Tools](#tools)
- [Epics](#epics)
- [Additional Information](#additional-information)


## Environment

- Technologies: List of technologies used.
- AWS Services: List of AWS services used.

## Summary

This pattern provides guidance on how to configure Amazon Simple Storage Service (Amazon S3) for optimal data lake performance, and then load incremental data changes from Amazon S3 into Amazon Redshift by using AWS Glue, performing extract, transform, and load (ETL) operations. 

The source files in Amazon S3 can have different formats, including comma-separated  values (CSV), XML, and JSON files. This pattern describes how you can use AWS Glue to convert the source files into a cost-optimized and performance-optimized  format like Apache Parquet. You can query Parquet files directly from Amazon Athena and Amazon Redshift Spectrum. You can also load Parquet files into Amazon Redshift, aggregate them, and share the aggregated data with consumers, or visualize the data by using Amazon QuickSight.

## Prerequisites and Limitations

### Prerequisites

- An active AWS account.
- An S3 source bucket that has the right privileges and contains CSV, XML, or JSON files.
### Assumptions

- The CSV, XML, or JSON source files are already loaded into Amazon S3 and are accessible from the account where AWS Glue and Amazon Redshift are configured.

- Best practices for loading the files, splitting the files, compression, and using a manifest are followed, as discussed in the Amazon Redshift documentation.

- The source system is able to ingest data into Amazon S3 by following the folder structure defined in Amazon S3.

- The Amazon Redshift cluster spans a single Availability Zone. (This architecture is appropriate because AWS Lambda, AWS Glue, and Amazon Athena are serverless.) For high availability, cluster snapshots are taken at a regular frequency.


## Architecture for building an ETL pipeline from Amazon S3 to Amazon Redshift using AWS Glue


### Source Technology Stack

- S3 bucket with CSV, XML, or JSON files.

### Target Technology Stack

- AWS S3
- Amazon Redshift.

### Target Architecture

![Alt text](https://github.com/gaurangkudale/AWS-Data-Pipeline/blob/main-gk/aws%20data-pipeline.png)

### Data flow
[![Alt text](<AWS Data Pipeline.drawio.png>)](https://github.com/gaurangkudale/AWS-Data-Pipeline/blob/main-gk/AWS%20Data%20Pipeline.drawio.png)
## Tools for building an ETL pipeline from Amazon S3 to Amazon Redshift using AWS Glue

- Amazon S3 – Amazon Simple Storage Service (Amazon S3) is a highly scalable object storage service. Amazon S3 can be used for a wide range of storage solutions, including websites, mobile applications, backups, and data lakes.

- AWS Lambda – AWS Lambda lets you run code without provisioning or managing servers. AWS Lambda is an event-driven service; you can set up your code to automatically initiate from other AWS services.

- Amazon Redshift – Amazon Redshift is a fully managed, petabyte-scale data warehouse service. With Amazon Redshift, you can query petabytes of structured and semi-structured data across your data warehouse and your data lake using standard SQL.

- AWS Glue – AWS Glue is a fully managed ETL service that makes it easier to prepare and load data for analytics. AWS Glue discovers your data and stores the associated metadata (for example, table definitions and schema) in the AWS Glue Data Catalog. Your cataloged data is immediately searchable, can be queried, and is available for ETL.

- Redash – Redash features a simple interface that allows you to create new queries, visualize the data and share dashboards. It integrates with the most popular databases out there, including MySQL, PostgreSQL and Redshift (and more). It's often used for business intelligence, since it's designed to be user friendly and accessible to anyone who is working with data within an organization.

## Steps to create end to end automated data pipeline

### Step: 1 Create the S3 buckets and folder structure

- Create the S3 buckets and folder structure
- Create separate S3 buckets for each data source type and a separate S3 bucket per source for the processed (Parquet) data.
- Create a separate bucket for each source, and then create a folder structure that's based on the source system's data ingestion frequency; for example, s3://source-system-name/date/hour. For the processed (converted to Parquet format) files, create a similar structure; for example, s3://source-processed-bucket/date/hour. For more information about creating S3 buckets, see the [Amazon S3 documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html).



### Step: 2 Create a data warehouse in Amazon Redshift

- Launch the Amazon Redshift cluster with the appropriate parameter groups and maintenance and backup strategy.
- Create the database schema. Follow Amazon Redshift best practices for table design. Based on the use case, choose the appropriate sort and distribution keys, and the best possible compression encoding. For best practices, see the [AWS documentation](https://docs.aws.amazon.com/redshift/latest/dg/c_designing-tables-best-practices.html).
- Configure workload management. Conﬁgure workload management (WLM) queues, short query acceleration (SQA), or concurrency scaling, depending on your requirements. For more information, see [Implementing workload management](https://docs.aws.amazon.com/redshift/latest/dg/cm-c-implementing-workload-management.html) in the Amazon Redshift documentation.

### Step 3 : Configure AWS Glue

- In the AWS Glue Data Catalog, add a connection for Amazon Redshift

- To add an AWS Glue connection
  1. Sign in to the AWS Management Console and open the AWS Glue console at https://console.aws.amazon.com/glue
  2. In the navigation pane, under Data catalog, choose Connections.
  3. Choose Add connection and then complete the wizard, entering connection properties as described in AWS Glue connection properties.

- **Define the AWS Glue Data Catalog for the source**: This step involves creating a database and required tables in the AWS Glue Data Catalog. You can either use a crawler to catalog the tables in the AWS Glue database, or deﬁne them as Amazon Athena external tables. You can also access the external tables deﬁned in Athena through the [AWS Glue Data Catalog](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html). See the AWS documentation for more information about deﬁning the Data Catalog.
- **Create an AWS Glue job to process source data**: The AWS Glue job can be a Python shell or PySpark to standardize, deduplicate, and cleanse the source data ﬁles. To optimize performance and avoid having to query the entire S3 source bucket, partition the S3 bucket by date, broken down by year, month, day, and hour as a pushdown predicate for the AWS Glue job. Load the processed and transformed data to the processed S3 bucket partitions in Parquet format. You can query the Parquet ﬁles from Athena. 

- **Create an AWS Glue job to load data into Amazon Redshift**: The AWS Glue job can be a Python shell or PySpark to load the data by upserting the data, followed by a complete refresh.


### Step 4 : Create a Lambda function
- **Create a Lambda function to run the AWS Glue job based on the defined Amazon S3 event** : The Lambda function should be initiated by the creation of the Amazon S3 manifest ﬁle. The Lambda function should pass the Amazon S3 folder location (for example, source_bucket/year/month/date/hour) to the AWS Glue job as a parameter. The AWS Glue job will use this parameter as a pushdown predicate to optimize ﬁle access and job processing performance. For more information, see the [AWS Glue documentation](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-python-calling.html)

- **Create an Amazon S3 PUT object event to detect object creation, and call the respective Lambda function** : The Amazon S3 PUT object event should be initiated only by the creation of the manifest ﬁle. The manifest ﬁle controls the Lambda function and the AWS Glue job concurrency, and processes the load as a batch instead of processing individual ﬁles that arrive in a speciﬁc partition of the S3 source bucket. For more information, see the [Lambda documentation](https://docs.aws.amazon.com/lambda/latest/dg/with-s3-example.html).



## Related resources for building an ETL pipeline from Amazon S3 to Amazon Redshift using AWS Glue


- [Amazon S3 documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html).
- [AWS Glue documentation](https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html).
- [Amazon Redshift documentation](https://docs.aws.amazon.com/redshift/latest/gsg/getting-started.html).
- [AWS Lambda](https://aws.amazon.com/lambda/).
- [YouTube Videos of ETL | AWS Glue | AWS S3 | Load Data from AWS S3 to Amazon RedShift](https://www.youtube.com/watch?v=8tr9kCJTBl4)

