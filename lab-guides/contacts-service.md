[Back to main guide](../README.md)

___

## Task 3 - Build and deploy Contacts-Service

In this section we will build and deploy the **contacts-service** as an ECS service. We will deploy the service on Container instances.

The ECS Task-Definition and Service-Definition are located in the contacts-service folder.

### Task 3.1 - Build contacts-service docker image

1. Navigate to the contacts-service folder

```bash
cd ../contacts-service
```

2. Create ECR Repository for contacts-service

```bash
aws ecr create-repository --repository-name contact-mgmt-app/contacts-service
```

3. Build the Docker Image and push it to ECR

```bash
docker build -t <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/contacts-service:latest .
```

4. Login to Elastic Container Registry (ECR)

```bash
aws ecr get-login --no-include-email | bash
```

5. Push the Docker image to ECR

```bash
docker push <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/contacts-service:latest
```

### Task 3.2 - Deploy the contacts-service as ECS Service

1. Create a CloudWatch LogGroup to collect contacts-service logs.

```bash
aws logs create-log-group --log-group-name contact-mgmt-app/contacts-service
```

2. Let us now register the ECS Task Definition for the contacts-service. A **task-definition.json** is located in the contacts-service folder, replace the parameter placeholders in the file with the values appropriate to your environment.

3. Register the ECS Task definition.

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

4. Replace the **`<ContactsServiceALBTargetGroupArn>`** in the service-definition.json with the appropriate value from the CloudFormation stack output and execute the below command to create an ECS Service.

```bash
aws ecs create-service --cli-input-json file://service-definition.json 
```

___
[Back to main guide](../README.md)