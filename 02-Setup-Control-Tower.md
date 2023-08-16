### Create KMS Key
We want to `encrypt all the things.`  Control Tower allows for you to setup encryption for AWS CloudTrail and AWS Config and we want
to do this.  Create a KMS key with the policy below.  Replace `ACCOUNT_ID` with the account id of your account.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
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
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
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
                "AWS": "arn:aws:iam::ACCOUNT_ID:root"
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

### Setup Control Tower

1. In the search bar in the console type `Organizations` and open the Organizations console.
2. Click on the `Create an organization` button
3. In the root account mailbox, you will receive an email asking you to verify the email address. Clicking on the button
in the email will open the `Organizations` console with a green bar at the top letting you know the email is confirmed.
4. In the search bar in the console type `Control Tower` and open the Control Tower console.
5. Click on the `Set up landing zone` button
6. Under `Region deny setting` select `Enabled`
7. If you plan on running in multiple regions, select them in the `Select additional regions for governance` box
8. Click `Next`
9. In the `Additional OU` box, change `Sandbox` to `Infrastructure`.  We are doing this because the Landing Zone Accelerator 
requires the OU for `Infrastructure` to exist before you can run its setup.
10. Click `Next`
11. In the `Log archive account` box, enter the email address that you are going to use for the account.  Remember, plus notation if 
you can use it.  For example: root-account+log@my-domain.com
12. In the account name, remove the space between `Log` and `Archive` so that it looks like `LogArchive`
13. In the `Audit Account` box, enter the email address to use as the root user for that account
14. Click `Next`
15. On the `Additional configurations` page scroll to the bottom and under `KMS Encryption`, select `Enable and customize encryption settings`
16. Select the KMS key you previously created
17. Click `Next`
18. On the `Review and set up landing zone` page scroll to the button and check the box for "I understand the permissions..."
19. Click `Set up landing zone`

You should then see a blue banner at the top of the page indicating the progress of the landing zone creation.  It says it is going to take about
60 minutes to complete but it will be a little bit shorter than that.  For now, we wait.  There isn't anything we can do until we have this setup.
You will receive several emails while AWS does things in the background. Keep an eye out for those.

