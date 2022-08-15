# ecs-scaling-scheduler
Scales all ecs services in the cluster to 0 or 1 on a specific schedule. 

Start times - 21:00 SUNDAY to THURSDAY UTC Time (07:00 AEST)

Stop times - 12:00 SUNDAY to THURSDAY UTC Time (22:00 AEST)

These times can be adjusted to your liking

## Setting the Start/Stop times

To set the events rules open the `main.yaml` and update the `ScheduleExpression` properties of the resources `StartEcsServicesRuleCloudwatch` and/or `StopEcsServicesRuleCloudwatch`. See the  [documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html ) for forming the cronjob expression 

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

## Accessing the API endpoint

The CloudFormation file will output the API endpoint for you. An API key is required to access the API endpoint. You can acquire this under the AWS API Gateway service. An example of the request, replacing the `<API-ENDPOINT>`, `<API-KEY>`,  `<ACTION>`, `<CLUSTER>` :

`<API-ENDPOINT>` - The API endpoint

`<API-KEY>` - The API key

`<ACTION>` - Valid values are `stop` or `start`

`<CLUSTER>` - The name of the ecs cluster

```
curl --location --request POST <API-ENDPOINT> \
--header 'x-api-key: <API-KEY>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "action": "<ACTION>"",
  "cluster": "<CLUSTER>"
}'
```

The API will return a status code 200 if successful
