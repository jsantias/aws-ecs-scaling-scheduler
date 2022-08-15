# ecs-scaling-scheduler
Scales all ecs service to 0 or 1 on a specific schedule

## main.yaml
Deploys the following resources to AWS:
1. AWS Lamba -> for scaling the ECS Services
2. AWS Cloudwatch rules -> one to send start command to Lambda, one for sending a stop command
3. AWS IAM Role -> IAM Role to attach to the Lambda Function
4. AWS IAM Policy -> Permissions for Lambda to write to CloudWatch Loggroups, and full ECS permissions
5. AWS API Gateway -> Allows clients to scale the ecs service through an API call (Uses API key authentication)

## lambda_function.py
Python3.8 script to scale all ECS services of a specific cluster either to 0 or 1.
The lambda takes the Cloudwatch Rule parameter for 'action' and 'cluster' to determine,
if the services should start or stop and on which cluster.

### lambda workflow
The lambda receives an 'action' value from Cloudwatch which is set to 'start' or 'stop'.
Additionally, the Cloudwatch rules submit the ECS Cluster name to the lambda.

Lambda uses the cluster name to query all services and put them into a list. Afterward
it iterates over this list and set the 'desired task count' to 1 or 0, depending on the 'action' value.

## lambda_function.zip
The zip file has to be uploaded to an S3 bucket before we run the Cloudformation code. 
The bucket name and the file key, needs to be entered in cloudformation. The cloudformation template
uses this information to pass the python code to the lambda function during its creation.

