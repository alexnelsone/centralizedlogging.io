Let's go ahead and filter the Security Hub findings per account.  In the `Add filter` box, we'll filter for 
`AWS Account ID` for our `Audit` account and a `Compliance Status` of `FAILED`.



## ECR.1 ECR private repositories should have image scanning configured
The first one we see is `ECR.1 ECR private repositories should have image scanning configured`.  This is being reported
by having `AWS Foundational Security Best Practices`.  Later, we are going to disable this and make sure we only have
`NIST SP 800-53 rev 5` enabled but this rule is also evaluated with those controls so let's take care of it.

Here is a screenshot of the finding's details.

![31-configure-lza.png](images%2F31-configure-lza.png)    
   
   
We can see that the Resource showing as non-compliant is prefixed with `cdk-accel`.  This repository isn't created by
LZA but by the bootstrap process of CDK which LZA uses to deploy its resources.  

1. Open the `Amazon Elastic Container Registry` console
2. Select `Scanning` under `Private registry`
3. Select the checkbox next to `Scan on push all repositories`
4. click `Save`


If you click on the `Rule(s)` drop down, you can select the rule from the details pane    
![32-configure-lza.png](images%2F32-configure-lza.png)    
    
Clicking this will open a new tab with the details of the rule.  From the `Actions` drop down, select `Re-evaluate`
    
![33-configure-lza.png](images%2F33-configure-lza.png)    
    
While it is being evaluated, if you are subscribed to `aws-controltower-aggregateSecurityNotifications`, you should get 
a notification indicating a status of `Compliant`.  The image below shows the relevant information from the notification.
In the  `NewEvaluationResult` boxed in red, you can see the Config rule name, the resource ID of the repository and
the new compliance type of `COMPLIANT`. Highlighted in blue is the `oldEvaluationResult` and the information to compare. 
There we can see that it had a compliance type of `NON_COMPLIANT`.
    
![34-configure-lza.png](images%2F34-configure-lza.png)    
    
This modification will need to be made in all accounts that were created as part of the Landing Zone build out and then again
for every account vended using the accelerator.  Once the modification is made on all accounts, you will need to wait a few hours for Security Hub
to report back to the centralized Audit account before seeing the change.  You can see the remediation in Security Hub
of the affected account, however.


## IAM.1 IAM policies should not allow full "*" administrative privileges

This one generally appears for the `Default-Boundary-Policy` initially after deploying.  If you used the `Default-Boundary-Policy`
from the LZA GitHub repo, the permissions are not _tight_ enough, and you should follow the instruction [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_iam_mfa-selfmanage.html).

If you used the example `boundary-policy.json` in this repo, you will not see this error. Updating the policy and re-running
the pipeline with the new `boundary-policy.json` will result in the following to `aws-controltower-aggregateSecurityNotifications`.


![35-configure-lza.png](images%2F35-configure-lza.png)

## SNS.2 Logging of delivery status should be enabled for notification messages sent to a topic
Seeing this in regard to `aws-controltower-AllConfigNotifications`. [Additional Info](https://docs.aws.amazon.com/securityhub/latest/userguide/sns-controls.html#sns-2)

In the `iam-config.yaml` file, the policies `SNS-Delivery-Status-Logging-Failure-Policy` and `SNS-Delivery-Status-Logging-Sucess-Policy`
and their associated roles, `SNS-Delivery-Status-Logging-Failure-Role` and `SNS-Delivery-Status-Logging-Failure-Role` were set up
to assist with setting up logging.  Because the SNS topic is created outside LZA, it cannot be managed by the LZA
confiuguration files.  This update will need to occur manually, and you will need to perform it as the `ControlTowerExecutionRole`
due to Service Control Policies that have an explicit deny for any other user.

The Audit account has 4 topics to modify.
The LogArchive account has 2 topics to modify.

When editing the topics in the SNS console, the IAM role is not a drop down.  You will need the arn for the success
and failure roles.

Details for remediation:    
https://docs.aws.amazon.com/sns/latest/dg/sns-topic-attributes.html

1. Open the SNS console
2. Select the radio button for the topic to edit
3. Click on the `Edit` button
4. Expand `Delivery status logging - optional`
5. Select the protocols to log under `Log delivery status for these protocols`
6. Enter the IAM role for successful deliveries
7. Enter the IAM role for failed deliveries

![36-configure-lza.png](images%2F36-configure-lza.png)    
    
The security notification will show the following new and old evaluation result for each SNS topic:
    
![37-configure-lza.png](images%2F37-configure-lza.png)    
    
## SNS.1 SNS topics should be encrypted at-rest using AWS KMS
This is coming up for the Control Tower SNS topics `aws-controltower-AllConfigNotifications`. You will need to perform this
as the `AWSControlTowerExecution` Role.

1. Open the SNS console
2. select the `aws-controltower-AllConfigNotifications` topic radio button
3. Click `Edit`
4. Expand the `Encryption` section
5. Toggle the switch next to `Encryption`
6. Select the `alias/accelerator/kms/snstopic/key`
7. click the `Save changes` button

## Athena.1 Athena workgroups should be encrypted at rest
This was appearing in the Audit account.  Because we are not using Athena at the moment, we are going to set encryption
but also turn off the workgroup.

1. Open the Athena console
2. Select `Workgroups` from the menu (Under `Administration`)
3. Select the radio button next to the workgroup name
4. Select `Actions` > `Edit`
5. Scroll to `Query result configuration` and expand it
6. Select the checkbox for `Encrypt query results`
7. Select the `SSE_KMS` option
8. Select the key with the `accelerator/kms/s3/key` alias
9. Select the `Set SSE_KMS as minimum encryption` option
9. Click the `Save changes` button

The workgroup should now have a status of `Turned off` in the workgroup console.


## Account.1 Security contact information should be provided for an AWS account.

1. Login to your account as root user
2. From the menu in the top right, select the account and then select `Account`
3. Scroll to the `Alternate Contact` section and click `Edit`
4. Fill in the necessary information
5. Click `Save`

## ECR.3 ECR repositories should have at least one lifecycle policy configured
For examples on lifecycle policies for repositories, see [here](https://docs.aws.amazon.com/AmazonECR/latest/userguide/lifecycle_policy_examples.html).

For a simple one, you could use.  You can modify the number of days if you like.  This example comes from the documentation
linked above.
```json
{
    "rules": [
        {
            "rulePriority": 1,
            "description": "Expire images older than 14 days",
            "selection": {
                "tagStatus": "untagged",
                "countType": "sinceImagePushed",
                "countUnit": "days",
                "countNumber": 14
            },
            "action": {
                "type": "expire"
            }
        }
    ]
}
```

Because during initial setup of LZA, cdk creates the ECR repositories (prefixed with `cdk-accel-container-assets`), this policy
needs to be added by hand since the repository is not managed by LZA.

1. Open the Amazon Elastic Container Registry console
2. Select `Repositories`
3. Select the checkbox next to the repository prefixed with `cdk-accel-container-assets` 
4. Select `Actions` drop down and then `Lifecycle policies`
5. Select `Actions` and then `edit JSON`
6. In the edit box that appears, paste the JSON from above
7. Click `Save`    

The information on the lifecycle rule will automatically fill in the `Priority`, `Rule description` and `Summary`.
    
![38-configure-lza.png](images%2F38-configure-lza.png)    



## 2.1.2 Ensure S3 Bucket Policy is set to deny HTTP requests
Add the following to the bucket policy replacing BUCKET_ARN.


```yaml
        {
            "Sid": "AllowSSLRequestsOnly",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "BUCKET_ARN",
                "BUCKET_ARN/*"
            ],
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        }

```

