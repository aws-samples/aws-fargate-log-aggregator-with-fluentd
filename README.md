## AWS Fargate and Fluentd based Log Aggregator

A reference architecture for running a Fluentd Log Aggregator on AWS Fargate, which forwards logs to Kinesis Firehose. See [Building a scalable log solution aggregator with AWS Fargate, Fluentd, and Amazon Kinesis Data Firehose](https://aws.amazon.com/blogs/compute/building-a-scalable-log-solution-aggregator-with-aws-fargate-fluentd-and-amazon-kinesis-data-firehose/).

### Building the Fluentd Docker image

First, customize the Fluentd configuration file in [src](src). Then, build it into the docker image:

```
docker build -t fluentd-aggregator src
```

### Launching the CloudFormation template

The Fluentd aggregator is designed to be deployed to a VPC with private subnets behind a NAT Gateway. The [aws-samples/ecs-refarch-cloudformation](https://github.com/aws-samples/ecs-refarch-cloudformation) repository has a CloudFormation template which will create a VPC with two public and two private subnets: [vpc.yaml](https://github.com/aws-samples/ecs-refarch-cloudformation/blob/master/infrastructure/vpc.yaml).

Then create an ECS Cluster (if you don't already have one):

```
aws ecs create-cluster --cluster-name fargate-fluentd-tutorial
```

Next, create an [ECS Task Execution IAM role](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).

Finally, launch [template.yml](template.yml):

```
aws cloudformation deploy --template-file template.yml --stack-name aggregator-service \
--parameter-overrides EnvironmentName=fluentd-aggregator-service \
DockerImage=<Fluentd image> \
VPC=<your VPC ID> \
Subnets=<private subnet 1, private subnet 2> \
Cluster=fargate-fluentd-tutorial \
ExecutionRoleArn=<your task execution IAM role ARN> \
MinTasks=2 \
MaxTasks=4 \
--capabilities CAPABILITY_NAMED_IAM
```

## License

This library is licensed under the Apache 2.0 License.
