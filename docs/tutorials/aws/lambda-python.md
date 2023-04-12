# Monitor Serverless Functions with AWS Distro for OpenTelemetry with New Relic (Python)

## References
 - https://aws-otel.github.io/docs/getting-started/lambda/lambda-python
 - https://aws-otel.github.io/docs/getting-started/lambda#custom-configuration-for-the-adot-collector-on-lambda
 - https://aws.amazon.com/blogs/opensource/auto-instrumenting-a-python-application-with-an-aws-distro-for-opentelemetry-lambda-layer/

## Code
Log into your AWS console and go to AWS Lambda.  On the top right, click **Create function**.
![Screenshot 1](/docs/tutorials/aws/lambda-python/lambda-python_01.png)

Select **Author from scratch**.  
![Screenshot 2](/docs/tutorials/aws/lambda-python/lambda-python_02.png)

Then under Basic Information, give your function a name.  Select a runtime (Python 3.9 or 3.8 should work for this example).  Select an architecture.  In this example, we use the x86_64 option.  No other changes are needed here, so click **Create function** on the bottom right.
 - **x86_64 (amd64)**
 - arm64

![Screenshot 3](/docs/tutorials/aws/lambda-python/lambda-python_03.png)

Note: Your function name will be your `entity.name` and `service.name` in New Relic.

### Code source

1. Create a new lambda function from scratch
    ```
    #! /usr/bin/python3

    import boto3
    import json
    import os

    def lambda_handler(event, context):
        client = boto3.client("s3")
        client.list_buckets()
        
        client = boto3.client("ec2")
        client.describe_instances()

        return {
            "statusCode": 200,
            "headers": {
                "Content-Type": "application/json"
            },
            "body": json.dumps({
                "Region ": os.environ['AWS_REGION']
            })
        }
    ```

    ![Screenshot 4](/docs/tutorials/aws/lambda-python/lambda-python_04.png)

2. Go to File > New File and add `collector.yaml` to the root of the project.  Replace api-key with your New Relic INGEST - LICENSE key.

    ```
    #collector.yaml in the root directory
    #Set an environemnt variable 'OPENTELEMETRY_COLLECTOR_CONFIG_FILE' to '/var/task/collector.yaml'

    receivers:
    otlp:
        protocols:
        grpc:
        http:

    exporters:
    logging:
        loglevel: debug
    otlp:
        endpoint: "otlp.nr-data.net:4317"
        headers:
        api-key: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXNRAL"

    service:
    pipelines:
        traces:
        receivers: [otlp]
        exporters: [otlp]
    ```
    ![Screenshot 5](/docs/tutorials/aws/lambda-python/lambda-python_05.png)


### Layers
Add a layer and **Specify an ARN**. Add the (Lambda layer)[https://aws-otel.github.io/docs/getting-started/lambda/lambda-python#add-the-arn-of-the-lambda-layer] accordingly.

| Architecture | ARN |
| ------------ | --- |
| x86_64 | arn:aws:lambda:**ca-central-1**:901920570463:layer:aws-otel-python-**amd64**-ver-1-16-0:2 |
| arm64 | arn:aws:lambda:**ca-central-1**:901920570463:layer:aws-otel-python-**arm64**-ver-1-16-0:2 |


## Configuration

## Permissions
This specific Python example requires read access to S3 buckets (`s3:ListAllMyBuckets`) and EC2 instances (`ec2:DescribeInstances`).  However, we’ll be simplifying this step.

Go to permissions and click on the **Role name**.  This should have been automatically created when the Lambda function was created.

On the right, click on **Add permissions** then select **Attach policies**.  Search for “AmazonS3ReadOnlyAccess” and “AmazonEC2ReadOnlyAccess” then **Add permissions**.

Note: If you want to generate errors, remove these permissions and text/invoke your Lambda function again.

![Screenshot 6](/docs/tutorials/aws/lambda-python/lambda-python_06.png)

## Environment variables

| Key | Value | Notes |
| --- | ----- | ----- |
| AWS_LAMBDA_EXEC_WRAPPER | /opt/otel-instrument | Enables instrumentation.  Remove to uninstrument. |
| OPENTELEMETRY_COLLECTOR_CONFIG_FILE | /var/task/collector.yaml | Uses custom collector configuration file. |

![Screenshot 7](/docs/tutorials/aws/lambda-python/lambda-python_07.png)


## Monitoring and operations tools
Active tracing (X-Ray) must be enabled.

![Screenshot 8](/docs/tutorials/aws/lambda-python/lambda-python_08.png)

![Screenshot 9](/docs/tutorials/aws/lambda-python/lambda-python_09.png)

Test your Lambda function and deploy.  Create a function URL if needed.

## New Relic
After testing or invoking your Lambda function from the Function URL, check New Relic for the data.

### NRQL
Query your data and use the following NRQL to see data:

```
SELECT * FROM Span
```

![Screenshot 10](/docs/tutorials/aws/lambda-python/lambda-python_10.png)

### APM & Services
Under OpenTelemetry, check for your Lambda function name to appear.

#### Summary
![Screenshot 11](/docs/tutorials/aws/lambda-python/lambda-python_11.png)

#### Distributed Tracing
![Screenshot 12](/docs/tutorials/aws/lambda-python/lambda-python_12.png)
![Screenshot 13](/docs/tutorials/aws/lambda-python/lambda-python_13.png)
![Screenshot 14](/docs/tutorials/aws/lambda-python/lambda-python_14.png)

#### Transactions
![Screenshot 15](/docs/tutorials/aws/lambda-python/lambda-python_15.png)

#### External Services
![Screenshot 16](/docs/tutorials/aws/lambda-python/lambda-python_16.png)

#### Errors Inbox
If you want some errors, remove permissions for S3 and EC2 read-only access and invoke the Lambda function again.
![Screenshot 17](/docs/tutorials/aws/lambda-python/lambda-python_17.png)
![Screenshot 18](/docs/tutorials/aws/lambda-python/lambda-python_18.png)
