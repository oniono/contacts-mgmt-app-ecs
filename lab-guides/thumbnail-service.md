[Back to main guide](../README.md)

___

## Task 1 - Build and deploy Thumbnail-Service

In this section we will build and deploy the **thumbnail-service** as an ECS service. We will deploy the service on Container instances. The thumbnail-service is a daemon process and no external interactions happens with this service. This service monitors the SQS queue and creates thumbnails, no TCP ports are required to be exposed.

The ECS Task-Definition and Service-Definition are located in the thumbnail-service folder.

### Task 1.1 - Build thumbnail-service docker image

1. Navigate to the thumbnail-service folder
```bash
cd ./thumbnail-service
```
2. Create ECR Repository for thumbnail-service
```bash
aws ecr create-repository --repository-name contact-mgmt-app/thumbnail-service
```

3. Build the Docker Image for the thumbnail-service.
```bash
docker build -t <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/thumbnail-service:latest .
```

4. Login to Elastic Container Registry (ECR)
```bash
aws ecr get-login --no-include-email | bash
```

5. Push the Docker image to ECR
```bash
docker push <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/thumbnail-service:latest
```

### Task 1.2 - Deploy the thumbnail-service as ECS Service

1. Create a CloudWatch LogGroup to collect thumbnail-service logs.
```bash
aws logs create-log-group --log-group-name contact-mgmt-app/thumbnail-service
```

2. The thumbnail-service needs access to SQS Queue and S3 bucket. A **Task Role** provides the necessary permissions for accessing the dependent AWS resources. The Task Role was created by the CloudFormation stack deployed in the Prerequisite section of this lab. To get the Task Role ARN, refer to **ThumbnailServiceTaskRoleArn** variable in the **Outputs** section of the CloudFormation Stack.

3. Let us now register the ECS Task Definition for the thumbnail-service. A **task-definition.json** is located in the thumbnail-service folder, replace the parameter placeholders in the file with the values appropriate to your environment.

4. Register the ECS Task definition.

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

5. Create an ECS Service from the above Task Definition.

```bash
aws ecs create-service --cli-input-json file://service-definition.json 
```

___

[Back to main guide](../README.md)