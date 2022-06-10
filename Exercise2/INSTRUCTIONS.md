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

  # Web server using outputs of hte network stack
  WebServer:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: !Sub "${S3BucketUrl}/ec2.yaml"
      Parameters:
        Subnet: !GetAtt Network.Outputs.PublicSubnet1Id
        VPC: !GetAtt Network.Outputs.VpcId
```

As you can see the templae creates two **nested stacks** and uses S3BucketUrl parameter to provide location of their templates.

In order to deploy this stack we need to upload all the templates to an S3 bucket. Let's do that by executing following command in the **Exercise2** directory:
```
aws s3 cp . s3://BUCKET_NAME_FROM_EXERCISE1 --recursive
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
aws cloudformation create-stack --stack-name Web --template-url https://test-s3bucket-vb4a4enxs4aw.s3.amazonaws.com/web.yaml --parameters ParameterKey=S3BucketUrl,ParameterValue=https://test-s3bucket-vb4a4enxs4aw.s3.amazonaws.com --capabilities CAPABILITY_IAM
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

The stack status indicates progress of the operation. Status vale **CREATE_COMPLETE** indicates that all resources have been deployed.

