For each of our accounts that are alertingon MFA issues with the root users, we will perform the following tasks.
You might want to consider doing this from a private browser window and to have your SSO login page, that lists all your
accounts, open. Having the SSO login page open, you can have a list of your accounts and the root account email displayed
to make it easier.

1. Open the public page to log in to the management console [https://aws.amazon.com/console/](https://aws.amazon.com/console/)
2. Click `Sign In`
3. On the Sign-in page, make sure that `Root user` option is selected.

![21-configure-lza.png](images%2F21-configure-lza.png)    
    
4. Enter the root email address for the first account.  In my case, I am going down my list from the SSO login page so the Audit account
is the first in my list.
5. Click `Next`
6. Because we set up these accounts using Control Tower and `Landing Zone Accelerator for AWS` we did not enter any passwords.  So 
we need to reset the passwords as we go along.  Click on the `Forgot password?` link

![22-configure-lza.png](images%2F22-configure-lza.png)   
    
7. On the `Password recovery` popup, enter the special characters and click on `Send email`. You will see a message indicating `Password email sent`. Click `Done`
8. You should receive an email with the subject `Amazon Web Services Password Assistance` with a link to use to reset your password
9. Copy the link and paste it into your private browser window
10. Enter your new password and confirm it
11. Click `Reset password`
12. When you see the `Password reset successful` message, click the `Sign in` button
13. Enter your root user email and password and you should be in!
14. Once in, you should see the name of your account in the upper right hand corner.  I started with my `Audit` account, so that is what I see.
15. Navigate to the `IAM` console
16. With the `IAM` console open, you should see a box labelled `Security recommendations`    
![24-configure-lza.png](images%2F24-configure-lza.png)    
17. Click on the `Add MFA` button next to `Add MFA for root user`
18. Enter a name for the device
19. Select the MFA Device type.  NOTE: One of the Security Hub findings for MFA on the root user is `Hardware MFA`.  If you select `Authenticator App`, it will not remediate that finding
20. Complete the MFA device setup

When you have completed, you should now see that you have an MFA device configured on the `My security credentials` page. Below is a screenshot of my configuration.

![25-configure-lza.png](images%2F25-configure-lza.png)    
    
Note that I did not use a hardware device, denoted by the `Virtual` device type.  If you want to remediate the `Hardware MFA` findings in Security Hub, you should
use a hardware device for MFA on your root user.  You can register up to 8 MFA devices of any combination of the currently supported MFA types with your AWS account root 
and IAM user. With multiple MFA devices, you only need one MFA device to sign in to the AWS console or create a session through the AWS CLI with that user.  So, if all you
have is the MFA app, start there.  You can always configure a hardware MFA later.

You can now log out of this account and repeat the steps on this page for all of your accounts.  Remember, as you create new accounts
using Landing Zone Accelerator on AWS, you will need to perform this for each account.