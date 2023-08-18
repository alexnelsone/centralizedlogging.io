When setting out to build an environment in the cloud, it's really important to think about how you are going to lay down 
the foundation that everything is going to be built on.  It's a bit difficult once you are well in the weeds, to start
making changes to your underlying infrastructure once you have workloads up and running.  So we want to get this somewhat
"right".
     
We are going to be building things out on AWS so we want to make sure we follow best practices.  At the moment, "best practice"
is building out what AWS refers to as a landing zone.   A landing zone is a "well-architected, multi-account AWS environment
that is a starting point from which you can deploy workloads and applications."  Looking at AWS Prescriptive Guidance
for [Setting up a secure and multi-account AWS environment](https://docs.aws.amazon.com/prescriptive-guidance/latest/migration-aws-environment/welcome.html),
it describes, in the introduction, that a successful cloud adoption starts with an environment that includes:
- An AWS environment with a multi-account architecture
- An initial security baseline
- Identity and Access Management
- Governance
- Data security
- Network design
- Logging

It describes all of these things together as the definition of a _landing zone_.  As we build out the infrastructure for 
this project, we are going to follow this multi-account approach.  Luckily, AWS offers a managed service, [AWS Control Tower](https://aws.amazon.com/controltower/),
that can help us automatically provision our landing zone environment.  AWS recommends that all new landing zones start with
AWS Control Tower and that is what we are going to do.  Since we already know this, we will go ahead and start the Control 
Tower process to establish the landing zone.

Additionally, we are going to be operating some workloads in AWS's GovCloud (US) regions.  So we will need to make sure
that we set up an account there as well.  We are doing this because we want to also set up some workloads in a more
regulated environment.

Lastly, regarding setup, we want to make things a little bit easier. So, to manage our environment we are going to utilize a tool
from AWS called `Landing Zone accelerator on AWS.`  This is going to let us configure things with config files once we have 
our base landing zone configured.

One thing to think about when creating accounts in AWS is email addresses.  When you create accounts in AWS, they are associated
with an email address that you use when you initially set up the account.  This email is known as the "root user."  You will
want to only use this user to set up the account, get in, create an admin user, and then lock up the root user info somewhere.
All the AWS documentation tells you to follow this process.  When creating the root user though, consider using '+ notation.'
Because you don't know how many accounts you are going to be creating and if you really don't want to manage a bunch of mailboxes
for each account's root user, plus notation is the way to go.  What that means is that you can create one mailbox like
`root-user@my-domain.com` and then use it as the basis for all the other accounts by appending +SOMETHING to the alias.  For example,
knowing that the first account you create using Control Tower is considered the 'management' account, you can create the mailbox
for `root-user@mydomain.com` and then when you sign up for the AWS account, use `root-user+mgmt-account@my-domain.com` as the email address
for the root user of that account.  

SPOILER-ALERT:  Control tower is going to create two other accounts.  One called Audit
and another called Logging.  In these cases we can use `root-user+audit@my-domain.com` and `root-user+logging@my-domain.com`
for those. Then all emails will go to the same mailbox.  If you need to route those messages to other responsible persons or groups,
just create rules in the mailbox.  This way, you have all communications regarding the environment in a centralized location.
Of course, the other option is to create a mailbox for each account and there might be some overarching circumstatnces, such 
as corporate policy, that dictate a mailbox per account.


I'm not going to go into the details of creating an AWS account beyond saying, "Go to aws.amazon.com" and click on the "Create an AWS Account"
button in the upper right hand corner.   Once you have an initial account setup in the commercial partition, you will 
need to set up your GovCloud account.  If you don't plan on using GovCloud (US), you don't need to do this.

### Setup GovCloud Account (Optional)

Initially, I'm not going to setup the GovCloud (US) side of this configuration.  At the moment, I plan on running workloads in
both the commercial and GovCloud (US) partitions, but I will work on the GovCloud side later.

1. login to your commercial account with the root user, the email address you used to create the account.
2. In the upper right hand corner, select the drop-down next to account name and select 'Account'    
    ![01-configure-lza.png](images%2F01-configure-lza.png)    
    
3. On the account page, scroll until you see 'Other Settings' and click `AWS GovCloud` on the line that says "To sign up for
AWS GovCloud (US), see AWS GovCloud."  If you do not see this, you did not login as the root user.
4. That will take you to the AWS GovCloud (US) Amendment to the AWS Customer Agreement for you to read.
5. Click the boxes stating you have read the agreement and that certify that you are a U.S. Person and qualify for access
6. Fill in the rest of the information on the form and click the 'Request Access' button

This will trigger all events on the AWS side to get you set up that will link this account to a GovCloud (US) account. 
Once done, you will be provided with API keys that you can use to get started with that account.  While we wait for that, 
we are going to go ahead and setup AWS Control Tower in our commercial region.





