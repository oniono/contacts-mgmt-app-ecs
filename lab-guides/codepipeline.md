[Back to main guide](../README.md)

___

## Task 6 - Create a CodePipeline for the User-Interface service

In this task we will setup a CodePipeline for the User-Interface service. The CodePipeline will be triggered when we commit a change to the CodeCommit repository. The CodePipeline will build the Docker image, push the new image to ECR and finally update the user-interface ECS service.

Before we begin ensure that you have installed Git and setup the Git credentials to access the CodeCommit repositories. For instructions on setting up the Git credentials refer the [documentation](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-ssh-unixes.html)

[Lab guide for Task-6](lab-guides/codepipeline.md)

### 6.1 - Create the CodePipeline prerequisite stack

Let us create the CodePipeline prerequistie CloudFormation stack. This CloudFormation stack creates the following resources:

- CodeCommit Repository 
- IAM Roles and Policies required by CodePipeline and CodeBuild
- CodePipeline artifact S3 Bucket

```bash
cd ../

aws cloudformation create-stack \
--stack-name contacts-mgmt-user-interface-service-codepipeline \
--template-body file://cfn-templates/codepipeline-prereq-cfn-template.yaml \
--capabilities CAPABILITY_IAM \
--parameters \
ParameterKey="EcrRepositoryArn",ParameterValue="arn:aws:ecr:<AWS-REGION>:<AWS-ACCOUNT-ID>:repository/contact-mgmt-app/user-interface-service"
```

### Task 6.2 - Create the CodeBuild Project

The CloudFormation stack created at the begining of the section created the prerequisites for the CodeBuild Project. The prerequisites include:

- **CodePipeline Artifact S3 Bucket** - The CodeBuild output artifact is stored in this S3 Bucket.
- **CodeBuild CloudWatch Log Group** - CodeBuild project build logs are sent to this Log Group.
- **IAM Service Role for CodeBuild** - CodeBuild will need IAM permission to pull the image from ECR, put the build artifact to the Artifact S3 Bucket and write build logs to CloudWatch Logs.

A **buildspec.yml** file is located in the **user-interface-service** folder. This file provides the build instructions to the CodeBuild service.

1. Create the CodeBuild project. Replace the serviceRole and environmentVariables parameters with values appropriate to your environment in **codebuild-project.json** file.

```bash
cd ./codepipeline
aws codebuild create-project --cli-input-json file://codebuild-project.json
```

### Task 6.3 - Create the CodePipeline

Now that we have the CodeCommit repository and CodeBuild project let us now create the CodePipeline. The CodePipeline will have 3 stages:
- **Source** - CodeCommit Repository
- **Build** - CodeBuild Project that will build the Docker image and push it to ECR.
- **Deploy** - This stage updates the ECS service **user-interface-service**

1. Create the CodePipeline. Replace parameter placeholders **`<CodePipelineServiceRoleArn>`** and **`<PipleineArtifactBucket>`** with values appropriate to your environment in **codepipeline.json** file.

```bash
aws codepipeline create-pipeline --cli-input-json file://codepipeline.json
```

2. Login to the AWS Console and navigate to the CodePipeline console to monitor the status of the CodePipeline.

### Task 6.4 - Create the CloudWatch Event to trigger CodePipeline on code change

A **CloudWatch Event Rule** is needed to trigger the CodePipeline when the source code changes in the CodeCommit repository. In this task we will create the CloudWatch Event Rule to trigger the CodePipeline we created in the previous task.

1. Create a CloudWatch Event Rule that will be triggered when there is a change in the CodeCommit repository.

```bash
aws events put-rule \
--name contacts-mgmt-user-interface-service-codepipeline-event-rule \
--description "Amazon CloudWatch Events rule to automatically start your pipeline when a change occurs in the AWS CodeCommit source repository and branch." \
--state ENABLED \
--event-pattern \
"{
    \"source\": [\"aws.codecommit\"],
    \"detail-type\": [\"CodeCommit Repository State Change\"],
    \"resources\": [\"<CodeCommitRepoArn>\"],
    \"detail\": {
        \"event\": [\"referenceCreated\",\"referenceUpdated\"],
        \"referenceType\": [\"branch\"],
        \"referenceName\": [\"master\"]
    }    
}"
```

2. Create a CloudWatch Event Target for the above CloudWatch Event Rule. The target will be the CodePipeline we created.

```bash
aws events put-targets \
--rule contacts-mgmt-user-interface-service-codepipeline-event-rule \
--targets \
"Id"="TargetCodePipeline",\
"Arn"="arn:aws:codepipeline:<AWS-REGION>:<AWS-ACCOUNT-ID>:contacts-mgmt-user-interface-service-codepipeline",\
"RoleArn"="<EventsServiceRoleArn>"
```

3. Now that we have configured the event rule and target, let us push a change to the CodeCommit repository and verify that our CodePipeline is triggered. We will change the login button text from "Log in" to "Sign-In" and push the changes to the CodeCommit Repository. The CodePipeline will automatically build the docker image and update the user-interface-service.

```bash
cd ../user-interface-service/
```

4. Edit the user-interface-service/user-interface-service-code/views/index.html and replace **`Log in</button>`** with **`Sign-In</button>`**.

5. Push the updated code to CodeCommit Repository.

```bash
git add *
git commit -m "changed login button text to Sign-In"
git push
```

6. Navigate to the CodePipeline Console to monitor your pipeline. Once the deployment is complete, you can verify the Log in page of the application.

___

[Back to main guide](../README.md)