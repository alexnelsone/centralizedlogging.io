Installing Landing Zone Accelerator on AWS installs a considerable amount of things.  We don't really need to jump into 
the details of all of that.  For example, at this time, we don't need to get too much into the weeds of a Lambda helper
function that assists with deployment.  Here we will stick to some basic things around security.  Where are we at right
now in regard to our installation and our compliance and security posture.
    
Let's start with security.

1. Login to the `Audit` account
2. Navigate to the `Security Hub` console

In my current configuration, on the Summary page, I am seeing the following under `Security standards`:

![19-configure-lza.png](images%2F19-configure-lza.png)    
    
In our `security-config.yaml` file under the Security Hub settings, we don't have anything 
configured.

```yaml
  securityHub:
    enable: true
    regionAggregation: true
    excludeRegions: []
    standards: []
```
It looks like `CIS AWS Foundations Benchmark v1.2.0` and `AWS Foundational Security Best Practices v1.0.0` has been 
enabled by default.  We are seeing an average Security score of 89%.  I am more concerned with `NIST Special Publication
800-53 Revision 5` compliance, so we will enable that shortly.
    
In our `Findings by Region` summary we are seeing the following:

![20-configure-lza.png](images%2F20-configure-lza.png)    
    
Let's click on the 19 Critical findings to see what we can fix.
    
Looks like all of them are related to MFA on the root user.  It also looks like some duplicates. For example we see:

```shell
1.14 Ensure hardware MFA is enabled for the root user
```
 and 

```shell
IAM.6 Hardware MFA should be enabled for the root user
```

The rules the beginning with the number, for example 1.14, are from `CIS AWS Foundations Benchmark` and the rules starting with
`IAM.6` are from `AWS Foundataional Security Best Practices`.  Yet another reason we should probably think about focusing 
on just one standard.  In order to avoid being billed multiple times for the same rule.  We will get to that later.  For now,
let's take care of some these Critical findings.