# Exercise 4

The purpose of this exercise is to illustrate how we can use CodePipeline to automatically apply changes to our infrastructure whenever a CloudFormation template held in a CodeCommit (git) repository changes.

There are two stages to this exercise:
* deployment of the CodeCommit repository and CodePipeline 
* deployment of the infrastructure using CodePipeline

## Deployment of the CodePipeline

Follow the steps below to deploy your pipeline:

1. Log in to AWS console and navigate to CloudFormation page
![](img/01-CreateStack.png)
2. Click the "Create Stack" button. You will be taken to the page shown below. Select the "Upload template file" option and upload **cicd.yaml** file from the **Exercise4** directory. Click **Next** when done.
![](img/02-UploadTemplate.png)
3. Fill in the stack details page as per the image below and click next. 
![](img/03-StackDetails.png)
4. Click **next** on the **Configure Stack Options** Page
5. At the bottom of the **Review** page, tick the checkbox to acknowledge that cloud formation may create IaM resources. Finally click the **Create stack** button.
![](img/04-StackOptions.png)
6.  Once the deployment has been initiated, you will be taken to main cloud formation page where you can observe progress of the deployment.
![](img/05-StackCreating.png)
7. Wait for the Stack to get deployed and continue with the next section

## Using CodePipeline

In the previous section we have deployed a stack consisting of a CodeCommit repository and CodePipeline build pipeline. Now we will illustrate how they work together. The purpose of  CodePipeline is to initiate a set of configurable build actions whenever content of a code reposistory changes. CodePipeline can work with a number of different repositories, but in our case we use CodeCommit. To see how it works follow the steps below:

1. In the AWS console navigate to CodePipeline. You should see our pipeline in a failed state:
![](img/06-FailedPipeline.png)
2. Click on the name of the pipeline (Iac) to get to its details screen:
![](img/07-FailedPipelineDetails.png)
3. As you can see the source stage which downloads the files from code repository has failed. If you click on the details you will get further information. The reason for the failure is simply lack of any files in the repository.
![](img/08-FailureReason.png)
4. The pipeline expects file named **infra.yaml** to be present in the repository. In order to create it, first navigate to **CodeCommit** in the AWS console. You can get there by selecting **Source** and then **Repositories** in the left hand side menu.
![](img/09-IaCrepo.png)
5. Click on the **Iac** repository to get to the details screen:
![](img/10-Add%20file.png)
6. As you can see, the repository is empty. Let's add a file by clicking **Add file** button and selecting **Upload file** option. Click **Choose file** button and pick **infra.yaml** file from the **Exercise4** directory. Fill in your name, email address and commit mesage. Once done click the **Commit changes** button.
![](img/11-Commit.png)
7. The commit confirmation screen will be shown, showing the content of the added file:
![](img/12-CommitConfirmation.png)
8. In the AWS console, navigate back to the **CodePipeline** page. You will notice that our  pipeline has started automatically.
![](img/13-PipelineInProgress.png)
9. Click on the name of the pipeline to get to the details screen. 
![](img/14-PipelineinProgress.png)
10. Once the pipeline completes, you should see a screen like the one below:
![](img/15-PipelineComplete.png)
11. Click on the **Details** link of the **Deploy** stage to see the stack that the code pipeline created:
![](img/16-CreatedStack.png)

## Next steps
We have seen how a change (commit) to the CodeCommit repository automatically triggers CodePipeline execution which in turn deploys the infrastructure. Feel free to experiment with content of the infra.yaml file by adding tags or other options ot the S3 bucket created. Each time you commit a change, the pipeline will run and deploy it modifying the infrastructure to match the cloud formation template.