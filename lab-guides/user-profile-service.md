[Back to main guide](../README.md)

___

## Task 4 - Build and deploy User-Profile-Service

In this section we will build and deploy the **user-profile-service** as an ECS service. We will deploy the service on Container instances.

The ECS Task-Definition and Service-Definition are located in the user-profile-service folder.

### Task 4.1 - Build user-profile-service docker image

1. Navigate to the user-profile-service folder

```bash
cd ../user-profile-service
```

2. Create ECR Repository for user-profile-service

```bash
aws ecr create-repository --repository-name contact-mgmt-app/user-profile-service
```

3. Build the Docker Image and push it to ECR

```bash
docker build -t <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/user-profile-service:latest .
```

4. Login to Elastic Container Registry (ECR)

```bash
aws ecr get-login --no-include-email | bash
```

5. Push the Docker image to ECR

```bash
docker push <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/user-profile-service:latest
```

### Task 4.2 - Deploy the user-profile-service as ECS Service

1. Create a CloudWatch LogGroup to collect user-profile-service logs.
```bash
aws logs create-log-group --log-group-name contact-mgmt-app/user-profile-service
```

2. Let us now register the ECS Task Definition for the user-profile-service. A **task-definition.json** is located in the user-profile-service folder, replace the parameter placeholders in the file with the values appropriate to your environment.

3. Register the ECS Task definition.

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

4. Replace the **`<UserProfileServiceALBTargetGroupArn>`** in the service-definition.json with the appropriate value from the CloudFormation stack output and execute the below command to create an ECS Service.

```bash
aws ecs create-service --cli-input-json file://service-definition.json 
```

___
[Back to main guide](../README.md)