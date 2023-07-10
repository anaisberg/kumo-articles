---
published: false
title: 'Understand the AWS SSO login'
cover_image:
description: 'Description of the article'
tags: aws, sso, tag3
series:
canonical_url:
---

![Logo kumo](./assets/logo_kumo_carre.png 'Logo Kumo')

This is an example of how to structure a blog post.

You can also take advantage of [embedme](https://github.com/zakhenry/embedme) to extract your code from the markdown file and make sure that what you're displaying in the markdown is always up to date too e.g.

```ts
// code/demo-code.ts

interface A {
  hello: string;
}
```



# Understanding the AWS SSO login

## How did we do before SSO?

Before using SSO, there were 3 main way to log into AWS :

1. **AWS Management Console:** using the user’s account credentials (email + password).
2. **AWS Command Line Interface (CLI):** using the CLI configured with their AWS access key ID and secret access key, from IAM.
3. **AWS SDKs and APIs:** by providing the AWS access key ID and secret access key in the code or using a configuration file.

These methods required users to manage separate sets of credentials for different AWS accounts and regions. Each time they accessed a different account or region, they had to provide the appropriate credentials.

AWS Single Sign-On (SSO) simplifies this process by enabling centralized user management.

# What is SSO? 

AWS Single Sign-On (SSO) is a service provided by AWS that simplifies the management of user access to multiple AWS accounts and applications. 

By providing a single set of credentials for accessing multiple AWS accounts and services. Users can log in once using their organization's identity provider (IdP) credentials and then access multiple AWS accounts without the need to enter separate credentials each time.

## Why do we use it?

The main advantages are: 

1. **Simplified Access Management:** administrators can define user permissions centrally and apply them across multiple accounts, making it easier to grant or revoke access as needed.
2. **Centralized User Management**: administrators can manage user identities, roles, and permissions from a single location, making it efficient to handle user onboarding, offboarding, and role changes.
3. **Integration with Identity Providers:** it allows organizations to leverage their existing identity systems and enable users to log in using their familiar corporate credentials. (IdP: Microsoft Active Directory, Okta, and Azure Active Directory, …).
4. **Single Sign-On Experience:** once authenticated with their organization's IdP, users can access various AWS accounts and applications without needing to re-enter credentials, improving productivity and reducing authentication fatigue.
5. **Increased Security:** reduces the risk of password-related issues. Users don't need to remember multiple passwords, reducing the likelihood of weak or reused passwords. Centralized access management also ensures consistent application of security policies and allows for easier auditing and monitoring of user activity.

*Overall, AWS SSO simplifies the management of user access to AWS accounts and applications, enhances security, and provides a better user experience by enabling single sign-on functionality.*

# New configuration: SSO introduces sso-sessions

In the v2 of the aws-cli, AWS introduced sessions to manage our SSO profiles.

### In aws-cli v1:

```
[profile dev]
sso_account_id = 123456789011
sso_role_name = FullAccess
sso_region = eu-west-2
sso_start_url = https://sso-portal.awsapps.com/start
region = west-1
output = json
```

### In aws-cli v2
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

## What changed in the new configuration: sso-sessions

We can now add sessions in our SSO configuration file. A session is linked to a start url and the region where the start url is hosted (different from the url where your account in hosted).

Then for one session, you can have multiple profiles. I can have a single session for my project start url, and different profiles linked to this session (so I have to configure only the session details once)

Another feature of the session is that you can restrict the access scope of the profiles. The default config is to grant the profile the access set to your listed accounts on the start url. 

So for the same start url, you could have a session allowing the profiles default access to your accounts, and one session restricting its profiles to only Read Access, e.g. (use sso access scope)

## Let's add a new profile
This time, we are going to add a profile that only has read access the account. 

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
You can you this profile by running. The command will automatically open an authorization page in your browser and fill the authorization code.
```
$ aws sso login --profile test-profile

// to log out
$ aws sso logout
```
To know which profile you are using, you can can run
```bash
aws sts get-caller-identity
```

# Take-aways
- AWS SSO makes it really easy for you to switch between profiles and to make the best use of the start url features. You don't have to deal with access keys anymore!
