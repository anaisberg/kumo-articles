# Integration with serverless-framework

## The old way with programmatic access

[The Serverless Frameowrk guide to set up your credentials](https://www.serverless.com/framework/docs/providers/aws/guide/credentials)

**[Watch the video guide on setting up credentials](https://www.youtube.com/watch?v=KngM5bfpttA)**

With the Serverless Framework and programmatic access to AWS, the authentication process was typically handled using AWS access keys. Here's how it worked:

1. **AWS Access Key ID and Secret Access Key:** To interact with AWS services programmatically, users needed to generate an AWS Access Key ID and Secret Access Key. These keys served as the user's credentials to make API calls to AWS services.
2. **Configuration and Credentials File:** The Serverless Framework relied on the AWS SDKs to communicate with AWS services. Users had to configure their AWS credentials on their local development machine, typically by creating a configuration file named **`~/.aws/credentials`** (on Unix-based systems) or **`%USERPROFILE%\.aws\credentials`** (on Windows). The credentials file contained entries for different AWS profiles, each consisting of the Access Key ID and Secret Access Key.
3. **AWS Profile Selection:** The Serverless Framework supported multiple AWS profiles, allowing users to switch between different accounts and regions. The desired AWS profile was specified either in the command or as an environment variable (**`AWS_PROFILE`**).
4. **Deploying Serverless Applications:** Once the credentials were set up, users could deploy their serverless applications using the Serverless Framework. The framework would internally use the AWS SDKs and leverage the provided credentials to authenticate the API requests made during deployment.

By using AWS access keys and configuring the appropriate AWS profiles, developers could programmatically interact with AWS services and deploy serverless applications using the Serverless Framework. However, it's important to note that access keys should be kept secure and not shared publicly or hardcoded in application code, as they grant significant permissions to AWS resources.

### **Quick Setup**

As a quick setup to get started you can export them as environment variables so they would be accessible to Serverless and the AWS SDK in your shell:

```bash
export AWS_ACCESS_KEY_ID=<your-key-here>
export AWS_SECRET_ACCESS_KEY=<your-secret-key-here>
# AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY are now available 
# for serverless to use

serverless deploy
```

### **Using AWS Profiles**

For a more permanent solution you can also set up credentials through AWS profiles. Here are different methods you can use to do so.

### **Setup with `serverless config credentials` command**

Serverless provides a convenient way to configure AWS profiles with the help of the `serverless config credentials` command.

Here's an example how you can configure the `default` AWS profile:

```
serverless config credentials \
  --provider aws \
  --key AKIAIOSFODNN7EXAMPLE \
  --secret wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

```

Take a look at the **`[config` CLI reference](https://www.serverless.com/framework/docs/providers/aws/cli-reference/config-credentials)** for more information about credential configuration.

### **Setup with the `aws-cli`**

To set them up through the `aws-cli` **[install it first](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)** then run `aws configure` **[to configure the aws-cli and credentials](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)**:

```bash
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: ENTER

```

Credentials are stored in INI format in `~/.aws/credentials`, which you can edit directly if needed. You can change the path to the credentials file via the AWS_SHARED_CREDENTIALS_FILE environment variable. Read more about that file in the **[AWS documentation](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-config-files)**

You can even set up different profiles for different accounts, which can be used by Serverless as well. To specify a default profile to use, you can add a `profile` setting to your `provider` configuration in `serverless.yml`:

```
service: new-service
provider:
  name: aws
  runtime: nodejs14.x
  stage: dev
  profile: devProfile

```

### **Use an existing AWS Profile**

To easily switch between projects without the need to do `aws configure` every time you can use environment variables. For example you define different profiles in `~/.aws/credentials`

```
[profileName1]
aws_access_key_id=***************
aws_secret_access_key=***************

[profileName2]
aws_access_key_id=***************
aws_secret_access_key=***************
```

Now you can switch per project (/ API) by executing once when you start your project `export AWS_PROFILE="profileName2"` in the Terminal. Now everything is set to execute all the `serverless` CLI options like `sls deploy`.

### **Using the `aws-profile` option**

You can always specify the profile which should be used via the `aws-profile` option like this:

```
serverless deploy --aws-profile profileName1
```

- how we use serverless-better-credentials
- Serverless better credentials in a plug-in introduced to help support aws single dog on sso)

It was supposed to be a temporary plug-in

- https://www.serverless.com/plugins/serverless-better-credentials

## Using AWS SSO with serverless framework

AWS Single Sign-On (SSO) integration with the Serverless Framework is not directly supported out of the box. 

However, it is possible to configure the Serverless Framework to work with AWS SSO by leveraging the `serverless-better-credentials` plugin. 
AWS SSO profiles configured to work with the AWS CLI should "just work" when this plugin is enabled. This includes prompting and attempting to automatically open the SSO authorization page in your default browser when the credentials require refreshing.

Full details about how to configure AWS SSO can be found in the [AWS CLI documentation](https://docs.aws.amazon.com/cli/latest/userguide/sso-configure-profile-token.html).

Once you have configured your SSO profile(s), you need to add the serverless-better-credentials plugin to your `serverless(.yml|.ts|.js)` config file. [The doc for a quick setup](https://www.npmjs.com/package/serverless-better-credentials).

**Specify the AWS Profile for Serverless Framework**

You need to tell the framework which profile you want to use to deploy your project. There are different ways of providing your profile credentials to the Serverless Framework. 

Credentials are resolved in this order by the Serverless Framework currently uses:

1. from profile: cli flag `--aws-profile`
2. from profile: env `AWS_${STAGE}_PROFILE`
3. from env - `AWS_${STAGE}_X`
4. from profile - `AWS_PROFILE` 
5. from env - `AWS_X`
6. from profile - serverless.yml > `provider.profile` (unless --aws-profile is specified)
    
    ```yaml
    provider:
      name: aws
      profile: my-profile
    ```
    
7. from config - serverless.yml > `provider.credentials`
8. from profile - `AWS_DEFAULT_PROFILE` || `default`

Take note that if you are using SSO with the approach AWS document (a shared `.aws/config` file) you'll also need to set the `AWS_SDK_LOAD_CONFIG` environment value to something truthy (e.g. `AWS_SDK_LOAD_CONFIG=1`), as described in the [AWS SDK documentation](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-region.html#setting-region-config-file).

## New configuration: SSO introduces sso-sessions

In the v2 of the aws-cli, AWS introduced session to manage our SSO profiles.

In aws-cli v1:

```
[profile dev]
sso_account_id = 111122223333
sso_role_name = SampleRole
ss*o_region = us-east-1
sso_start_url = https://my-sso-portal.awsapps.com/start
region = us-west-2
output = json*
```

Now, in your config file, in addition to having the regular properties listed above, you can have a session linked to a profile. The start url, region, and registration scope are lined to the session, to better reflect what is going on in aws.

```
[profile my-dev-profile]
sso_session = my-sso
sso_account_id = 123456789011
sso_role_name = readOnly
region = us-west-2
output = json

[sso-session my-sso]
sso_region = us-east-1
sso_start_url = https://my-sso-portal.awsapps.com/start
sso_registration_scopes = sso:account:access
```

The issue is that the serverless-better-credential plugin only knows how to parse the v1 file to get the profile and the start url, but now that the start url is separated from our profile details, they can’t fetch it.

### **Current quick fix**

The immediate solution is not to provide a session name when configuring your profile. This will default to your configuration file looking like the v1 config file. 

```bash
$ aws configure sso
SSO session name (Recommended): [press space to leave empty]
SSO start URL [None]: https://my-sso-portal.awsapps.com/start
SSO region [None]: us-east-1
SSO registration scopes [None]: sso:account:access
```

Then go through the usual flow to select the account and role you want to link with your profile

### **Better, long-term fix**

What we need to use the session pragmatically is to be able to link the profile config to the session config, ie matching the session name in the profile with the available sessions and link its params to our profile.

I opened a PR on the serverless-better-credentials repo to do just that. Instead of just looking at the profiles configured through sso, we parse the sessions too, enriching the properties linked to the profile we use.

- Trick to use: need to specify the config folder
