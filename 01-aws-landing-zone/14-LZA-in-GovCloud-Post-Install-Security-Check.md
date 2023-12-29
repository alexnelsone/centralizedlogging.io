If you quickly jumped here from the previous step, you will need to wait a few hours.  It takes time for the security
services to start reporting to the `Audit` account and if you look at things now, you will see a misrepresentation of
the out-of-the-box security posture.

If it's "been a while" since you finished the last step, continue. 

Let's take a look at the `Security Hub` console in the GovCloud `Audit` account and see what we have.

Being logged into the GovCloud `Management` account:
1. Select the account drop down in the upper right hand corner of the console
2. Select `Switch role`
3. Enter the `Audit` account `Account Id` in the `Account*` box
4. Enter `AWSControlTowerExecution` into the `Role*` box
5. Enter a name in the `Display Name` box.  I like to use something like `govcloud-Audit` so that I know which `Audit` 
acocunt I am in.
6. Click `Switch Role`

You should now see the `Display name` you entered at the top right hand corner of the console

1. Open the `Security Hub` console
2. 
