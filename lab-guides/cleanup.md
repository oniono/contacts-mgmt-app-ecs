[Back to main guide](../README.md)

___

Congratulations on successfully completing the workshop. Time to cean up or environment.

1. Delete the CloudWatch Event and Target.

```bash
aws events remove-targets \
--rule contacts-mgmt-user-interface-service-codepipeline-event-rule \
--ids TargetCodePipeline

aws events delete-rule \
--name contacts-mgmt-user-interface-service-codepipeline-event-rule
```

2. Delete the CodePipeline.

```bash
aws codepipeline delete-pipeline --name contacts-mgmt-user-interface-service-codepipeline
```

3. Delete the CodeBuild Project.

```bash
aws codebuild delete-project --name contacts-mgmt-user-interface-service-build-project
```

4. Delete all objects in the Artifact S3 Bucket.

```bash
aws s3 rm --recursive s3://<PipleineArtifactBucket>
```

5. Delete the contacts-mgmt-user-interface-service-codepipeline CloudFormation Stack

```bash
aws cloudformation delete-stack --stack-name contacts-mgmt-user-interface-service-codepipeline
```

6. Stop and delete all ECS Services.

```bash
# user-interface-service
aws ecs update-service --cluster contacts-mgmt-app-cluster --service user-interface-service --desired-count 0
aws ecs delete-service --cluster contacts-mgmt-app-cluster --service user-interface-service

# user-profile-service
aws ecs update-service --cluster contacts-mgmt-app-cluster --service user-profile-service --desired-count 0
aws ecs delete-service --cluster contacts-mgmt-app-cluster --service user-profile-service

# contacts-service
aws ecs update-service --cluster contacts-mgmt-app-cluster --service contacts-service --desired-count 0
aws ecs delete-service --cluster contacts-mgmt-app-cluster --service contacts-service

# thumbnail-service
aws ecs update-service --cluster contacts-mgmt-app-cluster --service thumbnail-service --desired-count 0
aws ecs delete-service --cluster contacts-mgmt-app-cluster --service thumbnail-service
```

7. Deregister all ECS Task Definitions. Deregitering Task Definitions is easier from the ECS Console. You can select multiple revisions of a Task Definition and deregister them at once.

```bash
# user-interface-service
aws ecs list-task-definitions --family-prefix contact-mgmt-app-user-interface-service --query taskDefinitionArns
# repeat below command if you have multiple revisions
aws ecs deregister-task-definition --task-definition contact-mgmt-app-user-interface-service:<REVISION_NUMBER>

# user-profile-service
aws ecs list-task-definitions --family-prefix contact-mgmt-app-user-profile-service --query taskDefinitionArns
# repeat below command if you have multiple revisions
aws ecs deregister-task-definition --task-definition contact-mgmt-app-user-profile-service:<REVISION_NUMBER>

# contacts-service
aws ecs list-task-definitions --family-prefix contact-mgmt-app-contacts-service --query taskDefinitionArns
# repeat below command if you have multiple revisions
aws ecs deregister-task-definition --task-definition contact-mgmt-app-contacts-service:<REVISION_NUMBER>

# thumbnail-service
aws ecs list-task-definitions --family-prefix contact-mgmt-app-thumbnail-service --query taskDefinitionArns
# repeat below command if you have multiple revisions
aws ecs deregister-task-definition --task-definition contact-mgmt-app-thumbnail-service:<REVISION_NUMBER>
```

8. Delete the ECR repositories.

```bash
aws ecr delete-repository --force --repository-name contact-mgmt-app/user-interface-service
aws ecr delete-repository --force --repository-name contact-mgmt-app/user-profile-service
aws ecr delete-repository --force --repository-name contact-mgmt-app/contacts-service
aws ecr delete-repository --force --repository-name contact-mgmt-app/thumbnail-service
```

9. Delete the CloudWatch Logs Groups we created for the ECS services.

```bash
aws logs delete-log-group --log-group-name contact-mgmt-app/user-interface-service
aws logs delete-log-group --log-group-name contact-mgmt-app/user-profile-service
aws logs delete-log-group --log-group-name contact-mgmt-app/contacts-service
aws logs delete-log-group --log-group-name contact-mgmt-app/thumbnail-service
```

10. Delete the ALB Listener rules.

```bash
# Get the Rule ARNs
aws elbv2 describe-rules --listener-arn  <ALBListenerArn> 'Rules[*].{Arn:RuleArn,Default:IsDefault}'

# Repeat the Command below for multiple Rules except for rule where - Default:true
aws elbv2 delete-rule --rule-arn  <RULE-ARN>
```

11. Delete the Objets in the ImageS3Bucket

```bash
aws s3 rm --recursive s3://<ImageS3BucketName>
```

12. Delete the Cognito User Pool Domain that your created in the [Configuring the Cognito User Pool](#configuring-the-cognito-user-pool).

```bash
aws cognito-idp delete-user-pool-domain \
--domain <DOMAIN-NAME> \
--user-pool-id <CognitoUserPoolId>
```

13. Delete the contact-mgmt-stack CloudFormation Stack.

```bash
aws cloudformation delete-stack --stack-name contact-mgmt-stack
```

___
[Back to main guide](../README.md)