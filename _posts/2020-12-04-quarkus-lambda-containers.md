---
layout: post
name: quarkus-lambda-containers
title: How to Deploy a Quarkus Lambda Application using Lambda Containers
date: 2020-12-04 14:00:00 +02:00
background: /img/bg-berlin.jpg
---

# How to Deploy a Quarkus Lambda Application using Lambda Containers

At re:Invent 2020 Amazon announced support for deploying and running containers directly to Lambda. Here is a link to the blog post for the announcement [https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/). Recently I've been messing around with [Quarkus](https://quarkus.io) which is a relatively new Java framework for build App and has already a pretty impressive eco-system. 

This demo shows how you can packages a Quarkus Lambda App using the new [Lambda containers](https://docs.aws.amazon.com/lambda/latest/dg/lambda-images.html) support. A basically used the [Quarkus - Amazon Lambda](https://quarkus.io/guides/amazon-lambd) guide as the basis for this demo.

## TL;DR 
The code for this demo is available on GitHub [https://github.com/base2Services/quarkus-lambda-container-demo](https://github.com/base2Services/quarkus-lambda-container-demo)

## Prerequisites
To complete this guide, you need:

* less than 30 minutes
* JDK 11 (AWS requires JDK 1.8 or 11)
* Apache Maven 3.6.2+
* An Amazon AWS account
* AWS CLI
* docker

## So Lets get started


## Creating the Maven Deployment Project

First we are going to create a basic quarkus lambda app using the maven archetype

These steps are taken from the [quarkus lambda guide](https://quarkus.io/guides/amazon-lambda#creating-the-maven-deployment-project)

```bash
mvn archetype:generate \
       -DarchetypeGroupId=io.quarkus \
       -DarchetypeArtifactId=quarkus-amazon-lambda-archetype \
       -DarchetypeVersion=1.10.2.Final
```

This will have created a standard quarkus lambda app in a directory based on maven artifactId you entered. I will use `quarkus-lambda-demo` for the rest of the guide

### Add the AWS Lambda Runtime Editor Dependency

Add the following dependency to the maven pom dependencies

```xml
....
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>aws-lambda-java-runtime-interface-client</artifactId>
            <version>1.0.0</version>
        </dependency>
....
```

## Build the demo

In the quarkus-lambda-demo run maven

```bash
$ mvn clean install
...
INFO] --- maven-install-plugin:2.4:install (default-install) @ quarkus-lambda-demo ---
[INFO] Installing /Users/aaronwalker/Workspaces/aaronwalker/quarkus-lambda-container-demo/quarkus-lambda-demo/target/quarkus-lambda-demo-1.0-SNAPSHOT.jar to /Users/aaronwalker/.m2/repository/com/base2services/quarkus-lambda-demo/1.0-SNAPSHOT/quarkus-lambda-demo-1.0-SNAPSHOT.jar
[INFO] Installing /Users/aaronwalker/Workspaces/aaronwalker/quarkus-lambda-container-demo/quarkus-lambda-demo/pom.xml to /Users/aaronwalker/.m2/repository/com/base2services/quarkus-lambda-demo/1.0-SNAPSHOT/quarkus-lambda-demo-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  13.732 s
[INFO] Finished at: 2020-12-03T15:10:39+01:00
```

This will create the required artifacts in the target directory

## Lambda Function

The quarkus-lambda-demo project has by default configured the test handler in the `quarkus-lambda-demo/src/main/resources/application.properties/ and we will use this handler for the demo

```
quarkus.lambda.handler=test
```

## Building the Dockerfile

Create a Dockerfile in the quarkus-lambda-container-demo using the `public.ecr.aws/lambda/java:8.al2` lambda java8 Amazon Linux 2 runtime container as the base

```Dockerfile
# (1)
FROM  public.ecr.aws/lambda/java:8.al2

ARG APP_NAME=quarkus-lambda-demo
ARG APP_VERSION=1.0-SNAPSHOT

# (2) Copies artifacts into /function directory
ADD ${APP_NAME}/target/${APP_NAME}-${APP_VERSION}-runner.jar /var/task/lib/${APP_NAME}.jar
ADD ${APP_NAME}/target/lib/  /var/task/lib/

# (3) Setting the command to the Quarkus lambda handler
CMD ["io.quarkus.amazon.lambda.runtime.QuarkusStreamHandler::handleRequest"]

```
1) Details about the lambda containers images can be found at https://docs.aws.amazon.com/lambda/latest/dg/images-create.html

2) Copies the runner and it's dependencies into the default WORKDIR which is /var/task

3) Overrides the CMD using the default Quarkus lambda handler

### Then build the docker image

```bash
$ docker build -t quarkus/lambda-demo .
....
Successfully built 09666b8a56b0
Successfully tagged quarkus/lambda-demo:latest
```

### Testing local using docker

You can now use this image to test the lambda execution locally using

```bash
$ docker run --rm -it -p 9000:8080 quarkus/lambda-demo:latest
....
INFO[0000] exec '/var/runtime/bootstrap' (cwd=/var/task, handler=)
....
```

This starts the AWS lambda runtime emulator and a web server listen locally on port 9000. You can test the test lambda handler using curl

```bash
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{"greeting":"herzlich willkommen", "name":"aaron"}'
....
{"result":"herzlich willkommen aaron","requestId":"d8a48f84-a166-429e-a8ec-d8bea2e7087c"}
```

## Push the container to ECR

In order to be able to create a lambda function from our container image we need to push it to a registry. I will use ECR in the guide

*** Assumes you have valid AWS credentials configured

### Create the ECR registry

```bash
$ aws ecr create-repository --repository-name quarkus/lambda-demo --region eu-central-1
....
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:eu-central-1:<aws-accountid>:repository/quarkus/lambda-demo",
        "registryId": "<aws-accountid>",
        "repositoryName": "quarkus/lambda-demo",
        "repositoryUri": "<aws-accountid>.dkr.ecr.eu-central-1.amazonaws.com/quarkus/lambda-demo",
        "createdAt": "2020-12-03T16:10:37+01:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    
```

### Tag and push the container image to ECR

```bash
$ aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin <aws-accountid>.dkr.ecr.eu-central-1.amazonaws.com
$ docker tag quarkus/lambda-demo  <aws-accountid>.dkr.ecr.eu-central-1.amazonaws.com/quarkus/lambda-demo
$ docker push <aws-accountid>.dkr.ecr.eu-central-1.amazonaws.com/quarkus/lambda-demo
....
The push refers to repository [<aws-accountid>.dkr.ecr.eu-central-1.amazonaws.com/quarkus/lambda-demo]
```

### Create the Lambda function

Create a SAM template to deploy the function

**sam.container.yaml**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: AWS Serverless Quarkus - quarkus-lambda-demo-1.0-SNAPSHOT

Parameters:
  ImageUri:
    Type: String

Resources:
  QuarkusLambdaDemo:
    Type: AWS::Serverless::Function
    Properties:
      PackageType: Image
      ImageUri: !Ref ImageUri
      MemorySize: 256
      Timeout: 15
      Policies: AWSLambdaBasicExecutionRole

Outputs:
  Function:
    Value: !Ref QuarkusLambdaDemo
```

Now deploy the template

```bash 
AWS_REGION=eu-central-1
AWS_ACCOUNT_ID=xxxxxx
$ aws cloudformation deploy \
  --stack-name quarkus-lambda-demo \
  --template-file sam.container.yaml \
  --parameter-overrides ImageUri=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/quarkus/lambda-demo:latest \
  --capabilities CAPABILITY_IAM \
  --region ${AWS_REGION}
```

Now invoke the function

```bash
$ aws lambda invoke \
    --cli-binary-format raw-in-base64-out \
    --function-name QuarkusLambdaDemo \
    --payload '{"greeting":"herzlich willkommen", "name":"aaron"}' \
    --region ${AWS_REGION}
    out.json

{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
}

$ cat out.json
{"result":"herzlich willkommen aaron","requestId":"02d6569d-0452-4ed7-bdbe-7e860a8592d8"}
```

## Summary

Without much modification it was possible to create a simple Quarkus Lambda App and package it as a container image. Quarkus gives you the ability to run the app locally but by running in the container gives you the same environment that the app will run in when deployed to Lambda. Another big advantage of using a container is that you aren't restricted by the lambda zip file size limit. 
