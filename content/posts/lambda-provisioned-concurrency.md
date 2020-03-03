---
title: "Lambda provisioned concurrency"
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

AWS Lambda as a main building block of serverless world. Overall serverless computing is great for various of use cases, but still have some limitations.
One of them could be latency. When lambda wasn't used for some time, an execution environment needs to be created, and  it could
take from 200ms to even 600ms, [depending on choosen language.](https://levelup.gitconnected.com/aws-lambda-cold-start-language-comparisons-2019-edition-%EF%B8%8F-1946d32a0244)
So when latency is critical for your application, additional 600ms could be painful. So here's Provisioned Concurrency comes to the rescue!
It creates execution environment just after function deployment, and it keeps function in initialized state, so latency is no longer an issue.


### How to setup provisioned concurrency

I think that it's always good to follow best practices, so on my blog I will try present demo's using
[infrastructure as a code](https://containersonaws.com/introduction/infrastructure-as-code/) approach.
Additional benefit is that you need to run template via SAM,
or AWS Cloudformation, instead manually clicking setup in a console.

In this post I will use SAM for demo purpose. Make sure you have installed SAM cli.

Setup is very simple. Let's go through it.

```bash
sam init
```

Then, in sam template we need to extend our function configuration by adding
ProvisionedConcurrencyConfig property:

```yaml
ProvisionedConcurrencyConfig:
  ProvisionedConcurrentExecutions: 5
```

But it's not enough. When you'll try to deploy you're function in this stage, you'll get following error:

```bash
Error: Failed to create changeset for the stack: lambda-provisioned-concurrency,
ex: Waiter ChangeSetCreateComplete failed: Waiter encountered a terminal failure state Status: FAILED.
Reason: Transform AWS::Serverless-2016-10-31 failed with: Invalid Serverless Application Specification document.
Number of errors found: 1. Resource with id [HelloWorldFunction] is invalid.
To set ProvisionedConcurrencyConfig AutoPublishALias must be defined on the function...
```

So as we clearly see we need include an alias to our function config.  Let's add it:

```yaml
AutoPublishAlias: live
```

So here's our final configuration:

```yaml
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello-world/
      Handler: hello-world
      Runtime: go1.x
      AutoPublishAlias: live
      ProvisionedConcurrencyConfig:
        ProvisionedConcurrentExecutions: 5
      Events:
        CatchAll:
          Type: Api
          Properties:
            Path: /hello
            Method: GET
```

So now we can deploy our function to AWS:

```bash
sam deploy --stack-name <you-stack-name> --guided --capabilities CAPABILITY_IAM
```

And after that, when deployment will be completed, you should be able to test your function.

### Results

To test results you can follow approch from AWS blog article. They're using ```ab```
tool to test function latency. Parameter ```n``` define number of request to perform.
Parameter ```c``` define number of multiple request perform at a time.

```bash
ab -n 1000 -c 10
```

You can run this function on two lambdas one with enabled provisioned concurrency, and on with disabled.
[On AWS blog](https://aws.amazon.com/blogs/aws/new-provisioned-concurrency-for-lambda-functions/) you could check results of similar experiment.
Also you could [run example](https://github.com/wbira/reinvent2019launches/tree/master/lambda.provisioned.concurrency) that I created on my github account.
