Now that both the `AWSAccelerator-Installer` and `AWSAccelerator-Pipeline` have run successfully, we can keep going with
configuring our landing zone.  But what happened?  What did we get?  Well, we know that LZA is built on CDK, and we know
that CDK builds out CloudFormation.  If you didn't know this before, you know it now.  Let's hop over to the CloudFormation
console in our management account to see what happened.

In the CloudFormation Console, if you click on `Stacks`, you should see somewhere about 22 stacks.  Stacks are displayed
from the bottom to the top by order of deployment.  So if you scroll all the way to the bottom, that was the first
stack installed, all the way at the top is the last stack installed.   In the list, anything prefixed with `AWSAccelerator`
was installed by the pipelines except the `AWSAccelerator-Installer` stack.  We did that one to kick things 
off.

### Stacks created by Landing Zone Accelerator
1. AWSAccelerator-CDKToolkit - Prepares the account to be able to run CDK 
2. AWSAccelerator-PipelineStack-ACCOUNT_ID-REGION 
3. AWSAccelerator-PrepareStack-ACCOUNT_ID-REGION 
4. AWSAccelerator-AccountsStack-ACCOUNT_ID-REGION 
5. AWSAccelerator-DependenciesStack-ACCOUNT_ID-REGION 
6. AWSAccelerator-KeyStack-ACCOUNT_ID-REGION 
7. AWSAccelerator-LoggingStack-ACCOUNT_ID-REGION 
8. AWSAccelerator-OrganizationsStack-ACCOUNT_ID-REGION 
9. AWSAccelerator-NetworkPrepStack-ACCOUNT_ID-REGION 
10. AWSAccelerator-SecurityStack-ACCOUNT_ID-REGION 
11. AWSAccelerator-OperationsStack-ACCOUNT_ID-REGION 
12. AWSAccelerator-SecurityResourcesStack-ACCOUNT_ID-REGION 
13. AWSAccelerator-NetworkVpcStack-ACCOUNT_ID-REGION 
14. AWSAccelerator-NetworkVpcEndpointsStack-ACCOUNT_ID-REGION 
15. AWSAccelerator-NetworkVpcDnsStack-ACCOUNT_ID-REGION 
16. AWSAccelerator-NetworkAssociationsGwlbStack-ACCOUNT_ID-REGION 
17. AWSAccelerator-NetworkAssociationsStack-ACCOUNT_ID-REGION 
18. AWSAccelerator-CustomizationsStack-ACCOUNT_ID-REGION 
19. AWSAccelerator-FinalizeStack-ACCOUNT_ID-REGION

### Security Notifications
During the pipeline execution, you should expect to get a few notifications with the subject of `Config Rules Compliance Change`.
The message body will JSON and triggered when the LZA creates logging buckets.  All of them will have a `complianceType` of
`COMPLIANT`.

You should see the following:
1. GR_AUDIT_BUCKET_PUBLIC_WRITE_PROHIBITED     
- For `aws-accelerator-s3-access-logs-ACCOUNT_ID-REGION` created by LZA in the centralized logging account.    
- For `aws-accelerator-central-logs-ACCOUNT_ID-REGION` created by LZA in the centralized logging account.    
- For `aws-accelerator-elb-access-logs-ACCOUNT_ID-REGION` created by LZA in the centralized logging account.    
- For `aws-accelerator-s3-access-logs-ACCOUNT_ID-REGION` created by LZA in the audit account.    

2. AWS-GR_AUDIT_BUCKET_PUBLIC_READ_PROHIBITED    
- For `aws-accelerator-s3-access-logs-ACCOUNT_ID-REGION` created by LZA in the centralized logging account.    
- For `aws-accelerator-central-logs-ACCOUNT_ID-REGION` created by LZA in the centralized logging account.    
- For `aws-accelerator-elb-access-logs-ACCOUNT_ID-REGION` created by LZA in the centralized logging account.    
- For `aws-accelerator-s3-access-logs-ACCOUNT_ID-REGION` created by LZA in the audit account.    



### Configuring OUs
LZA does a lot of things for us but it doesn't create Organizational Units (OUs).  For what I am building here, I know we are going
to need a few OUs created in AWS Organizations.

1. Open the Organizations console and make sure you on the `AWS accounts` page.
2. Click the checkbox next to `Root` 
3. Select `Actions` > `Create new`
4. For `Organizational unit name` enter `GovCloud`
5. For tags, I like to know if things were manually created, so I add a key of "manually_created" with a value of "True", you can add it if you want.
6. Click on `Create organizational unit`
7. You should see the new `GovCloud` OU listed under the `Root`

When GovCloud (US) accounts are created, they are associated with a commercial (or standard) partition account for billing purposes.
There is a 1:1 relationship.  So for every GovCloud account you will want to create, you will have a standard partition account. The GovCloud
OU is where you will put all the standard accounts associated with the GovCloud (US) accounts.  This OU will have restrictions on it
to disable the ability of any services to run in them.  The accounts exist for billing purposes only, you DO NOT want to
run workloads in them.  Putting the standard partition accounts into the OU and locking it down will prevent them from being used for
workloads.

Now, there is ANOTHER way that you can create Organizational Units.  We created the GovCloud OU using the Organizations
console to set up a demonstration for what happens when you configure things outside of Control Tower.  Control Tower is our
primary management tool for our landing zone.  So hop on over to the Control Tower console.  When the dashboard comes up
you will notice that under `Organizational units` it says `2` even though we just created the GovCloud OU.  Click on the number
`2`. Then under the `Registered organization units` section click on `View all organizational units`.

You should see the `GovCloud` OU that we just created showing a state of `Unregistered`.  Select the circle to the left of
`GovCloud`.  Then select `Actions` and then `Register organizational unit`.  This is going to let Control Tower know we want
it to manage this OU.  On the next page, scroll down to the bottom, check the box next to the `I understand the risks...` and 
click on `Register OU`.

You will get a blue status bar at the top of the page indicating that Control Tower is registering the OU.  It says it will take about an 
hour, but it is much less than that.   You will know it is complete when you sees a green status bar at the top of the page
and the `State` of the `GovCloud` OU will show `Registered`.

Now let's create a few more OUs that we know we will need.  I mentioned that I will have an account that will house my
code repositories and pipelines.  I'll create an OU called `DevTools` for that.  Then I want to have a target account.  An 
account that I can use to push solutions into.  So I'll create an OU called `Workload-A` and under that I'll create some stage OUs.
So under `Workload-A`, I'll have a `Development`, `Stage` and `Production` OU.  

At least, this is what I'm thinking right now.  There is no 'right' or 'wrong' here.  Only the best decision to be made at the 
time the decision needs to be made.  Nothing is written in stone, and it can be changed later. By configuring the structure
around my workloads, I can more easily isolate that workload if there is a problem and more easily understand how much it costs
to run that workload.  We will end up with a structure similar to the diagram below.    
    
![01-configure-lza.png](images%2F02-configure-lza.png)    

The accounts shown in the `DevTools` and `Development` OU have dashed lines to denote they have not been created yet. For each
stage, I'll have accounts specific to the services.  For example, the current plan is to create an account to house all databases in order
to separate the data layer with an account boundary.  Any specific AWS services that are used, like IoT, I would consider
if it is best to separate that service into its own account.

Let's go ahead and create the OU structure using the Control Tower console.
   
### Create additional OUs
1. Open the Control Tower console if you are not there already
2. Select `Organization` from the menu
3. Click `Create resources`
4. Click `Organziational unit`
5. For `OU name` enter `DevTools`
6. For `Parent OU` enter `Root`
7. Click `Add`

The same indicators will appear as when the GovCloud OU was registered.  Once complete, add a `Workload-A` OU under root
and then the `Development`, `Stage` and `Production` OUs under `Workload-A`.  When completed, we will have a structure like
the one below:

![03-configure-lza.png](images%2F03-configure-lza.png)    
    
Before we get too deep into things, there _might_ be an issue with a new account in which AWS will limit what you can do.
They refer to this as a `Containment` score or you might hear it referred to as a `trust` score.  More info [here](https://towardsaws.com/containment-score-of-aws-3a893231e948).
Let's just try to avoid this all together.
    
1. Open the EC2 console
2. Select `Launch instance`
3. Set the number of instances to `5`
4. For `Name` put anything you like.  I just put `test`
5. Scroll down to `Key pair (login)` and select `Proceed without a key pair (Not Recommended)` - We are not logging into these machines.
6. Under `Network settings` click on `Edit`
7. For `Auto-assign public IP` choose `Disable`
8. Select `Select existing security group`
9. In the drop-down select the `default` security group 
10. Leave everything else the same and click on `Launch instance`
11. Click `View all instances`
    
Wait for all the instances to spin up and let them run for about 45 minutes.  Then you can terminate them all.
    

### Clone CodeCommit repository
Next up we need to clone our CodeCommit repository that contains our skeleton config files that were put there by the LZA 
installer. [Need help?](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-git-remote-codecommit.html)
    
We are going to be using some configuration provided in the LZA GitHub repository under [`references/sample-configurations`](https://github.com/awslabs/landing-zone-accelerator-on-aws/tree/main/reference/sample-configurations).
Right now, our cloned repo from CodeCommit should look like this:

![05-configure-lza.png](images%2F05-configure-lza.png)    
    
Let's get started.
1. In a browser, open  [reference/sample-configurations/aws-best-practices](https://github.com/awslabs/landing-zone-accelerator-on-aws/tree/main/reference/sample-configurations/aws-best-practices) from the samples-configurations in the LZA repo
2. In the local repo, create a directory called `service-control-policies`
3. Create the `guardrails-1.json`, `guardrails-2.json` and `quarantine.json` from the `aws-best-practices` location in GitHub into your local repo
4. Create the `lockdown-govCloud-accounts.json` with contents from [`reference/sample-configurations/aws-best-practices-govcloud-us/commercial-config/service-control-policies/lockdown-govCloud-accounts.json`](https://github.com/awslabs/landing-zone-accelerator-on-aws/blob/main/reference/sample-configurations/aws-best-practices-govcloud-us/commercial-config/service-control-policies/lockdown-govCloud-accounts.json)
5. Create a directory called `dynamic-partitioning`
6. Create the `log-filters.json` file in the directory with the contents from [reference/sample-configurations/aws-best-practices/dynamic-partitioning/log-filters.json](https://github.com/awslabs/landing-zone-accelerator-on-aws/blob/main/reference/sample-configurations/aws-best-practices/dynamic-partitioning/log-filters.json)
7. Create a directory called `iam-policies`
8. Create a file called `boundary-policy.json` and place the contents from [reference/sample-configurations/aws-best-practices/iam-policies/boundary-policy.json](https://github.com/awslabs/landing-zone-accelerator-on-aws/blob/main/reference/sample-configurations/aws-best-practices/iam-policies/boundary-policy.json) in the file
9. Create a directory called `tagging-policies`
10. Create a file called `org-tag-policy.json` and place the contents from [reference/sample-configurations/aws-best-practices/tagging-policies/org-tag-policy.json](https://github.com/awslabs/landing-zone-accelerator-on-aws/blob/main/reference/sample-configurations/aws-best-practices/tagging-policies/org-tag-policy.json)
11. Create a directory called `backup-policies`
12. Create a file called `backup-plan.json` and place the contents from [reference/sample-configurations/aws-best-practices/backup-policies/backup-plan.json](https://github.com/awslabs/landing-zone-accelerator-on-aws/blob/main/reference/sample-configurations/aws-best-practices/backup-policies/backup-plan.json)
13. Create a directory called `vpc-endpoint-policies`
14. Create the `default.json` and `ec2.json` files with contents from [here](https://github.com/awslabs/landing-zone-accelerator-on-aws/tree/1614a01824c5a43f97fadfb8ec0c3627a0f343dd/reference/sample-configurations/aws-best-practices/vpc-endpoint-policies)


### Configure accounts-config.yaml
1. Open the accounts-config.yaml file
2. Remove the `[]` after `workloadAccounts:`
3. Add the following on the next line. Don't forget to update the email addresses.
```yaml
  - name: Network
    description: Centralized Network account for commercial partition
    email: <network>@example.com  <----- UPDATE EMAIL ADDRESS
    organizationalUnit: Infrastructure
  - name: DevTools
    description: Centralized DevTools account for commercial partition
    email: <devtools>@example.com  <----- UPDATE EMAIL ADDRESS
    organizationalUnit: DevTools
  - name: Workload-A-Database
    description: Database account for Workload-A in commercial partition
    email: <workload-a-db>@example.com  <----- UPDATE EMAIL ADDRESS
    organizationalUnit: Workload-A/Development
  - name: Workload-A-Application
    description: Application account for Workload-A in commercial partition
    email: <workload-a-application>@example.com  <----- UPDATE EMAIL ADDRESS
    organizationalUnit: Workload-A/Development    
```
   
### Configure global-config.yaml
1. Open the global-config.yaml file
2. Change the first line from `homeRegion: us-east-1` to `homeRegion: &HOME_REGION REGION` Changing REGION to your home region
3. Add the following after `cdkOptions` and before `controlTower:` at the same level. Don't forget to edit the email address
```yaml
snsTopics:
  deploymentTargets:
    organizationalUnits:
      - Root
  topics:
    - name: Security
      emailAddresses:
        - <security-notifications>@example.com  <----- UPDATE EMAIL ADDRESS
```
4. Under the logging section, remove the `[]` after `accountTrails:` and add the following:
```yaml
      - name: AccountTrail
        regions:
          - *HOME_REGION
        deploymentTargets:
          accounts: []
          organizationalUnits:
            - Root
        settings:
          multiRegionTrail: true
          globalServiceEvents: true
          managementEvents: true
          s3DataEvents: true
          lambdaDataEvents: true
          sendToCloudWatchLogs: true
          apiErrorRateInsight: false
          apiCallRateInsight: false
```
5. Replace `sessionManager` section from
```yaml
  sessionManager:
    sendToCloudWatchLogs: false
    sendToS3: false
    excludeRegions: []
    excludeAccounts: []
    lifecycleRules: []
    attachPolicyToIamRoles: []
```
to:
```yaml
  sessionManager:
    sendToCloudWatchLogs: false
    sendToS3: true
    lifecycleRules:
      - enabled: true
        abortIncompleteMultipartUpload: 7
        expiration: 730
        noncurrentVersionExpiration: 730
    attachPolicyToIamRoles:
      - EC2-Default-SSM-AD-Role
```
6. After the `sessionManager` section add the following:
```yaml
  cloudwatchLogs:
    dynamicPartitioning: dynamic-partitioning/log-filters.json 
```
7. After the `cloudwatchLogs:` section add the following to confgiure lifecycle policies on your logging buckets
```yaml
  accessLogBucket:
    lifecycleRules:
      - enabled: true
        abortIncompleteMultipartUpload: 7
        expiration: 1000
        noncurrentVersionExpiration: 1000
        transitions:
          - storageClass: GLACIER_IR
            transitionAfter: 365
        noncurrentVersionTransitions:
          - storageClass: GLACIER_IR
            transitionAfter: 365
  centralLogBucket:
    lifecycleRules:
      - enabled: true
        abortIncompleteMultipartUpload: 7
        expiration: 1000
        noncurrentVersionExpiration: 1000
        transitions:
          - storageClass: GLACIER_IR
            transitionAfter: 365
        noncurrentVersionTransitions:
          - storageClass: GLACIER_IR
            transitionAfter: 365
  elbLogBucket:
    lifecycleRules:
      - enabled: true
        abortIncompleteMultipartUpload: 7
        expiration: 1000
        noncurrentVersionExpiration: 1000
        transitions:
          - storageClass: GLACIER_IR
            transitionAfter: 365
        noncurrentVersionTransitions:
          - storageClass: GLACIER_IR
            transitionAfter: 365 
```
8. Let's add some billing reporting and alerting.  Add the following after the lifecycle policies:
```yaml
reports:
  costAndUsageReport:
    compression: Parquet
    format: Parquet
    reportName: accelerator-cur
    s3Prefix: cur
    timeUnit: DAILY
    refreshClosedReports: true
    reportVersioning: CREATE_NEW_REPORT
    lifecycleRules:
      - enabled: true
        abortIncompleteMultipartUpload: 7
        expiration: 730
        noncurrentVersionExpiration: 730
  budgets:
    - deploymentTargets:
        accounts:
          - Management
      name: accel-budget
      timeUnit: MONTHLY
      type: COST
      amount: 2000
      includeUpfront: true
      includeTax: true
      includeSupport: true
      includeSubscription: true
      includeRecurring: true
      includeOtherSubscription: true
      includeDiscount: true
      includeCredit: false
      includeRefund: false
      useBlended: false
      useAmortized: false
      unit: USD
      notifications:
        - type: ACTUAL
          thresholdType: PERCENTAGE
          threshold: 100
          comparisonOperator: GREATER_THAN
          subscriptionType: EMAIL
          address: <budget-notifications>@example.com  <----- UPDATE EMAIL ADDRESS
        - type: ACTUAL
          thresholdType: PERCENTAGE
          threshold: 90
          comparisonOperator: GREATER_THAN
          subscriptionType: EMAIL
          address: <budget-notifications>@example.com  <----- UPDATE EMAIL ADDRESS
        - type: ACTUAL
          thresholdType: PERCENTAGE
          threshold: 80
          comparisonOperator: GREATER_THAN
          subscriptionType: EMAIL
          address: <budget-notifications>@example.com  <----- UPDATE EMAIL ADDRESS
        - type: ACTUAL
          thresholdType: PERCENTAGE
          threshold: 75
          comparisonOperator: GREATER_THAN
          subscriptionType: EMAIL
          address: <budget-notifications>@example.com  <----- UPDATE EMAIL ADDRESS
        - type: ACTUAL
          thresholdType: PERCENTAGE
          threshold: 50
          comparisonOperator: GREATER_THAN
          subscriptionType: EMAIL
          address: <budget-notifications>@example.com  <----- UPDATE EMAIL ADDRESS
```
9. Lastly, add the `backup` section
```yaml
backup:
  vaults:
    - name: BackupVault
      deploymentTargets:
        organizationalUnits:
          - Root
```
### Configure iam-config.yaml
1. Open the iam-config.yaml file
2. Leave `providers: []` as it is
3. Replace `policySets:` with the following:
```yaml
policySets:
  - deploymentTargets:
      organizationalUnits:
        - Root
    policies:
      - name: Default-Boundary-Policy
        policy: iam-policies/boundary-policy.json 
```
4. Replace `roleSets:` with the following:
```yaml
roleSets:
  - deploymentTargets:
      organizationalUnits:
        - Root
    roles:
      - name: EC2-Default-SSM-AD-Role
        instanceProfile: true
        assumedBy:
          - type: service
            principal: ec2.amazonaws.com
        policies:
          awsManaged:
            - AmazonSSMManagedInstanceCore
            - AmazonSSMDirectoryServiceAccess
            - CloudWatchAgentServerPolicy
      # This role is utilized by the Backup Plans defined in global-config.yaml
      # We create this role in every account where we plan to have Backup Plans
      # and Backup Vaults
      - name: Backup-Role
        assumedBy:
          - type: service
            principal: backup.amazonaws.com
        policies:
          awsManaged:
            - service-role/AWSBackupServiceRolePolicyForBackup
            - service-role/AWSBackupServiceRolePolicyForRestores
      # This role is utilized by the Budgets Config defined in global-config.yaml
      # We create this role in every account where we plan to have AWS Budgets
      # A service linked role is not available for AWS Budgets
      - name: Budgets-Role
        assumedBy:
          - type: service
            principal: budgets.amazonaws.com
        policies:
          awsManaged:
            - AWSBudgetsActionsWithAWSResourceControlAccess
```
5. Replace `groupSets:` with the following:
```yaml
groupSets:
  - deploymentTargets:
      organizationalUnits:
        - Root
    groups:
      - name: Administrators
        policies:
          awsManaged:
            - AdministratorAccess 
```
6. Replace `userSets` with the follwing:
```yaml
 userSets:
  - deploymentTargets:
      accounts:
        - Management
    users:
      - username: breakGlassUser01
        group: Administrators
        boundaryPolicy: Default-Boundary-Policy
      - username: breakGlassUser02
        group: Administrators
        boundaryPolicy: Default-Boundary-Policy
```

### Configure organization-config.yaml
1. Open the organization-config.yaml file
2. Ensure you have all the OUs that were previously created manually
```yaml
organizationalUnits:
  - name: Security
  - name: Infrastructure
  - name: Workload-A
  - name: Workload-A/Development
  - name: Workload-A/Stage
  - name: Workload-A/Production
  - name: DevTools
  - name: GovCloud 
```
3. After the `organizationalUnits:` section, add the following
```yaml
quarantineNewAccounts:
  enable: true
  scpPolicyName: Quarantine 
```
4. Replace the `serviceControlPolicies:` section with
```yaml
serviceControlPolicies:
  - name: AcceleratorGuardrails1
    description: >
      Accelerator GuardRails 1
    policy: service-control-policies/guardrails-1.json
    type: customerManaged
    deploymentTargets:
      organizationalUnits:
        - Infrastructure
  - name: AcceleratorGuardrails2
    description: >
      Accelerator GuardRails 2
    policy: service-control-policies/guardrails-2.json
    type: customerManaged
    deploymentTargets:
      organizationalUnits:
        - Infrastructure
  - name: Quarantine
    description: >
      This SCP is used to prevent changes to new accounts until the Accelerator
      has been executed successfully.
      This policy will be applied upon account creation if enabled.
    policy: service-control-policies/quarantine.json
    type: customerManaged
    deploymentTargets:
      organizationalUnits: [] 
```
5. Replace the `taggingPolicies:` section with 
```yaml
# https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_tag-policies.html
taggingPolicies:
  - name: TagPolicy
    description: Organization Tagging Policy
    policy: tagging-policies/org-tag-policy.json
    deploymentTargets:
      organizationalUnits:
        - Root 
```
6. Replace the `backupPolicies:` section with
```yaml
backupPolicies:
  - name: BackupPolicy
    description: Organization Backup Policy
    policy: backup-policies/backup-plan.json
    deploymentTargets:
      organizationalUnits:
        - Root
```

### Configure network-config.yaml
1. open network-config.yaml
2. Add `homeRegion: &HOME_REGION REGION` as the first line of the file. Change `REGION` to match the region you have been working with.
3. In the `defaultVpc:` section, change `delete` to `true`
4. Replace `endpointPolicies:` with
```yaml
endpointPolicies:
  - name: Default
    document: vpc-endpoint-policies/default.json
  - name: Ec2
    document: vpc-endpoint-policies/ec2.json 
```
5. At the very bottom of the file, after `vpcs: []` add
```yaml
vpcFlowLogs:
  trafficType: ALL
  maxAggregationInterval: 600
  destinations:
    - cloud-watch-logs
  destinationsConfig:
    cloudWatchLogs:
      retentionInDays: 30
  defaultFormat: false
  customFields:
    - version
    - account-id
    - interface-id
    - srcaddr
    - dstaddr
    - srcport
    - dstport
    - protocol
    - packets
    - bytes
    - start
    - end
    - action
    - log-status
    - vpc-id
    - subnet-id
    - instance-id
    - tcp-flags
    - type
    - pkt-srcaddr
    - pkt-dstaddr
    - region
    - az-id
    - pkt-src-aws-service
    - pkt-dst-aws-service
    - flow-direction
    - traffic-path 
```

We are going to stop here with a basic config of the network-config.yaml file.  Later, we will dig a little deeper into 
the configuration and setup VPCs and routing and all the other fun stuff.  When we do that, we want it to be our central
focus.  Right now, we are (sort of) running through the house and turning on all the lights we want on.

### Configure security-config.yaml

1. Open the security-config.yaml file
2. Add `homeRegion: &HOME_REGION REGION` as the first line of the file. Change `REGION` to match the region you have been working with.
3. Change `ebsDefaultVolumeEncryption`, `enable:` to `true`
4. Change `s3PublicAccessBlock`, `enable` to `true`
5. Change `scpRevertChangesConfig`, `enable` to `true`
6. Add `snsTopicName: Security` under `scpRevertChangesConfig`
7. Set `enable` under `guardduty` to `true`
8. Set `enable` under `guardduty > S3Protection` to `true`
9. Set `enable` under `guardduty > exportConfiguration` to `true`
10. Set `overrideExisting` under `guardduty > exportConfiguration` to `true`
11. Set `enable` under `securityHub` to `true`
12. Set `regionAggregation` under `securityHub` to `true`
13. Set `enable` under `accessAnalyzer` to `true`
14. Set `enableConfigurationRecorder` under `awsConfig` to `true`
15. Add `enableDeliveryChannel: true` under `awsConfig`

We are going to stop here with the security-config.yaml for the same reasons as the network-config.yaml.  We will come back
to it later and add more configurations.

Let's go ahead and check the files back in and trigger the pipeline to run again.  All the updated files up to this point
are in the 01-sample-configurations directory.

