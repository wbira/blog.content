---
title: "Test Post"
date: "2020-03-02"
description: "Lambda provisioned concurrency"
aws: [
  "aws",
  "lambda",
  "serverless"
]
type: "post"
---

### What is Provisioned Concurrency for AWS Lambda?


### How to setup provisioned concurrency

Setup is very simple. Let's go through it.
I will use SAM for demo purpose. Make sure you have installed SAM cli.

``` sam init ```

Then, in sam template we need to extend our function configuration by adding
ProvisionedConcurrencyConfig property:

```
ProvisionedConcurrencyConfig:
  ProvisionedConcurrentExecutions: 5
```

But it's not enough. When you'll try to deploy you're function in this stage, you'll get following error:

```
Error: Failed to create changeset for the stack: lambda-provisioned-concurrency, ex: Waiter ChangeSetCreateComplete failed: Waiter encountered a terminal failure state Status: FAILED. Reason: Transform AWS::Serverless-2016-10-31 failed with: Invalid Serverless Application Specification document. Number of errors found: 1. Resource with id [HelloWorldFunction] is invalid. To set ProvisionedConcurrencyConfig AutoPublishALias must be defined on the function
```

So as we clearly see we need include an alias to our function config. Let's add it:

``` AutoPublishAlias: live ```

So now we can deploy our function to AWS:

``` sam deploy --stack-name <you-stack-name> --guided --capabilities CAPABILITY_IAM ```

And after that, when deployment will be completed, you should be able to test your function.

### Results

To test results you can follow approch from AWS blog article. They're using ```ab```
tool to test function latency.

```ab -n 1000 -c 10 ```

You can run this function on two lambdas one with enabled provisioned concurrency, and on with disabled.
