# Troubleshooting CodePipeline<a name="troubleshooting"></a>

The following information might help you troubleshoot common issues in AWS CodePipeline\.

**Topics**
+ [Pipeline error: A pipeline configured with AWS Elastic Beanstalk returns an error message: "Deployment failed\. The provided role does not have sufficient permissions: Service:AmazonElasticLoadBalancing"](#troubleshooting-aeb1)
+ [Deployment error: A pipeline configured with an AWS Elastic Beanstalk deploy action hangs instead of failing if the "DescribeEvents" permission is missing](#troubleshooting-aeb2)
+ [Pipeline error: A source action returns the insufficient permissions message: "Could not access the CodeCommit repository `repository-name`\. Make sure that the pipeline IAM role has sufficient permissions to access the repository\."](#troubleshooting-service-role-permissions)
+ [Pipeline error: A Jenkins build or test action runs for a long time and then fails due to lack of credentials or permissions](#troubleshooting-jen1)
+ [Pipeline error: A pipeline created in one AWS Region using a bucket created in another AWS Region returns an "InternalError" with the code "JobFailed"](#troubleshooting-reg-1)
+ [Deployment error: A ZIP file that contains a WAR file is deployed successfully to AWS Elastic Beanstalk, but the application URL reports a 404 not found error](#troubleshooting-aeb2)
+ [Pipeline artifact folder names appear to be truncated](#troubleshooting-truncated-artifacts)
+ [Add GitClone permissions for connections](#codebuild-role-connections)
+ [Pipeline error: A deployment with the CodeDeployToECS action returns an error message: "Exception while trying to read the task definition artifact file from: <source artifact name>"](#troubleshooting-ecstocodedeploy-size)
+ [Need help with a different issue?](#troubleshooting-other)

## Pipeline error: A pipeline configured with AWS Elastic Beanstalk returns an error message: "Deployment failed\. The provided role does not have sufficient permissions: Service:AmazonElasticLoadBalancing"<a name="troubleshooting-aeb1"></a>

**Problem:** The service role for CodePipeline does not have sufficient permissions for AWS Elastic Beanstalk, including, but not limited to, some operations in Elastic Load Balancing\. The service role for CodePipeline was updated on August 6, 2015 to address this issue\. Customers who created their service role before this date must modify the policy statement for their service role to add the required permissions\. 

**Possible fixes:** The easiest solution is to edit the policy statement for your service role as detailed in [Add permissions to the CodePipeline service role](security-iam.md#how-to-update-role-new-services)\.

After you apply the edited policy, follow the steps in [Start a pipeline manually](pipelines-rerun-manually.md) to manually rerun any pipelines that use Elastic Beanstalk\.

Depending on your security needs, you can modify the permissions in other ways, too\. 

## Deployment error: A pipeline configured with an AWS Elastic Beanstalk deploy action hangs instead of failing if the "DescribeEvents" permission is missing<a name="troubleshooting-aeb2"></a>

**Problem:** The service role for CodePipeline must include the `"elasticbeanstalk:DescribeEvents"` action for any pipelines that use AWS Elastic Beanstalk\. Without this permission, AWS Elastic Beanstalk deploy actions hang without failing or indicating an error\. If this action is missing from your service role, then CodePipeline does not have permissions to run the pipeline deployment stage in AWS Elastic Beanstalk on your behalf\.

**Possible fixes:** Review your CodePipeline service role\. If the `"elasticbeanstalk:DescribeEvents"` action is missing, use the steps in [Add permissions to the CodePipeline service role](security-iam.md#how-to-update-role-new-services) to add it using the **Edit Policy** feature in the IAM console\.

After you apply the edited policy, follow the steps in [Start a pipeline manually](pipelines-rerun-manually.md) to manually rerun any pipelines that use Elastic Beanstalk\.

## Pipeline error: A source action returns the insufficient permissions message: "Could not access the CodeCommit repository `repository-name`\. Make sure that the pipeline IAM role has sufficient permissions to access the repository\."<a name="troubleshooting-service-role-permissions"></a>

**Problem:** The service role for CodePipeline does not have sufficient permissions for CodeCommit and likely was created before support for using CodeCommit repositories was added on April 18, 2016\. Customers who created their service role before this date must modify the policy statement for their service role to add the required permissions\. 

**Possible fixes:** Add the required permissions for CodeCommit to your CodePipeline service role's policy\. For more information, see [Add permissions to the CodePipeline service role](security-iam.md#how-to-update-role-new-services)\.

## Pipeline error: A Jenkins build or test action runs for a long time and then fails due to lack of credentials or permissions<a name="troubleshooting-jen1"></a>

**Problem:** If the Jenkins server is installed on an Amazon EC2 instance, the instance might not have been created with an instance role that has the permissions required for CodePipeline\. If you are using an IAM user on a Jenkins server, an on\-premises instance, or an Amazon EC2 instance created without the required IAM role, the IAM user either does not have the required permissions, or the Jenkins server cannot access those credentials through the profile configured on the server\. 

**Possible fixes:** Make sure that Amazon EC2 instance role or IAM user is configured with the `AWSCodePipelineCustomActionAccess` managed policy or with the equivalent permissions\. For more information, see [AWS managed \(predefined\) policies for CodePipeline](managed-policies.md)\.

If you are using an IAM user, make sure the AWS profile configured on the instance uses the IAM user configured with the correct permissions\. You might have to provide the IAM user credentials you configured for integration between Jenkins and CodePipeline directly into the Jenkins UI\. This is not a recommended best practice\. If you must do so, be sure the Jenkins server is secured and uses HTTPS instead of HTTP\.

## Pipeline error: A pipeline created in one AWS Region using a bucket created in another AWS Region returns an "InternalError" with the code "JobFailed"<a name="troubleshooting-reg-1"></a>

**Problem:** The download of an artifact stored in an Amazon S3 bucket will fail if the pipeline and bucket are created in different AWS Regions\.

**Possible fixes:** Make sure the Amazon S3 bucket where your artifact is stored is in the same AWS Region as the pipeline you have created\.

## Deployment error: A ZIP file that contains a WAR file is deployed successfully to AWS Elastic Beanstalk, but the application URL reports a 404 not found error<a name="troubleshooting-aeb2"></a>

**Problem:** A WAR file is deployed successfully to an AWS Elastic Beanstalk environment, but the application URL returns a 404 Not Found error\.

**Possible fixes:** AWS Elastic Beanstalk can unpack a ZIP file, but not a WAR file contained in a ZIP file\. Instead of specifying a WAR file in your `buildspec.yml` file, specify a folder that contains the content to be deployed\. For example:

```
version: 0.2

phases:
  post_build:
    commands:
      - mvn package
      - mv target/my-web-app ./
artifacts:
  files:
    - my-web-app/**/*
 discard-paths: yes
```

For an example, see [AWS Elastic Beanstalk Sample for CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/sample-elastic-beanstalk.html)\.

## Pipeline artifact folder names appear to be truncated<a name="troubleshooting-truncated-artifacts"></a>

**Problem:** When you view pipeline artifact names in CodePipeline, the names appear to be truncated\. This can make the names appear to be similar or seem to no longer contain the entire pipeline name\.

**Explanation:** CodePipeline truncates artifact names to ensure that the full Amazon S3 path does not exceed policy size limits when CodePipeline generates temporary credentials for job workers\.

Even though the artifact name appears to be truncated, CodePipeline maps to the artifact bucket in a way that is not affected by artifacts with truncated names\. The pipeline can function normally\. This is not an issue with the folder or artifacts\. There is a 100\-character limit to pipeline names\. Although the artifact folder name might appear to be shortened, it is still unique for your pipeline\.

## Add GitClone permissions for connections<a name="codebuild-role-connections"></a>

When you use an AWS CodeStar connection in a source action and a CodeBuild action, there are two ways the input artifact can be passed to the build:
+ The default: The source action produces a zip file that contains the code that CodeBuild downloads\.
+ Git clone: The source code can be directly downloaded to the build environment\. 

  The Git clone mode allows you to interact with the source code as a working Git repository\. To use this mode, you must grant your CodeBuild environment permissions to use the connection\.

To add permissions to your CodeBuild service role policy, you create a customer\-managed policy that you attach to your CodeBuild service role\. The following steps create a policy where the `UseConnection` permission is specified in the `action` field, and the connection ARN is specified in the `Resource` field\. 

**To use the console to add the UseConnection permissions**

1. To find the connection ARN for your pipeline, open your pipeline and click the \(i\) icon on your source action\. You add the connection ARN to your CodeBuild service role policy\.

   For this example, the connection ARN is:

   ```
   arn:aws:codestar-connections:eu-central-1:123456789123:connection/sample-1908-4932-9ecc-2ddacee15095
   ```  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/images/gitclone-configuration.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)

1. To find your CodeBuild service role, open the build project used in your pipeline and navigate to the **Build details **tab\. 

1. Choose the **Service role** link\. This opens the IAM console where you can add a new policy that grants access to your connection\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/images/gitclone-configuration-role.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)

1. In the IAM console, choose **Attach policies**, and then choose **Create policy**\.

   Use the following sample policy template\. Add your connection ARN in the `Resource` field, as shown in this example:

   ```
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": "codestar-connections:UseConnection",
               "Resource": "insert connection ARN here"
           }
       ]
   }
   ```

   On the **JSON** tab, paste your policy\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/images/gitclone-role-policy.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)

1. Choose **Review policy**\. Enter a name for the policy \(for example, **connection\-permissions**\), and then choose **Create policy**\.

1. Return to the page where you were attaching permissions, refresh the policy list, and select the policy you just created\. Choose **Attach policies**\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/images/gitclone-role-policy-attach.png)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)![\[Image NOT FOUND\]](http://docs.aws.amazon.com/codepipeline/latest/userguide/)

## Pipeline error: A deployment with the CodeDeployToECS action returns an error message: "Exception while trying to read the task definition artifact file from: <source artifact name>"<a name="troubleshooting-ecstocodedeploy-size"></a>

**Problem:** 

The maximum artifact ZIP size in the CodePipeline deploy action to ECS through CodeDeploy \(the `CodeDeployToECS` action\) is 3 MB\. Also, ZIP files that use input streams together with the store (uncompressed) archiving method are not supported. The following error message is returned when artifact sizes exceed 3 MB, or if the ZIP file uses stored (uncompressed) input streams: 

Exception while trying to read the task definition artifact file from: <source artifact name>

**Possible fixes:** Create an artifact with a compressed size less than 3 MB\. Also, verify that the ZIP file does not use the combination of input streams and the store (uncompressed) archiving method\.

## Need help with a different issue?<a name="troubleshooting-other"></a>

Try these other resources:
+ Contact [AWS Support](https://aws.amazon.com/contact-us/)\.
+ Ask a question in the [CodePipeline forum](https://forums.aws.amazon.com/forum.jspa?forumID=197)\.
+ [Request a quota increase](https://console.aws.amazon.com/support/home#/case/create%3FissueType=service-limit-increase)\. For more information, see [Quotas in AWS CodePipeline](limits.md)\.
**Note**  
It can take up to two weeks to process requests for a quota increase\.