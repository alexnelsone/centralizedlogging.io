Because we didn't set up Control Tower on the GovCloud side, SSO was not configured for us.  Let's go ahead and get that 
configured.

1. Open the `IAM Identity Center (successor to AWS Single Sign-On)` console in the management account
2. Click on the `Enable` button

By default, the `Identity source` is going to be `Identity Center Directory`.  This means that the user directory is being
managed by AWS SSO.  This is fine for now.  Later, we'll see how we can change the `Identity source` to either be 
Azure or Google.  Right now, you are probably logged in using your root user email (which you shouldn't do) or a user
you created in the management account with administrator privileges (much better than using the root account and the suggested
approach from AWS).

Let's create an SSO user with admin permissions so that you can use that user instead of an IAM user or the root user account.

## Create a user
1. From the menu select `Groups`
2. Click the `Create group` button
3. For `Group name` enter `AWS-Administrators`
4. For `Description - optional` enter `Users in this groups have full administrator privileges across all accounts`
5. Click `Create group`
6. Click `Users` from the menu
7. Click `Add user`
8. For `Username` enter an identifier for the user.  I like to use email addresses.  For example, `username@domain.com`.
9. Leave `Send an email to this user with password setup instructions.` selected
10. Enter an email into the `Email address` box
11. Confirm the email in the `Confirm email address` box
12. Enter a `First name` and `Last time` if you like.  This will autopopulate the Display name
13. Click `Next`
14. On the `Add user to groups` page, select the `AWS-Administrators` group you created
15. Click `Next`
16. Click `Add user`

The user will receive an email with the subject `Invitation to join IAM Identity Center (successor to AWS Single Sign-On)`
with a link they can use to set the password.

## Create a permission set
1. Select `Permission sets` from the menu
2. Click `Create permission set`
3. Leave `Predefined permission set` selected
4. Select `AdministratorAccess` from the `Policy for predefined permission set`
5. Click `Next`
6. Leave the `Permission set name` as `AdministratorAccess`
7. Set your desired `Session Duration`
8. Click `Next`
9. Click `Create`

## Assign permissions to accounts
1. Select `AWS Accounts` from the menu
2. In the `Organizational structure` section, select all the accounts in the organization
2. Click on `Assign users or groups`
3. Select the `AWS-Administrators` group you created earlier
4. Click `Next`
5. Select the `AdministratorAccess` permission set you created earlier
6. Click `Next`
7. Click `Submit`

A progress indicator will appear showing that the permission set is being pushed out to the selected accounts and when
finished, you will be redirected to the `AWS accounts` page.

If you go back to the SSO dashboard, under `Settings summary` you can access the environment using the `AWS access portal URL`
and the account you just created.  When logged in as the user, they will be presented with the list of accounts they were
provided access to as well as a drop-down showing the `AdministratorAccess` role that they can use to enter the console
or access the account programmatically.