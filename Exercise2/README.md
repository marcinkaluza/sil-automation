# Exercise 2

The purpose of this exercise is to illustrate how we can use modular CloudFormation templates to deploy complex architectures. We will create a VPC and an EC2 instance hosting an instance of Apache web server.

The top level template is located in the **web.yaml** file and looks as follows:

```
Parameters:
  S3BucketUrl:
    Type: String
    Description: S3 bucket where the child templates are stored

Resources:
  # Network stack
  Network:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub "${S3BucketUrl}/vpc.yaml"

  # Web server using outputs of the network stack
  WebServer:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub "${S3BucketUrl}/ec2.yaml"
      Parameters:
        Subnet: !GetAtt Network.Outputs.PublicSubnet1Id
        VPC: !GetAtt Network.Outputs.VpcId
```

As you can see, the templae creates two **nested stacks** and uses S3BucketUrl parameter to provide location of their templates.

Before we start, let's set some environment variables so we do not have to type them over and over again. Let's set two environment variables to the values of outputs from Exercise 1:

```
export BUCKET_NAME=<your bucket name here>
export BUCKET_URL=<your bucket URL here>
```

we can verify that the variables have been set correctly by executing 

```
echo $BUCKET_NAME
echo $BUCKET_URL
```

In order to deploy this stack we need to upload all the templates to an S3 bucket. Let's do that by executing following command in the **Exercise2** directory.
```
aws s3 cp . s3://$BUCKET_NAME --recursive
```

The command should produce output similar to the one below:
```
upload: ./web.yaml to s3://test-s3bucket-vb4a4enxs4aw/web.yaml  
upload: ./vpc.yaml to s3://test-s3bucket-vb4a4enxs4aw/vpc.yaml  
upload: ./ec2.yaml to s3://test-s3bucket-vb4a4enxs4aw/ec2.yaml 
upload: ./INSTRUCTIONS.md to s3://test-s3bucket-vb4a4enxs4aw/INSTRUCTIONS.md
```

Now let's deploy the top level template:

```
aws cloudformation create-stack --stack-name Web --template-url $BUCKET_URL/web.yaml --parameters ParameterKey=S3BucketUrl,ParameterValue=$BUCKET_URL --capabilities CAPABILITY_IAM
```

The command should immediately return producing output similar to the one below:

```
{
    "StackId": "arn:aws:cloudformation:eu-west-1:608998855138:stack/Web2/e8253050-e8c7-11ec-91c4-0a90db923703"
}
```

The command works asynchronously and starts the tack creation in the background. In order to check the status of the stack we need to execute following command:

```
aws cloudformation describe-stacks --stack-name Web
```

This should produce output similar to the one below:

```
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:eu-west-1:608998855138:stack/Web3/4f2b7a50-e8ca-11ec-aeb3-06d33b112233",
            "StackName": "Web3",
            "Parameters": [
                {
                    "ParameterKey": "S3BucketUrl",
                    "ParameterValue": "https://test-s3bucket-vb4a4enxs4aw.s3.amazonaws.com"
                }
            ],
            "CreationTime": "2022-06-10T14:33:35.203000+00:00",
            "RollbackConfiguration": {},
            "StackStatus": "CREATE_IN_PROGRESS",
            "DisableRollback": false,
            "NotificationARNs": [],
            "Tags": [],
            "EnableTerminationProtection": false,
            "DriftInformation": {
                "StackDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
```

The StackStatus indicates progress of the operation. Status value of **CREATE_COMPLETE** indicates that all resources have been deployed. At this stage the output value of the stack can be seen:

```
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:eu-west-1:608998855138:stack/Web/8636b7d0-e8cf-11ec-bd5c-023a03f4704f",
            "StackName": "Web",
            "Parameters": [
                {
                    "ParameterKey": "S3BucketUrl",
                    "ParameterValue": "https://test-s3bucket-vb4a4enxs4aw.s3.amazonaws.com"
                }
            ],
            "CreationTime": "2022-06-10T15:10:55.019000+00:00",
            "RollbackConfiguration": {},
            "StackStatus": "CREATE_COMPLETE",
            "DisableRollback": false,
            "NotificationARNs": [],
            "Capabilities": [
                "CAPABILITY_IAM"
            ],
            "Outputs": [
                {
                    "OutputKey": "ServerIp",
                    "OutputValue": "34.240.212.83",
                    "Description": "IP address of the web server"
                }
            ],
            "Tags": [],
            "EnableTerminationProtection": false,
            "DriftInformation": {
                "StackDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
```

Take note of the public IP address of the deployed web server. Let's test it from the command line:

```
curl -v http://IP_ADDRESS_HERE -o /dev/null
```

This should return ouptut similar to the one below. 

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 34.240.212.83...
* TCP_NODELAY set
* Connected to 34.240.212.83 (34.240.212.83) port 80 (#0)
> GET / HTTP/1.1
> Host: 34.240.212.83
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 403 Forbidden
< Date: Fri, 10 Jun 2022 15:23:43 GMT
< Server: Apache/2.4.53 ()
< Upgrade: h2,h2c
< Connection: Upgrade
< Last-Modified: Tue, 12 Apr 2022 11:59:29 GMT
< ETag: "e2e-5dc73cb04ea40"
< Accept-Ranges: bytes
< Content-Length: 3630
< Content-Type: text/html; charset=UTF-8
< 
{ [3630 bytes data]
100  3630  100  3630    0     0  84418      0 --:--:-- --:--:-- --:--:-- 84418
* Connection #0 to host 34.240.212.83 left intact
* Closing connection 0
```

You can also verify the operation of the web server by opening the IP address in a web browser (use http:// prefix)

Since the stack runs on an EC2 instance, to avoid unnecessary costs remove the deployed infrastructure by executing following command:

```
aws cloudformation delete-stack --stack-name Web
```