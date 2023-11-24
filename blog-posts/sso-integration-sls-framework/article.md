---
published: false
title: 'Use the AWS SSO with the Serverless Framework and deploy your stack easily'
cover_image:
description: 'Use AWS SSO to deploy your serverless-framework app.'
tags: aws, sso, tutorial, serverless-framework
series:
---

In this article, we'll explore how to use the Serverless Framework with the `serverless-better-credentials` plugin to log in to your AWS profile using Single Sign-On (SSO). We'll cover the new SSO configuration with sessions, and provide TypeScript examples to help you get started.

To learn more about SSO and SSO sessions, you can check out [this article](https://dev.to/slsbytheodo/understand-the-aws-sso-login-configuration-4am7).

## Prerequisites

Before we dive into the specifics of using the `serverless-better-credentials` plugin with SSO, ensure that you have the following prerequisites:

1. Serverless Framework: You should have the Serverless Framework installed. If not, you can install it using npm or yarn.

```bash
npm install -g serverless
```

2. AWS CLI + SSO Plugin: Make sure you have the AWS CLI installed and configured with your AWS account. Install the AWS CLI SSO plugin as well for managing SSO configurations. Full details in the [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/sso-configure-profile-token.html).

```bash
aws configure sso install
```

3. AWS SSO Configuration: Set up AWS Single Sign-On and configure your AWS SSO application.

4. AWS SDK: You'll need the `aws-sdk` package to interact with AWS services in your Serverless service.

```bash
npm install aws-sdk
```

## Using AWS SSO with the serverless framework

AWS Single Sign-On (SSO) integration with the Serverless Framework is not directly supported out of the box. That is where the `serverless-better-credentials` plugin comes in. It allows you to use your AWS SSO credentials to deploy your Serverless service.

However, the plugin did not support SSO sessions until recently. The trick was to skip the session parameter when setting up your profile, so your config file would look like the one before the SSO session feature was released. To learn more about sso sessions, [you can read this](https://dev.to/slsbytheodo/understand-the-aws-sso-login-configuration-4am7).

The latest version of the plugin (1.2.1) now supports SSO sessions, which makes it easier to use SSO with the Serverless Framework.

### Specify the your AWS profile

You need to tell the framework which profile you want to use to deploy your project. There are different ways of providing your profile credentials to the Serverless Framework.

I would recommend using an environment variable and then assigning its value to the provider.profile property.

```yaml
provider:
  name: aws
  profile: my-profile
```

Take note that if you are using SSO with the approach AWS document (a shared `.aws/config` file) you'll also need to set the `AWS_SDK_LOAD_CONFIG` environment value to something truthy (e.g. `AWS_SDK_LOAD_CONFIG=1`), as described in the [AWS SDK documentation](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-region.html#setting-region-config-file).

### Setting Up serverless-better-credentials

Now, let's set up the `serverless-better-credentials` plugin to work with your AWS SSO configuration.

### Install the Plugin:

In your Serverless project directory, install the serverless-better-credentials plugin, with the version 1.2.1 or higher:

```bash
npm install serverless-better-credentials@1.2.1 --save-dev
```

### Configure the Plugin:

In your serverless.yml file, add a section for the serverless-better-credentials plugin configuration. Make sure to specify the profileName for your AWS SSO profile.

```yaml
service: my-serverless-service
provider:
  name: aws
  profile: my-profile
  plugins:
    - serverless-better-credentials
```

### Install Required AWS SDK:

You'll need the aws-sdk package to interact with AWS services in your Serverless service. Install it by running:

```bash
npm install aws-sdk
```

Using serverless-better-credentials with SSO With the plugin configured, you can now use it to deploy your Serverless service using your AWS SSO credentials.

### Login with AWS SSO:

To log in using AWS SSO, you can run the following command, which will open a web browser for you to sign in:

```bash
aws sso login --profile my-dev-profile
```

Follow the on-screen instructions to complete the SSO login process.

### Deploy Your Serverless Service:

Once you've successfully logged in with AWS SSO, you can deploy your Serverless service as you normally would using the Serverless Framework. For example, to deploy your service:

```bash
serverless deploy
```

This will use your SSO credentials for authentication.

## TypeScript Example

Here's a simple TypeScript example of a Serverless service with an AWS Lambda function:

```typescript
// handler.ts

import { APIGatewayProxyHandler } from 'aws-lambda';

export const hello: APIGatewayProxyHandler = async (event, _context) => {
  return {
    statusCode: 200,
    body: JSON.stringify({
      message: 'Hello from Serverless!',
    }),
  };
};
```

In your serverless.yml, you would define your service and function like this:

```yaml
service: my-serverless-service
provider:
  name: aws
  runtime: nodejs18.x
  stage: dev
  region: us-east-1
functions:
  hello:
    handler: handler.hello
```

Now, you can run serverless deploy after logging in with AWS SSO to deploy your service.

## Conclusion

In this article, we've covered how to set up and use the _serverless-better-credentials_ plugin with AWS SSO for deploying your Serverless service. This approach simplifies the authentication process and ensures that you're using your SSO credentials securely.

### References

- [AWS SSO Login Configuration](https://dev.to/slsbytheodo/understand-the-aws-sso-login-configuration-4am7)
- [AWS CLI SSO Plugin](https://docs.aws.amazon.com/cli/latest/userguide/sso-configure-profile-token.html)
- [The serverless-better-credentials package](https://www.npmjs.com/package/serverless-better-credentials).
