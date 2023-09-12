Now that the pipeline is completed, you now have your GovCloud accounts provisioned with their associated commercial accounts.
If you open `Organization` in the `Control Tower` console in the commercial management account, you should see something
similar to this.

![42-configure-lza.png](images%2F42-configure-lza.png)    
    
The accounts in the red box are what was created in the previous step.  These are the Commercial accounts associated with
your GovCloud accounts.  They are in the GovCloud OU which has an SCP attached to it to deny use of any services in these accounts.


    
## Get GovCloud Account Mappings

1. In the Commercial management account, open the `DynamoDB` console
2. Select `Tables` from the menu to list your tables    
![43-configure-lza.png](images%2F43-configure-lza.png)    
3. Look for the table that contains `govCloudAccountMappings` in the name and click on it    
![44-configure-lza.png](images%2F44-configure-lza.png)    
4. Click on the `Explore table Items` button. You should see a table containing the `commercialAccountId`, `accountName`,
and `govCloudAccountId`
![45-configure-lza.png](images%2F45-configure-lza.png)    
5. Copy the contents of this table to a text file or some place where you can see them later
6. You can now logout of the commercial management account


## Create Organization in GovCloud
We are now going to repeat the process of deploying LZA, but we will do it in the `GovCloud Management Account`. There
are a few differences, but the process is a little similar.

1. Login to the `GovCloud Management Account`
2. Navigate to the `Organizations` console
3. Click the `Create an organization` button
4. You should see a green banner across the top of the console showing `You successfully created an AWS organization.`


## Create KMS Key
Just like our commercial side accounts, we want to `encrypt all the things.`  Control Tower allows for you to set up 
encryption for AWS CloudTrail and AWS Config, and we want to do this.  

1. Open the `KMS Console` in the `Management account`
2. Click the `Create a key` button
3. Leave `Symmetric` and `Encrypt and Decrypt` selected
4. Click `Next`
5. For `Alias` enter `kms-key-for-lza`
6. For `Description` enter `KMS key used by LZA`
7. Click `Next`
8. For `Key administrators` select `Admin`
9. Click `Next`
10. For `Define key usage permissions` select `Admin`
11. Click `Next`
12. Replace the `Key policy`, with the one below.  Replace `ACCOUNT_ID` with the account id of your account.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws-us-gov:iam::ACCOUNT_ID:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws-us-gov:iam::ACCOUNT_ID:root"
            },
            "Action": [
                "kms:CancelKeyDeletion",
                "kms:Create*",
                "kms:Delete*",
                "kms:Describe*",
                "kms:Disable*",
                "kms:Enable*",
                "kms:Get*",
                "kms:List*",
                "kms:Put*",
                "kms:Revoke*",
                "kms:ScheduleKeyDeletion",
                "kms:TagResource",
                "kms:UntagResource",
                "kms:Update*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws-us-gov:iam::ACCOUNT_ID:root"
            },
            "Action": [
                "kms:Decrypt",
                "kms:DescribeKey",
                "kms:Encrypt",
                "kms:GenerateDataKey*",
                "kms:ReEncrypt*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow Cloudtrail and Config to encrypt/decrypt logs",
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "config.amazonaws.com",
                    "cloudtrail.amazonaws.com"
                ]
            },
            "Action": [
                "kms:Decrypt",
                "kms:GenerateDataKey*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow attachment of persistent resources",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws-us-gov:iam::ACCOUNT_ID:root"
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": "true"
                }
            }
        }
    ]
}
```
13. Click `Finish`

## Set up Secret Key for Github
This only needs to be completed if you will be pulling the LZA code from GitHub.
1. Open the Secrets Manager console
2. Click `Store a new secret`
3. Select `Other type of secret`
4. Under `Key/value pairs`, select `Plaintext`
5. Remove all text from the textbox and paste your GitHub Token in the box
6. For `Encryption key` leave `aws/secretsmanager` selected
7. Click Next
8. For `Secret name` enter `accelerator/github-token`
9. For `Description` you can put something like `Used by LZA to retrieve LZA code from GitHub public repo.`
10. Click `Next`
11. Scroll to the bottom of the page and select `Store`


## Run the LZA installer
1. Open the CloudFormation console
2. Select the `Create stack` dropdown and select `Create new stack`
3. Enter the URL to the LZA installer template `https://s3.amazonaws.com/solutions-reference/landing-zone-accelerator-on-aws/latest/AWSAccelerator-InstallerStack.template`
4. For Stack Name enter `AWSAccelerator-InstallerStack`
5. If you want to send notifications to a specific email for approvals, enter it in the `Manual Approval Stage notification email list`
6. Enter the email address of the management account for GovCloud
7. Enter the email address of the logging account for GovCloud
8. Enter the email address of the audit account for GovCloud
9. Change the `Control Tower Environment` to `No`
10. Click `Next`
11. Click `Next`
12. On the Review page, scroll to the bottom and click the checkbox next to `I acknowledge...`
13. Click `Submit`
14. Wait for the `AWSAccelerator-InstallerStack` stack to show `CREATE_COMPLETE`
15. Open the `CodePipeline` console and select the `AWSAccelerator-Installer` pipeline
16. Wait for the pipeline to complete. It is complete when both `Source` and `Install` stages are green
17. Select  `Pipelines` from the breadcrumbs menu, select the `AWSAccelerator-Pipeline`
18. Wait until the `Prepare` stage fails.  This is expected.