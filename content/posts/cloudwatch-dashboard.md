---
title: "Monitoring in AWS"
date: "2020-03-29"
description: "How to setup cloudwatch dashboards"
tags: [
  "aws",
  "cloudwatch",
  "logs",
  "serverless"
]
type: "post"
---

Nowadays to create successful project we need a bit more than developed application. We should have CI/CD solution for delivery changed to upper environments. To proof, that system is still working we can run set of e2e tests, but there's one more thing, that we should setup in our system.
And this missing thing is monitoring, which will help us to keep contol over health and performance of system. It may help keep optimal percent of utilization resources in a cloud, that will lead to reduced costs.
 <!--more-->
There are plenty of different third party tools, but in this article I'll show you how to setup simple with monitoring in AWS cloud.
Amazon Cloudwatch is monitoring service, that can gather logs and metrics form infrastructure resources, or even on-premise applications.
It allows to creates highly customized dashbords, with different types of widgets, that can be based on query logs, resource or custom metrics.
Additionaly we can setup alarms based on metrics. So with dashboards feature we have single point of view to our application data, that can be spread across different AWS regions, or even multiple accounts. I will show you how to create Cloudwatch dashboard in a console fist, and then how to defined it, as a infrastructure as a code.



Invocations
Errors count vs Success Rate
Logs with errors
Custom Metric Filters


Jak stworzyć cloudformation
