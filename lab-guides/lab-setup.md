[Back to main guide](../README.md)|[Next](create-lambda-functions.md)

___

# 1. Lab Setup

In this activity we will deploy the CloudFormation template to create required IAM roles and configure S3 bucket for hosting static web application.

___

## Deploy the CloudFormation Template

1. Click on one of the buttons below to launch CloudFormation stack in one of the AWS regions.

Region | Launch
-------|-----
US East (N. Virginia) | [![Launch Solution in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=NotificationsWorkflow&templateURL=https://s3.amazonaws.com/reinvent-notification-workflow-workshop/notification-workflow-prereq.yaml)
US West (Oregon) | [![Launch Solutionin us-west-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=NotificationsWorkflow&templateURL=https://s3.amazonaws.com/reinvent-notification-workflow-workshop/notification-workflow-prereq.yaml)


2. Click **Next** on the Select Template page.

3. Enter a **Stack Name** or leave the default and click **Next**.

4. On the Options page, leave all the defaults and click **Next**.

5. On the Review page, check the box to acknowledge that CloudFormation will create IAM resources and click **Create**.
    ![Acknowledge IAM Screenshot](images/cfn-ack-iam.png)

6. Click on Stacks in  the top navigation to see the CloudFormation Stack screen. You should see the `NotificationsWorkflow` CloudFormation template in progress.

7. After CloudFormation template is complete, select the stack Name - `NotificationsWorkflow`, Click on the **Resources** tab from the bottom pane. You will see various AWS resources including S3 buckets and IAM roles created.

8. Make a Note of the stack Output parameters. You can find the stack output at the **Outputs** tab from the bottom pane.

	![Stack Output](images/cfn-output.png)
___

[Back to main guide](../README.md)|[Next](create-lambda-functions.md)