[Back to main guide](../README.md)

___

## Task 5 - Build and deploy User-Interface-Service

In this section we will build and deploy the **user-interface-service** as an ECS service. We will deploy the service as an **ECS Fargate** service with **auto-scaling** behind an **Application Load Balancer (ALB)**.

The ECS Task-Definition and Service-Definition are located in the user-interface-service folder.

### Task 5.1 - Build user-interface-service docker image

1. Navigate to the user-interface-service folder.

```bash
cd ../user-interface-service
```

Let us look at the **config/endpoints.js** file, specifically the variables **contact_ep** and **user_ep**. Since we are using Service Discovery, we were able to pre-populate these endpoints with know domain names - http://user-profile-service.internal.

2. Create ECR Repository for user-interface-service

```bash
aws ecr create-repository --repository-name contact-mgmt-app/user-interface-service
```

3. Build the Docker Image and push it to ECR

```bash
docker build -t <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/user-interface-service:latest .
```

4. Login to Elastic Container Registry (ECR)

```bash
aws ecr get-login --no-include-email | bash
```

5. Push the Docker image to ECR

```bash
docker push <AWS-ACCOUNT-ID>.dkr.ecr.<AWS-REGION>.amazonaws.com/contact-mgmt-app/user-interface-service:latest
```

### Task 5.2 - Deploy the user-interface-service as ECS Service

There are a few prerequisites for deploying an ECS service on Fargate. This involves creating a **Task Execution Role** and **Security Group**. For tasks that use the Fargate launch type, the task execution role is required to pull container images from Amazon ECR or to use the awslogs log driver. Since Fargate launch type supports awsvpc network mode, a Security Group is required to provide network control to the containers launched by the ECS service.

1. The ALB, Target Group and Security Group were created by the CloudFormation stack created in the Prerequisite section.

2. Create a CloudWatch LogGroup to collect user-interface-service.
```bash
aws logs create-log-group --log-group-name contact-mgmt-app/user-interface-service
```

3. Let us now register the ECS Task Definition for the user-interface-service. A **task-definition.json** is located in the user-interface-service folder, replace the parameter placeholders in the file with the values appropriate to your environment. Since we are deploying the user-interface-service on ECS Fargate, we have set the **requiresCompatibilities** to FARGATE and **networkMode** to awsvpc. FARGATE only supports awsvpc network mode.

4. Register the ECS Task definition.

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

5. Replace the **`<UserInterfaceServiceALBTargetGroupArn>`**, **`<PRIVATE-SUBNET-01-ID>`**, **`<PRIVATE-SUBNET-02-ID>`** and **`<UserInterfaceServiceSecurityGroupId>`** in the service-definition.json with the appropriate value from the CloudFormation stack output and execute the below command to create an ECS Service.

```bash
aws ecs create-service --cli-input-json file://service-definition.json 
```

### Task 5.3 - Accessing the Contacts Management application

1. You can now access the application using ALB url. You can get the ALB URL from the CloudFormation Stack Output variable **ALBDNSName**.

2. Use the **Sign Up** link to register.

3. You will receive an email with a verification link to verify your account. Use the link in the email to verify your account.

4. Once the verification is complete, you can login to the application.

5. You can use the **Import CSV** button to import sample contacts into the application. Use the **sampleContacts.csv** file to load the sample contacts. 

___

[Back to main guide](../README.md)