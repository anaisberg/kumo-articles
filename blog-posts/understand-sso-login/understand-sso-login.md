---
published: false
title: 'Understand the AWS SSO login configuration'
cover_image: https://raw.githubusercontent.com/anaisberg/kumo-articles/master/blog-posts/understand-sso-login/assets/sessions-2.jpg
description: 'Gain confidence on setting up, configuring and using the AWS SSO'
tags: aws, sso, tag3, tutorial
series:
canonical_url: understand-aws-sso-configuration
---

## TL;DR

AWS SSO makes it easy for you to switch between profiles. In this article, you'll learn how to easily set up your AWS profiles, to switch between them, and use the automatic SSO login! Moreover, we will focus on the new SSO sessions parameters.

## What is SSO? 

AWS Single Sign-On (SSO) is a service provided by AWS that simplifies the management of user access to multiple AWS accounts and applications. 

By providing a single set of credentials for accessing multiple AWS accounts and services. Users can log in once using their organization's identity provider (IdP) credentials and then access multiple AWS accounts without the need to enter separate credentials each time.

### How did we do before SSO?

Before using SSO, there were 3 main ways to log into AWS :

1. **AWS Management Console:** using the user’s account credentials (email + password).
2. **AWS Command Line Interface (CLI):** using the CLI configured with their AWS access key ID and secret access key, from IAM.
3. **AWS SDKs and APIs:** by providing the AWS access key ID and secret access key in the code or using a configuration file.

These methods required users to manage separate sets of credentials for different AWS accounts and regions. Each time they accessed a different account or region, they had to provide the appropriate credentials.

AWS Single Sign-On (SSO) simplifies this process by enabling centralized user management. It was made available in the `aws-cli` with the V2.

### Why do we use SSO?

The main advantages are: 

1. **Simplified Access Management:** administrators can define user permissions centrally and apply them across multiple accounts, making it easier to grant or revoke access as needed.
2. **Centralized User Management**: administrators can manage user identities, roles, and permissions from a single location, making it efficient to handle user onboarding, offboarding, and role changes.
3. **Integration with Identity Providers:** it allows organizations to leverage their existing identity systems and enable users to log in using their familiar corporate credentials. (IdP: Microsoft Active Directory, Okta, and Azure Active Directory, …).
4. **Single Sign-On Experience:** once authenticated with their organization's IdP, users can access various AWS accounts and applications without needing to re-enter credentials, improving productivity and reducing authentication fatigue.
5. **Increased Security:** reduces the risk of password-related issues. Users don't need to remember multiple passwords, reducing the likelihood of weak or reused passwords. Centralized access management also ensures the consistent application of security policies and allows for easier auditing and monitoring of user activity.

*Overall, AWS SSO simplifies the management of user access to AWS accounts and applications, enhances security, and provides a better user experience by enabling single sign-on functionality.*

### How to configure a profile?
You need to have a user account within an organization. Go to the start url of the access portal and sign in. (It looks like this: `https://sso-portal.awsapps.com/start`). Once you're signed in with your unique identifier, you will have access to all the accounts the organization has set for you.

Now you can configure your profile locally. The most common way to do so is with the `aws-cli` command
```bash
$ aws configure sso
```

It will save your profile configuration in the `~/.aws/config` file.

### What are the different parameters?

In aws-cli v1, a configuration file looks like this:

```
[profile dev]
sso_account_id = 123456789011
sso_role_name = FullAccess
sso_region = eu-west-2
sso_start_url = https://sso-portal.awsapps.com/start
region = west-1
output = json
```

You can have different profiles that point to the same account. However, the parameters give the profiles different use cases:
- `sso_account_id`: Your account id with the organization. (1 account id = 1 line on the access portal)
- `sso_role_name`: The IAM role that defines the permissions a user with this profile will have.
- `sso_region`: The region where the access portal is hosted.
- `sso_start_url`: The URL to your organization's access portal (you can get it in the invitation email or the IAM Identity Center service in the console).
- `region`: All the commands made through this profile will be sent to this region.
- `output`: The default output format for the commands.

Most of you may already be used to all of this, as it is considered a legacy configuration. Let's see what changed.
## New configuration: SSO introduces sso-sessions

In the 2.90 release of the aws-cli, AWS introduced sessions to manage our SSO profiles. Now, we do not need to refresh our tokens periodically when they expire. The refreshed tokens are automatically fetched by our SDK or tool.

### What changed in the new configuration: sso-sessions

We can now add sessions in our SSO configuration file. A session is linked to a start url and the region where the start url is hosted (different from the url where your account is hosted).

Then for one session, you can have multiple profiles. I can have a single session for my project start url, and different profiles linked to this session (so I have to configure only the session details once)

Another feature of the session is that you can restrict the access scope of the profiles. The default config is to grant the profile the access set to your listed accounts on the start url. 

So for the same start url, you could have a session allowing the profiles default access to your accounts, and one session restricting its profiles to only Read Access, e.g. (use sso access scope)

### What are the new settings
Now, in your config file, in addition to having the regular properties listed above, you can have a session linked to a profile. The start url, region, and registration scope are linked to the session, to better reflect what is going on in aws.

```text
[profile dev-profile]
sso_session = devto-article
sso_account_id = 123456789011
sso_role_name = FullAccess
region = eu-west-1
output = json

[sso-session devto-article]
sso_region = eu-west-2
sso_start_url = https://sso-portal.awsapps.com/start
sso_registration_scopes = sso:account:access
```
Let's break it down:
- In the profile, we now have `sso_session` points to the session linked to this profile
- The fields `sso_region` and `sso_start_url` are moved to the session's configuration, as they do not depend on the profile but on the access portal.
- There is a new parameter,  `sso_registration_scopes` that grants the scopes allowed to an application. It is the minimum access the accounts need to have to retrieve the access tokens. There is still very little documentation about it, [here is what AWS provides](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-config-sso_registration_scopes).


### Let's add a new profile with a session
This time, we are going to add a profile that only has read access to the account. 

```bash
$ aws configure sso

SSO session name (Recommended): devto-article
SSO start URL [None]: https://sso-portal.awsapps.com/start
SSO region [None]: eu-west-2
SSO registration scopes [sso:account:access]:

Using the account ID: 123456789011
There are 2 roles available to you.
> ReadOnly
  FullAccess
Using the role name "ReadOnly"

CLI default client region [eu-west-1]: eu-west-1
CLI default output format [json]: json
CLI profile name [123456789011_ReadOnly]: test-profile 
```
Our configuration file now looks like this:

```text
[profile dev-profile]
sso_session = devto-article
sso_account_id = 123456789011
sso_role_name = FullAccess
region = eu-west-1
output = json

[profile test-profile]
sso_session = devto-article
sso_account_id = 123456789012
sso_role_name = ReadOnly
region = eu-west-1
output = json

[sso-session devto-article]
sso_region = eu-west-2
sso_start_url = https://sso-portal.awsapps.com/start
sso_registration_scopes = sso:account:access
```

### Tips
You can you this profile by running. The command will automatically open an authorization page in your browser and fill in the authorization code.
```
$ aws sso login --profile test-profile

// to log out
$ aws sso logout
```
To know which profile you are using, you can run
```bash
aws sts get-caller-identity
```

## Take-aways
- AWS SSO makes it easy for you to switch between profiles and to make the best use of the start url features. You don't have to deal with access keys anymore!
- SSO sessions add one level of abstraction to have different access patterns with the same organization.
- Overall, the commands to set a profile and session do not change a lot, but if you decide to use sso sessions, make sure that the non-native AWS tools you use are maintained and have integrated the changes.
