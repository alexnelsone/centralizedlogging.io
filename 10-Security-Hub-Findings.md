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


Follow these instructions to remediate:    
https://docs.aws.amazon.com/sns/latest/dg/sns-topic-attributes.html