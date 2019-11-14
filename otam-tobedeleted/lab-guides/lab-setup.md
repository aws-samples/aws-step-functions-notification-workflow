[Back to main guide](../README.md)|[Next](sct.md)

___

# Lab Setup

In this activity we will deploy the CloudFormation template to create required lab environment.

The environment for this lab consists of:
- An EC2 instance with following components 
   - AWS Schema Conversion Tool (SCT)
   - Source Oracle database
   - SQL Developer
   - Sample Web Application 
- An Amazon RDS Aurora PostgreSQL instance used as the target database

___

## Deploy the CloudFormation Template

1. Click on one of the buttons below to launch CloudFormation stack in one of the AWS regions.

Region | Launch
-------|-----
US East (N. Virginia) | [![Launch Solution in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=DMSWorkshop&templateURL=https://reinvent-2019-oracle-aurora.s3.amazonaws.com/GPSTEC315_lab_template.json)
US West (Oregon) | [![Launch Solutionin us-west-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=DMSWorkshop&templateURL=https://reinvent-2019-oracle-aurora.s3.amazonaws.com/GPSTEC315_lab_template.json)

2. Click **Next** on the Select Template page.

3. Enter a **Stack Name** or leave the default and click **Next**.

4. On the Options page, leave all the defaults and click **Next**.

5. On the Review page, click **Create**.
    
6. Click on Stacks in  the top navigation to see the CloudFormation Stack screen. You should see the `DMSWorkshop` CloudFormation template in progress.

![Stack Progress](images/stack-progress.png)

7. After CloudFormation template is complete, select the stack Name - `DMSWorkshop`, Click on the **Resources** tab. You will see various AWS resources.

8. Make a Note of the stack Output parameters. You can find the stack output at the **Outputs** tab.

![Stack Output](images/cfn-output.png)
___

[Back to main guide](../README.md)|[Next](sct.md)