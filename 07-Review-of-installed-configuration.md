Now that our pipeline has run all the way through with some additional configuration, let's see what we got.  We created
4 new accounts.


| Account Name               | Purpose                                                                                                                 |
|----------------------------|-------------------------------------------------------------------------------------------------------------------------|
| Network                    | This is the account we will use as our central location for network services.                                           |
| DevTools                   | An account that will house our code repositories and CICD pipelines                                                     |
| Workload-A-Database-Dev    | An account to house databases for development environment for Workload-A                                                |
| Workload-A-Application-Dev | An account for development of applications for Workload-A.  It will access databases in Workload-A-Database-Dev account |


While the pipeline was running, a few emails came in.  

For each account we received:

1. Email saying "Welcome to Amazon Web Services"

![06-configure-lza.png](images%2F06-configure-lza.png)    

2. Email invitation to join IAM Identity Center.  This email is sent to the root email address used to create the account. 
We DO NOT have to accept this invite.

![07-configure-lza.png](images%2F07-configure-lza.png)

3. Email indicating that the AWS Account is ready

![08-configure-lza.png](images%2F08-configure-lza.png)

4. Email regarding AWS Cost Anomaly detection associated with the management account.

![10-configure-lza.png](images%2F10-configure-lza.png)    


In the `global-config.yaml` file we configured an SNS topic called security. This topic was created in our LogArchive account.
We received an email to confirm the subscription.

![09-configure-lza.png](images%2F09-configure-lza.png)

We also received several messages from the `Config Rules Compliance Change` SNS topic for each new resource created/modified.
Below is the list of alerts received from the pipeline run.

### Config Rule Compliance Change Alerts

| Resource Type              | Number of alerts |
|----------------------------|------------------|
| AWS::::Account             | 106              |
| AWS::Athena::WorkGroup     | 6                |
| AWS::CloudFormation::Stack | 129              |
| AWS::CloudTrail::Trail     | 18               |
| AWS::ECR::Repository       | 18               |
| AWS::IAM::Group            | 6                |
| AWS::IAM::Policy           | 108              |
| AWS::IAM::Role             | 441              |
| AWS::Kinesis::Stream       | 1                |
| AWS::KMS::Key              | 74               |
| AWS::Lambda::Function      | 238              |
| AWS::S3::Bucket            | 109              |
| AWS::SNS::Topic            | 28               |
| TOTAL                      | 1282             |

This is a lot of information, we'll have to do something about this or our inbox(es) will get full fast and if we receive too
many, there is a chance that something falls through the cracks if someone starts to consider this "noise" and "can be ignored."




