---
title: CDK App with Github Action
date: 2023-01-18 15:08:52
categories: DevOps
tags:
    - Infra as Code
    - CDK
top:
---
# Background

In our business use case, we need to create a proxy server to redirect traffic from a specific region. To achieve this, we leverage on aws lambda + api gateway with proper CORs setting, and allowed methods. 

At first, we did this in aws console directly. It works fine, however, it would be more maintainable if we could achieve infra as code, benefits would be:

- Speed
- Low risk of human errors
- Improved consistency
- Eliminate configuration drift - correct source change mistake along with the pipeline deployment
- Improved security strategies
- Stable and scalable env
- Self documentation

# CDK Introduction

## Concepts

- CDK
    - **Cloud Development Kit**, is a framework for defining cloud infrastructure in code (IaC) and provisioning it through AWS CloudFormation.
    - We could leverage this tool to quickly build reliable, scalable, cost-effective applications in the cloud with the considerable expressive power of a programming language. [AWS CDK Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/home.html)
- Construct
    - **the basic building blocks of AWS CDK apps**.
    - encode configuration detail, boilerplate, and glue logic for using one or multiple AWS services.
- CloudFormation
    - CloudFormation gives an easy way to create a collection of related AWS and third-party resources, and provision and manage them in an orderly and predictable fashion
    - **Repeatable deployment, easy rollback, and drift detection**
    - Using a configuration language (YAML or JSON)
- Stacks
    - **a collection of AWS resources that you can manage as a single unit in CloudFormation**

## How does CDK work?

- AWS CDK takes the code we write (Stacks and Construct)
- Compiles it down to CloudFormation
- During cdk deployment, cloudFormation will help us create all resources we defined in CDK

![CDK Flow](https://s2.loli.net/2023/01/18/VGPw8zLkhREjegl.png)
## Benefits of Using CDK

- Customize, share, and reuse **constructs**
    - [Type](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html)
        - L1 Construct
            - CFN Resources – directly represent all resources available in AWS CloudFormation
        - L2 Construct
            - **L2**, also represent AWS resources, but with a higher-level, intent-based **API**.
            - Incorporate the defaults, boilerplate, and glue logic you'd be writing yourself with a CFN Resource construct
        - L3 Construct
            - ***patterns***.
            - These constructs are designed to help you complete common tasks in AWS, often involving multiple kinds of resources
            - [aws-ecs-patterns.ApplicationLoadBalancedFargateService](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ecs_patterns.ApplicationLoadBalancedFargateService.html) construct
                - represents an architecture that includes an AWS Fargate container cluster employing an Application Load Balancer.
                - help you to define load balancer, fargate service, ECS, ECR
                - we could also define our network setting like VPC, subnet, cloudMap inside the construct
- Powered by AWS CloudFormation
    - repeatable deployment, easy rollback, and drift detection
- Use familiar programming languages, tools, and workflows

Per the guide, inside Amazon, new features would be first developed in typescript, then leverage on some parsing tool to translate to different languages. Thus in this MVP, I’ll also use typescript.

# Development Process

**Prerequisite: You need to at least have permission to the corresponding aws account to move forward**

- The development loop consists of
    - Check CDK Doc, try to use constructs
    - Run `cdk diff` and `cdk synth`
        - check the output and make sure the changes are expected
    - Run `cdk deploy` to deploy aws resources
        - You should be able to see corresponding stacks in AWS Console - CloudFormation
        - Once the stack is deployed successfully, you should be able to see resources get created
    - If you find something wrong, you could just change the code, CDK and CloudFormation will help to take care of the update of your stack
    - If the stack is in some convoluted status, you could try to do `cdk destroy` or just manually delete the corresponding stack from the CloudFormation console page
        - remember the deletion needs to be started with the higher level stack, if there are some other stacks that rely on your current to-be-deleted stack, your deletion would not be able to finish
        - could happen when you deal with ALB/ NLB
        

## ****Build CDK app using constructs****

- CDK has different level construct per [guide](https://docs.aws.amazon.com/cdk/v2/guide/constructs.html). In our case, we should try our best to interact with **high-level constructs/ patterns** which have common tasks predefined. One good example would be [ApplicationLoadBalancedFargateService](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ecs_patterns.ApplicationLoadBalancedFargateService.html), we could leverage on such construct directly instead of defining load balancer, route table, IAM roles, ECR, ECS, Fargate, etc. on our own. It could save a lot of time, and also help us insist on the best practice per Amazon team guidance.
- [CDK API Doc](https://docs.aws.amazon.com/cdk/api/v2/) would be a great reference during the development
- One example in our case

```java
export class DemoStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const handler = new lambda.Function(this, "handler", {
      runtime: lambda.Runtime.NODEJS_16_X,
      code: lambda.Code.fromAsset("resource"),
      handler: "index.handler",
      timeout: cdk.Duration.seconds(30),
    });

    const lambdaIntegration = new HttpLambdaIntegration(
      "LambdaIntegration",
      handler
    );

    const httpApi = new apigatewayv2.HttpApi(this, "test", {
      apiName: "api-gateway",
      corsPreflight: {
        allowOrigins: [
          "https://llchen60.com",
        ],
        allowHeaders: ["paul",],
        allowMethods: [apigatewayv2.CorsHttpMethod.GET],
      },
      createDefaultStage: false,
    });

    httpApi.addStage("prod", {
      stageName: "prod",
      autoDeploy: true,
    });

    httpApi.addRoutes({
      path: "/llchen/{path+}",
      methods: [apigatewayv2.HttpMethod.GET],
      integration: lambdaIntegration,
    });
  }
}

const app = new cdk.App();
new DemoStack(app, "DemoStack");
```

## `cdk diff` to check local changes

- Compares the specified stack and its dependencies with the deployed stacks or a local CloudFormation template
- We should make sure the diff change are what we want locally before deployment

## `cdk synth` to generate CloudFormation file

- Synthesizes and prints the CloudFormation template for one or more specified stacks
- One Example when we generate our lambda server with api gateway
    - IAM Role
    - S3 Bucket for Code
    - Lamda handler
    - Api Gateway
    - Metadatas for defined resources
    

## ****`cdk deploy` - deploy to CloudFormation**

- Deploys one or more specified stacks
- Log would show the change and deployment progress

## Useful Commands

```tsx
// install cdk globally 
npm install -g aws-cdk

// configure access token, secret, account id, preferred region here
aws configure

mkdir cdkAppDictName 
cdk init app --language typescript

// check command guide for reference https://docs.aws.amazon.com/cdk/v2/guide/cli.html 
// list stacks in the app 
cdk list 

// synthesize the prints the cloudformation template 
cdk synth 

// bootstrap CDK toolkit stack 
cdk bootstrap 

cdk destroy 

// Compares the specified stack and its dependencies with the deployed stacks or a local CloudFormation template
cdk diff 

// Deploys one or more specified stacks
cdk deploy
```

# CI/ CD workflow setup

- After PR, we want to do a automatic deployment in github
    - there are couple options we could leverage on, E.G
        - buildkite
        - github action
- In this project, we use github action

## Github Action Core Concept

- Workflow
    - configurable automated process that will run one or more jobs
    - defined in yaml file
    - reside under `.github/workflows`
- Event
    - an activity to trigger a workflow to run
- Job
    - a set of steps in a workflow that execute on the same runner
    - executed in order
- Action
    - Perform a complex but frequently repeated task
    - Use action to help reduce the amount of repetitive code
- Runner
    - server runs the workflow when they are triggered

## Github Action Syntax

- that’s the main part we need to use
- we could define our action under  `.github/workflows` , define what events should trigger the workflow
- here we also need to define which runner we want to use, we could use public runner offered by github, or we could use self hosted one

[https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#run-name](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#run-name)

## Action YML File to integrate with Slack and CDK

- Prerequisite
    - this action would need several secrets for us to access slack or aws
    - we could generate them in corresponding platform, and put them under **github repository secrets section**
- Code

```tsx
name: "cdk deploy"
on:
  push:
    branches:
      - master
jobs:
  aws_cdk:
    runs-on: "your runner"
    steps:
      - name: Post to a Slack Channel Before Deployment
        id: slack-before-deploy
        uses: slackapi/slack-github-action@v1.23.0
        with:
          # Slack channel id, channel name, or user id to post message.
          # See also: https://api.slack.com/methods/chat.postMessage#channels
          channel-id: "channel-id"
          # For posting a rich message using Block Kit
          payload: |
            {
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Hello World*"
                  }
                }
              ]
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      - name: Checkout repo
        uses: actions/checkout@v3
      - uses: actions/setup-node@v2
        with:
          node-version: "14"
      - name: Configure aws credentials
        uses: aws-actions/configure-aws-credentials@master
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
          aws-region: us-west-1
      - name: Install dependencies
        run: npm install -g yarn && yarn
      - name: Synth stack
        run: yarn cdk synth
      - name: Deploy stack
        run: yarn cdk deploy --all
      - name: Post to a Slack Channel Post Deployment
        id: slack-after-deploy-success
        if: ${{ success() }}
        uses: slackapi/slack-github-action@v1.23.0
        with:
          # Slack channel id, channel name, or user id to post message.
          # See also: https://api.slack.com/methods/chat.postMessage#channels
          channel-id: "channel-id"
          # For posting a rich message using Block Kit
          payload: |
            {
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "text msg"
                  }
                }
              ]
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      - name: Post to a Slack Channel Post Deployment
        id: slack-after-deploy-failure
        if: ${{ failure() }}
        uses: slackapi/slack-github-action@v1.23.0
        with:
          # Slack channel id, channel name, or user id to post message.
          # See also: https://api.slack.com/methods/chat.postMessage#channels
          channel-id: "channel-id"
          # For posting a rich message using Block Kit
          payload: |
            {
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "test msg"
                  }
                }
              ]
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

That’s literally the whole process for us to integrate CDK with github actions, let me know if you have any questions! :)