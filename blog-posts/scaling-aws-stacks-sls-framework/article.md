---
published: false
title: 'Override the 500 resource limit of AWS CloudFormation templates with Serverless Framework'
cover_image: https://raw.githubusercontent.com/anaisberg/kumo-articles/master/blog-posts/scaling-aws-stacks-sls-framework/assets/header.png
description: 'Easily scale your services and workaround the 500 resource limit of AWS CloudFormation templates with Serverless Framework'
tags: aws, tutorial, cloudformation, serverless, serverless-framework
series:
---

# Override the 500 resource limit of AWS CloudFormation templates with Serverless Framework

AWS provides a robust infrastructure for deploying serverless applications using AWS CloudFormation. However, it's not uncommon to encounter resource limits when working on large serverless projects. One way to tackle this challenge is by splitting your CloudFormation stacks into smaller, more manageable units. AWS suggests themselves to use _nested stacks_ to overcome the 500-resource limit. In this article, we'll explore how to achieve this using the Serverless Framework and a plugin called `serverless-plugin-split-stacks`. We'll also discuss some tips and tricks to make the process smoother.

Nested stacks are a safe way to address the resource limit wall whilst keeping the benefits of a single stack. The main benefit of a single stack is that you can deploy and rollback all resources at once. As nested stacks have a dependency to their parent stack, so if one of the nested stacks fails to deploy, the parent stack will roll back to its previous state. We keep an atomic behavior of our stack.

### Prerequisites

Before we dive into the details, make sure you have the following in place:

1. AWS Account
2. Node.js and npm installed
3. Serverless Framework installed (`npm install -g serverless`)
4. A Serverless Framework project set up

### Splitting Stacks with serverless-plugin-split-stacks

#### Installation

First, you need to install the `serverless-plugin-split-stacks` plugin. Navigate to your Serverless project directory and run the following command:

```shell
npm install serverless-plugin-split-stacks --save-dev
```

#### Configuration

In your `serverless.ts` (or `serverless.yml`) file, add the plugin to the `plugins` section:

```typescript
 plugins: ['serverless-plugin-split-stacks'],
```

and configure it under the `custom` section:

```typescript
custom: {
  // ...
  splitStacks: {
    perFunction: false,
    perType: false,
    perGroupFunction: true,
    nestedStackCount: 10,
  },
}
```

- `perFunction`: Setting this to `true` would split the stack for each AWS Lambda function. Setting it to `false` keeps your structure manageable.
- `perType`: If you want to split stacks based on resource types (e.g., DynamoDB tables, S3 buckets), set this to `true`.
- `perGroupFunction`: This option is set to `true`, which splits stacks based on logical groupings of functions.
- `nestedStackCount`: disabled if not specified ; it controls how many resources are deployed in parallel.
- `stackConcurrency: number`: disabled if not specified ; it controls how many stacks are deployed in parallel.

#### Renaming Existing Lambdas

Here is the tricky part. The plugin doesn't work with existing resources. You can force the migration of an existing resource to a new stack if you use a custom migration with `force: true` but it will delete the resource and recreate it. It might create conflicts in CloudFormation or issues with IAM. This is not a good idea for production environments.

No worries, there is a workaround.

If you have existing Lambdas, you need to rename them to follow the new structure. The easiest way is to suffix their names with an underscore `_`. For example, if you had a Lambda named `myFunction`, you can rename it to `myFunction_`.

Just like that, without being deleted, your lambdas are moved to the new stack.

#### Deploying the Stacks

When deploying your Serverless project for the first time with these changes, ensure you set the `disableRollback` parameter to `false`. This way, if something goes wrong during deployment, AWS will not automatically roll back the changes. Here's how you can set it:

```shell
sls deploy --disableRollback false
```

After the initial deployment, it's recommended to set `disableRollback` back to `true` for safety reasons. This ensures that the stack rolls back to its previous state in case of deployment failures.

### Example

Let's illustrate these steps with a simple example. Consider a Serverless service with three functions: `userFunction`, `orderFunction`, and `paymentFunction`.

1. Install the `serverless-plugin-split-stacks` plugin.

2. Configure your `serverless.ts` file:

```yml
service: my-serverless-app

frameworkVersion: '>=2.50.0'
plugins:
  - serverless-plugin-split-stacks

provider:
  name: aws
  runtime: nodejs18.x

custom:
  splitStacks:
    perFunction: false
    perType: false
    perGroupFunction: true
    nestedStackCount: 10

functions:
  userFunction:
    handler: userFunction.handler
  orderFunction:
    handler: orderFunction.handler
  paymentFunction:
    handler: paymentFunction.handler
```

3. Rename existing Lambdas if necessary.

4. Deploy the stacks as explained above.

### 💭 Limitations

Hitting the resource limit may be a sign that your application is becoming too complex. It's a good idea to review your architecture and see if you can simplify it or re-organize it. Having smaller services is easier to maintain and manage.

### 🧠 Conclusion

By splitting your AWS CloudFormation stacks using the `serverless-plugin-split-stacks`, you can overcome the 500-resource limit and manage your serverless applications more effectively. This approach not only makes your infrastructure more scalable but also eases the maintenance of your serverless projects as they grow.

However, sub-stacks have their limits too and it does not mean you can grow them out indefinitely. Having to split your stacks is a sign that your application is becoming too complex and you should review your architecture.

#### References

- [AWS CloudFormation Limits](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)
- [AWS CloudFormation Nested Stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)
- [Serverless Framework](https://www.serverless.com/)
- [Serverless Plugin Split Stacks](https://www.npmjs.com/package/serverless-plugin-split-stacks)
