Now that we have the base Control Tower setup we have our basic landing zone configured.  What does this give us?  
Not much in terms of operational capability.  Just foundational components.  We have 3 accounts.  We have the original 
account that we created to trigger everything, the management account, a centralized logging account to receive logs 
into s3, the logging account, and an account to receive security events, the audit account.

Some of the other things we get out of the box are:
1. Centralized guardrails in the form of Service Control Policies from AWS Organizations
2. a way to centrally manage the deployment and orchestration of accounts that would have the SCPs attached.
3. SSO configured using AWS IAM Identity Center

We can see a lot of things in the Control Tower Dashboard.  There we can see in the `Environment summary` that we have
2 organizational units and 3 accounts.  We can also see in the `Enabled control summary` that we have 21 Preventative
controls and 3 Detective controls (at least as of the time of this writing, 24 in total).  And if we click on the 21, we end up on a 
page that says `All controls` and a list of 427 controls!  What?  An easy way to see them is by using the awscli.  Open 
the `AWS Organizations` console in another tab, click on `AWS accounts` and select the `Security` OU. Copy it's ARN and 
paste it at the of this command:

```shell
aws controltower list-enabled-controls --target-identifier REPLACE_ME_WITH_ARN
```

Why the `Security` OU?  Because controls are not applied to the root OU.   Not all are applied to the `Infrastructure` OU.
Ones that are applied to both are indicated below with an asterisk (*).  Why do we want to know this? Well, if we are 
working in a regulated environment or if we care about security, this is good to know in case we get asked (ie, audited).

### Default Detective and Preventative Controls deployed by AWS Control Tower 

1. AWS-GR_AUDIT_BUCKET_DELETION_PROHIBITED
2. AWS-GR_AUDIT_BUCKET_PUBLIC_READ_PROHIBITED
3. AWS-GR_AUDIT_BUCKET_PUBLIC_WRITE_PROHIBITED
4. AWS-GR_CLOUDTRAIL_CHANGE_PROHIBITED*
5. AWS-GR_CLOUDTRAIL_CLOUDWATCH_LOGS_ENABLED*
6. AWS-GR_CLOUDTRAIL_ENABLED*
7. AWS-GR_CLOUDTRAIL_VALIDATION_ENABLED*
8. AWS-GR_CLOUDWATCH_EVENTS_CHANGE_PROHIBITED*
9. AWS-GR_CONFIG_AGGREGATION_AUTHORIZATION_POLICY*
10. AWS-GR_CONFIG_AGGREGATION_CHANGE_PROHIBITED*
11. AWS-GR_CONFIG_CHANGE_PROHIBITED*
12. AWS-GR_CONFIG_ENABLED*
13. AWS-GR_CONFIG_RULE_CHANGE_PROHIBITED*
14. AWS-GR_CT_AUDIT_BUCKET_ENCRYPTION_CHANGES_PROHIBITED
15. AWS-GR_CT_AUDIT_BUCKET_LIFECYCLE_CONFIGURATION_CHANGES_PROHIBITED
16. AWS-GR_CT_AUDIT_BUCKET_LOGGING_CONFIGURATION_CHANGES_PROHIBITED
17. AWS-GR_CT_AUDIT_BUCKET_POLICY_CHANGES_PROHIBITED
18. AWS-GR_DETECT_CLOUDTRAIL_ENABLED_ON_SHARED_ACCOUNTS
19. AWS-GR_IAM_ROLE_CHANGE_PROHIBITED*
20. AWS-GR_LAMBDA_CHANGE_PROHIBITED*
21. AWS-GR_LOG_GROUP_POLICY*
22. AWS-AWS-GR_REGION_DENY*
23. AWS-GR_SNS_CHANGE_PROHIBITED*
24. AWS-GR_SNS_SUBSCRIPTION_CHANGE_PROHIBITED*

So now what?  We don't want to deploy any workloads into the audit, logging or management account.  That's not
what they are for.  We need to set things up for our workload accounts. But wait.  Not so fast.  We have a few more 
infrastructure components to set up as part of best practices.  There are a few more accounts that we need to provision
to help us.  Here's what we have:

Management Account: Centralized account for billing and for configuring our landing zone.  Here is where we will set up any
additional SCPs, create Organizational Units, provision accounts, etc.

Audit Account:  We have this setup so that we can set it as the administrative account for our landing zone for services
like Security Hub, GuardDuty, etc.  This way we have a centralized location to look at security events and a place we can
send auditors, so we don't have to give them access to everything.

Logging Account: All of our logs will go here. So if we set up any log monitoring, we can do so from a centralized location.

Additionally, we now want to set up:

Networking Account:  For our commercial side we want to set up centralized ingress and egress.  We don't want to have to 
configure an Internet Gateway or NAT gateway in a bunch of accounts.  If we run this centrally we can control traffic flow.
We will also use this account to centralize VPC gateway and service endpoints.  This will keep all our traffic traversing
the AWS network and not hopping out and back in.

DevTools Account: This account we will use to have the code repositories for our workloads and our CI/CD pipelines.

Development Account:  We will set this up initially as a testing environment.  This will be the target of any initial CI/CD
pipelines we create in the DevTools account.

TODO: INSERT_DIAGRAM_HERE

But before we get going with all that.  Lets setup some more tooling to help us out with our landing zone management.  A while
back there was a tool called [Customizations for AWS Control Tower (CfCT)](https://docs.aws.amazon.com/controltower/latest/userguide/cfct-overview.html.
This was a set of CloudFormation templates that you could customize to deploy changes into Control Tower using a pipeline.
On top of that you would be able to deploy [AWS Security Reference Architecture (AWS SRA)](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/welcome.html)
and you would have a pretty good set up.  Recently though, there has been an effort at AWS to consolidate and modernize this.
The end result is [Landing Zone Accelerator on AWS](https://aws.amazon.com/solutions/implementations/landing-zone-accelerator-on-aws/).
This is an open source project built using AWS CDK that abstracts configuring the landing zone environment by letting you
use yaml files to pass in configuration (as opposed to writing CloudFormation like with CfCt).

Moving forward, rather than writing `Landing Zone Accelerator on AWS`, I will use 'LZA'.

The implementation guide can be found [here](https://docs.aws.amazon.com/solutions/latest/landing-zone-accelerator-on-aws/solution-overview.html) and
the source code [here](https://github.com/awslabs/landing-zone-accelerator-on-aws).  Let's set this up now.  Since we are going
to be running workloads in both the standard and GovCloud (US) regions, we are going to configure our environment based
on what is described as [`Option 1`](https://docs.aws.amazon.com/solutions/latest/landing-zone-accelerator-on-aws/deployment-options-for-aws-govcloud-us-workloads.html) in the Implementation Guide.

### Deploy Landing Zone Accelerator on AWS

The following will be performed in the management account in the AWS standard (commercial) region.  You should have already 
created an admin account that you can login to that account with.  Remember, using the root account to perform admin
tasks is a "No! No!".

1. Create a GitHub personal access token.  You need this so that the LZA solution can check the public repository in GitHub
for any updates.  This is part of the pipeline process.  It uses this to automatically poll the public repo.  It doesn't push or share anything.
If your organization is uncomfortable with this, you can mirror or create a copy of the repo into AWS CodeCommit.
2. In the management account, open Secrets Manager
3. Click on `Store a new secret`
4. Select `Other type of secret`
5. Select `Plaintext`
6. Delete the contents of the box, usually `{"":""}`
7. Replace it with your GitHub token
8. For encryption key you can set it to `aws/secretsmanager`, the AWS managed key or use the that was created when creating control tower 
9. For the secret name use `accelerator/github-token`
10. Add a description.  Something like `Used by Landing Zone Accelerator to access GitHub` will work.
11. You can add a tag.  I like to know when things are manually created so I add a key of "manually_created" with a value of `True` (note the capital 'T').
12. Click `Next`
13. Click `Next`
14. Click `Store`. If you are presented with the list of Secrets but don't see the one you just created, click on the refresh button.  It's there.
    
The remaining [prerequisites](https://docs.aws.amazon.com/solutions/latest/landing-zone-accelerator-on-aws/prerequisites.html) are already done.  We did that when we initially set up Control Tower.

16. Open the CloudFormation Console
17. Ensure that `Template is ready` is selected
18. For `Specific template` select `Amazon S3 URL`
19. In the `Amazon S3 URL` field enter `https://solutions-reference.s3.amazonaws.com/landing-zone-accelerator-on-aws/latest/AWSAccelerator-InstallerStack.template`
20. Click `Next`
21. For `Stack Name` use `AWSAccelerator-InstallerStack`
22. Look for `Manual Approval Stage` and enter an email address that should be notified for the manual approval stage of the pipeline.
23. Enter your management account email
24. Enter your log account email
25. Enter your audit account email
26. Everything else can remain as is
26. Click `Next`
27. Click `Next`
28. Check the box next to `I acknowledge that AWS CloudFormation might create IAM resources.`
29. Click `Submit`

And now, we wait. A while.  You can keep an eye on the CloudFormation console until you see your `AWSAccelerator-Installer`
stack show `CREATE_COMPLETE`.  

### BUT WAIT!  THERE'S MORE.
Once your stack shows `CREATE_COMPLETE`, open the AWS CodePipeline console.  You should see a pipeline caled `AWSAccelerator-Installer`
with a status of `In progress`.  This is now going to install the rest of the solution for you.  The installer pipeline is pulling
the LZA code from GitHub and executing the CDK.  Once the `Install` stage of that pipeline completes, you will see another
pipeline running.   That pipeline, the second one, is the one we will primarily be working with.  This is the pipeline 
that will configure the landing zone based on the configuration files.  The first run of the pipeline is just deploying
an skeleton set of configuration files.  Once the `AWSAccelerator-Installer` pipeline is complete, you can find those
files in CodeCommit.

