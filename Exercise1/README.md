# Exercise 1

The purpose of this exercise is to familiarize you with the basics of CloudFormation templates. Sample file **s3.yaml** illustrates anatomy of a simple CludFormation template:
```
# Optional template format indicator
AWSTemplateFormatVersion: "2010-09-09"

# List of template parameters (optional)
Parameters:
  Name:
    Type: String
    Description: Name of the bucket to be created

# Section containing resources to be created (required)
Resources:
  S3Bucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: !Ref Name # Refes to the parameter

```

Templates describe AWS resources and can be deployed through CloudFormation. Once deployed, CloudFormation creates a **stack**. In order to deploy the template, in the **Exercise1** directory execute following command:

```
aws cloudformation deploy --stack-name Test --template-file s3.yaml --parameter-overrides Name=mybucket
```

Once executed the command should produce output below:
```
Waiting for changeset to be created..
Waiting for stack create/update to complete

Failed to create/update the stack. Run the following command
to fetch the list of events leading up to the failure
aws cloudformation describe-stack-events --stack-name Test
```

As you can see the command failed because an s3 bucket "mybucket" already exists. This is a common problem with resources that require globally unique names. In order to alleviate this problem, let's modify the template and deploy it once more. Before we do it though we need to delete the "Test" stack as it has been created in invalid state. To do so let's execute the following command:

```
aws cloudformation delete-stack --stack-name Test
```
Once deleted, modify the template in the s3.yaml file so it reads as follows:

```
AWSTemplateFormatVersion: 2010-09-09

Resources:
  S3Bucket:
    Type: "AWS::S3::Bucket"
```

When deployed, CloudFormation will generate unique bucket name automatically. To redeploy it execute the following:

```
aws cloudformation deploy --stack-name Test --template-file s3.yaml
```

This time we should get output similar to the following:

```
Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - Test
```

While the creation of stack has been succesfull, we do not know the name of the bucket that has been created. Let's modify the template once more and redeploy it so that we get some outputs. Let's modify the s3.yaml file by adding some outputs as per the content below:

```
AWSTemplateFormatVersion: 2010-09-09

Resources:
  S3Bucket:
    Type: "AWS::S3::Bucket"

Outputs:
  BucketName:
    Value: !Ref S3Bucket
    Description: Name of the sample Amazon S3 bucket with a lifecycle configuration.
  BucketDomain:
    Value: !Join 
      - "//"
      - - "https:"
        - !GetAtt S3Bucket.DomainName
    Description: Domain name of the bucket
```

We can now redeploy the existing stack

```
aws cloudformation deploy --stack-name Test --template-file s3.yaml
```

Once redeployed, we can check stack outputs by executing following command:

```
aws cloudformation describe-stacks --stack-name Test
```

Take note of the outpus, and save their values for future reference as we will need them in the next exercise. Notice the semi-random nature of the bucket's name.

```
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:eu-west-1:608998855138:stack/Test/fbe4e320-e8bc-11ec-8959-0a59b206a19d",
            "StackName": "Test",
            "ChangeSetId": "arn:aws:cloudformation:eu-west-1:608998855138:changeSet/awscli-cloudformation-package-deploy-1654867986/c65dea4d-0f84-4938-9da5-d319c5d4aa02",
            "CreationTime": "2022-06-10T13:33:07.122000+00:00",
            "LastUpdatedTime": "2022-06-10T13:33:12.424000+00:00",
            "RollbackConfiguration": {},
            "StackStatus": "CREATE_COMPLETE",
            "DisableRollback": false,
            "NotificationARNs": [],
            "Outputs": [
                {
                    "OutputKey": "BucketName",
                    "OutputValue": "test-s3bucket-854eav7hruyl",
                    "Description": "Name of the sample Amazon S3 bucket with a lifecycle configuration."
                },
                {
                    "OutputKey": "BucketDomain",
                    "OutputValue": "https://test-s3bucket-854eav7hruyl.s3.amazonaws.com",
                    "Description": "Domain name of the bucket"
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