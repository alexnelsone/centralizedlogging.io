
Before we check in our configuration changes to our repository and running the pipeline, we can do a few checks locally
to make sure things are ok.  If we do have an error, we will want to check it locally rather than running the pipeline and
finding out during pipeline execution that we have a problem.

1. Clone the Landing Zone Accelerator on AWS GitHub repo to a directory outside your config (https://github.com/awslabs/landing-zone-accelerator-on-aws)
2.Go into the cloned directory
3. `cd source`
4. `yarn clean up`
4. `yarn install`
5. `yarn build`
5. `yarn lerna link`

<details>
    <summary>:exclamation: Some additional commands (MAYBE)</summary>
    <p>While testing, I also had to run the following for this to work. In the past I did not. Not sure if this is new. <br />
    <code>npm install -g ts-node</code> <br />>
    <code>npm i --save-dev @types/node</code>
    </p>
</details>

6. CODEBUILD_SRC_DIR_Config=PATH_TO/aws-accelerator-config
7. yarn validate-config $CODEBUILD_SRC_DIR_Config


IF all is good, you should see something like this:
```shell
2023-08-17 16:47:26.885 | info | config-validator | Config source directory -  PATH-TO/aws-accelerator-config
2023-08-17 16:47:26.893 | info | accounts-config-validator | accounts-config.yaml file validation started
2023-08-17 16:47:26.893 | info | global-config-validator | global-config.yaml file validation started
2023-08-17 16:47:26.894 | info | global-config-validator | email count: 1
2023-08-17 16:47:26.894 | info | iam-config-validator | iam-config.yaml file validation started
2023-08-17 16:47:26.895 | info | network-config-validator | network-config.yaml file validation started
2023-08-17 16:47:26.896 | info | organization-config-validator | organization-config.yaml file validation started
2023-08-17 16:47:26.897 | info | security-config-validator | security-config.yaml file validation started
2023-08-17 16:47:26.897 | info | config-validator | Config file validation successful.

```

Once you see this, you can go ahead and commit and push your changes to the CodeCommit repository.  Once the changes have 
been pushed, you can open the CodePipeline console, select the `AWSAccelerator-Pipeline` and click `Release change`.
