No that both the `AWSAccelerator-Installer` and `AWSAccelerator-Pipeline` have run successfully, we can keep going with
configuring our landing zone.  But what happened?  What did we get?  Well, we know that LZA is built on CDK and we know
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