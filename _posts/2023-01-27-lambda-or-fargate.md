---
layout: post
name: lambda-or-fargate
layout: post
title: Choosing Between AWS Lambda and Fargate
time: 2023-01-27 14:00:00 +1:00
background: /img/bg-home.jpg
---

A common question I receive frequently is when to use AWS Lambda over AWS Fargate. Both being "serverless" services, it can be challenging to determine which one is more suitable for a specific use case. Let's dive deeper and explore the differences between these two services to help make a more informed decision.

AWS Lambda and Amazon Fargate are both services offered by Amazon Web Services (AWS) that allow users to run and scale code without having to manage the underlying infrastructure. While these services share some similarities, they are designed for different use cases and come with their own set of advantages and limitations.

AWS Lambda is a serverless computing service that allows you to run code without provisioning or managing servers. According to the AWS Lambda website, "AWS Lambda automatically scales and monitors your applications so you don't have to. You can build serverless applications that automatically scale based on incoming request traffic. You can also use AWS Lambda to create new back-end services where compute resources are automatically triggered based on custom requests.” This makes it a great choice for event-driven, short-lived processes, such as image or video processing, and real-time stream processing.

On the other hand, Amazon Fargate is a fully managed container orchestration service that allows you to run and scale containers without having to manage the underlying infrastructure. According to the AWS Blog, "Amazon Fargate makes it easy to run and scale containerized applications without having to manage the underlying infrastructure. You can launch and stop containers, scale your applications, and monitor their performance using the same familiar Amazon EC2 and Amazon ECS APIs and tools you already use.” This makes it a great choice for running and scaling long-running applications that require more control over the underlying infrastructure, such as microservices and big data workloads.

It's also worth noting that Lambda and Fargate can be used together as well, for example, you could use Lambda to trigger an action that then runs a containerized task on Fargate. According to the AWS Blog, "You can use AWS Lambda to process records in an Amazon Kinesis stream, detect image and video labels using Amazon Rekognition, and even run your own custom logic written in any programming language. With these capabilities, you can use AWS Lambda to build entire applications or augment the functionality of existing ones.”

In conclusion, when deciding between AWS Lambda and Amazon Fargate, it's important to consider the specific requirements of your application and use case. AWS Lambda is a great choice for event-driven, short-lived processes, while Amazon Fargate is a great choice for running and scaling long-running applications that require more control over the underlying infrastructure. Additionally, these services can be used together to build even more powerful and scalable applications.