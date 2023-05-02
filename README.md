# External Staging in Snowsight/Snowflake using Data from AWS S3 Bucket.

Snowflake is a cloud-based data warehouse that allows users to store and query structured and semi-structured data. One of the features of Snowflake is the ability to use external stages, which are references to locations where data files are stored outside of Snowflake, such as AWS S3 buckets or other object based storage services from other cloud providers   

Using external stages has several benefits, such as:   
• Reducing the storage costs and maintenance overhead of duplicating data in Snowflake.   
• Enabling faster data loading and unloading by avoiding network transfers between Snowflake and external locations.   
• Supporting incremental data loading by detecting changes in the external files.   

The following steps describe how to create and use an external stage in Snowflake using data from AWS S3:
    
1) Create an AWS S3 bucket and upload the data files that you want to use in Snowflake. The data files can be in various formats, such as CSV, JSON, Parquet, etc. Make sure that the S3 bucket and the data files have the appropriate permissions and encryption settings for Snowflake to access them.   

2) Create an IAM role in AWS that grants Snowflake access to the S3 bucket. The IAM role should have a trust policy that allows Snowflake to assume the role, and a permission policy that allows the role to read and write to the S3 bucket.        
    
    a) Creating a policy:
    ```JSON
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "VisualEditor0",
                "Effect": "Allow",
                "Action": "s3:ListBucket",
                "Resource": "arn:aws:s3:::snowflake-stage-bucket",
                "Condition": {
                    "StringLike": {
                        "s3:prefix": "Consumer Complaints/*"
                    }
                }
            },
            {
                "Sid": "VisualEditor1",
                "Effect": "Allow",
                "Action": [
                    "s3:PutObject",
                    "s3:GetObject",
                    "s3:DeleteObjectVersion",
                    "s3:DeleteObject",
                    "s3:GetObjectVersion"
                ],
                "Resource": "arn:aws:s3:::snowflake-stage-bucket/Consumer Complaints/*"
            },
            {
                "Sid": "VisualEditor2",
                "Effect": "Allow",
                "Action": "s3:GetBucketLocation",
                "Resource": "arn:aws:s3:::snowflake-stage-bucket"
            }
        ]
    }
    ```   
    b) The trust relationships has to be altered as below:   
    ```JSON
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "AWS": "*****<AWS ARN>***********"
                },
                "Action": "sts:AssumeRole",
                "Condition": {
                    "StringEquals": {
                        "sts:ExternalId": "*****<EXTERNAL ID>*********"
                    }
                }
            }
        ]
    }
    ```

3) Create an external stage in Snowflake that references the S3 bucket and the IAM role. You can use the CREATE STAGE command to create the external stage. You need to specify the URL of the S3 bucket, the prefix of the data files, and the ARN of the IAM role.
   
For example:
```SQL
CREATE STAGE my_external_stage
URL = 's3://my-s3-bucket/my-prefix'
CREDENTIALS = (AWS_ROLE = 'arn:aws:iam::123456789012:role/my-iam-role');
```

4) Use the external stage to load data from S3 into Snowflake tables. You can use the COPY INTO command or the Snowpipe feature to load data from the external stage into Snowflake tables. You need to specify the name of the external stage, the name of the target table, and any additional options for file format, error handling, etc.    

For example:
```SQL
COPY INTO my_table
  FROM @my_external_stage
  FILE_FORMAT = (TYPE = CSV)
  ON_ERROR = CONTINUE;
```
Now, you will be able to see the objects in the S3 bucket here in the staged area under your schema as shown below:   
![StagedFilesInSnowflake](https://github.com/saheen619/External-Staging-in-Snowflake/blob/main/Snips/Externally%20Staged%20Content%20from%20S3%20in%20Snowflake.JPG?raw=true)


Monitor and manage the external stage and its files. You can use various commands and functions in Snowflake to view and modify the properties of the external stage, list and delete its files, refresh its metadata, etc. For example:

-- Show properties of an external stage   
```SQL
DESC STAGE my_external_stage;
```

-- List files in an external stage   
```SQL
LIST @my_external_stage;
```

-- Delete files in an external stage   
```SQL
REMOVE @my_external_stage/pattern*;
```
-- Refresh metadata of an external stage   
```SQL
ALTER STAGE my_external_stage REFRESH;
```
## Tech Used

SNOWFLAKE - Snowsight   
AWS S3    
AWS IAM   

## Authors

[Saheen Ahzan](https://github.com/saheen619)


## Feedback

If you have any feedback, please reach out at saheen619.klm@gmail.com
