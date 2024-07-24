
# Pre-Requisites #


AWS account 
AWS IAM user 'devops-admin' user created with administrator access
workstation server or local server - Git installed - Optional Or you can use laptop

---------------------------------------------------------------------------------------

End-End CI/CD Devployment of Java Project on Elastic-Beanstalk
==============================================================
![image](https://github.com/pratiksuryvnshi/DevOps-AWS-Projects/assets/90375660/4f8620a8-4197-43d4-b31d-d9f74f93a764)


Lab - STEPS TO IMPLEMENTATION OF PROJECT
========================================

Note: All AWS operations should be done with "devops-admin" IAM user. Hence login to AWS console with "devops-admin" user. 

Code Repo - https://devops-training-usecases-repo.s3.ap-south-1.amazonaws.com/uc1-awsdevops-elasticbeanstalk-s3/aws-devops-ebs-java.zip


AWS Services Used
-----------------

CodeCommit
CodeBuild
Code Artifacts
CodeDeploy
CodePipeline
S3
SNS
Elastic Beanstalk
IAM


++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Creating Repository in AWS CodeCommit ***
=========================================

1. Login to AWS console and go to CodeCommit service
2. Click on Create Repository
3. Enter Repository Name - "aws-devops-ebs-java-repo"
4. Click on Create

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Creating IAM User to Access the CodeCommit Repository ***
============================================================

Follow the below URL or steps mentioned below

https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html?icmpid=docs_acc_console_connect_np

OR

1. Open IAM service -> Go to users tab -> Click on Add users
2. Enter the name as - aws-devops-ebs-java-codecommit-user
3. Click Next -> Attach policy directly
4. Search Policy "AWSCodeCommitPowerUser", select and click next
5. Click on create user.
6. Once created, select that user and click on Security Credential tab
8. In section - HTTPS Git credentials for AWS CodeCommit, Click on generate credential button
9. Note down the username and password displayed in notepad and close the window. This credential will be used for check-in in to code commit.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Clonning CodeCommit Repo into locally
=========================================

Note - You can use your laptop as well instead of EC2 workstation

1. Download https://devops-training-usecases-repo.s3.ap-south-1.amazonaws.com/uc1-awsdevops-elasticbeanstalk-s3/aws-devops-ebs-java.zip extract
into temp folder
2. Open the VS Code application and clone the above created repo 
e.g. git clone <URL>

3. It will clone empty repository
4. Copy content above temp directory into cloned repo directory
5. git add .
6. git commit -m "Initial Commit" .
7. git push.
8. Verify all files are commited into code commit master branch.



++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Configuring CodeArtifact ***  
================================

1. Login to AWS console and go to CodeArtifact service.
2. click on Create Repository
3. Enter name - aws-devops-ebs-java-artifacts
4. Select "maven-central-store" in upstream -> Next
5. In Domain -> Select "This AWS account"
6. Select domain listed -> Next -> Next
7. Click on create repository


++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Configuring CodeBuild ***
============================

Please note - Create one bucket in Mumbai region with name - aws-devops-ebs-java-s3-artifacts.
You may observe error during creation about - bucket already exists - Please create unique bucket name.



1. Click on Build Section on left hand navigation pane and select build projects. Click on Create Build Project
2. Enter project name - aws-devops-ebs-java-codebuild. Enter same name in description
3. In Source Section, select AWS CodeCommit as Source Provider
4. Select repostiory - aws-devops-ebs-java-repo
5. Reference Type - Branch
6. Select branach - master
7. In Environment Section - Managed Image
8. Select Operating system - Ubuntu
9. Runtime - Standard
10. Image - Select image of vesion 4.0
11. Service Role section - Select New Service Role. Keep Service Role Name as default.
12. In BuildSpec section - No need to change anything as we have buildspec.yml file on required default path
13. In Artifacts section - Type - Amazon S3
14. In Logs section - Enable Cloudwatch logs. 
15. Click on Create build project
16. Click on start build button
17. Verify the logs and build job should be successful
18. Verify that artifacts are also uploaded to S3 bucket - aws-devops-ebs-java-s3-artifacts   	- Verify war file is uploaded to S3 bucket

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Configuring Code Pipeline job with Dev/Prod Elastic beanstalk environment ***
============================================================================

Please note - Before configuring code deploy, we need to create target elastic beanstalk dev and prod environments

1. Go to Elastic Beanstalk Service -> Click on Create application -> Create web server environment
2. Enter application name as - aws-devops-ebs-java-dev
3. Select platform - Tomcat
4. Keep default values for branch, version
5. Select Sample application code and click on create application
6. Wait for few minutes, your beanstalk tomcat environment should be up and running.
7. Once environment is up, click on application -> click on URL.
8. Verify Application should be up and running

Similarly, create one more environment with name as - aws-devops-ebs-java-prod

1. Go to Code pipeline service
2. Click on create pipeline
3. Enter pipeline name as - aws-devops-ebs-java-pipeline
4. Click on new service role and keep role name as default-> click next
5. Select source as AWS CodeCommit
6. Select repository name as - aws-devops-ebs-java-repo
7. Branch name - master. Keep all other options default -> Next
8. In Build Section - Select build provider - AWS CodeBuild
9. Region - Mumbai
10. Project Name - aws-devops-ebs-java-codebuild
11. Build Type - Single build -> Next
12. In Deploy section, select AWS elastic beanstalk
13. Region Mumbai
14. Select application name - aws-devops-ebs-java-dev
15. Select envrionment name - aws-devops-ebs-java-dev  -> Next
16. Click on create pipeline
17. All the stages source, build and deploy should be successful.
18. Go to elastic beanstalk environment - aws-devops-ebs-java-dev and access URL    - Verify application is deployed and accessible

Please note - Access Elastic beanstalk URL as below
https://<URL>/demo

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Configuring Production EBS environment into pipeline ***
============================================================

1. Go to pipeline - > Select pipeline - aws-devops-ebs-java-pipeline
2. Click on edit
3. After deploy section -> click on add stage 
4. Enter name as - aws-devops-ebs-java-prod-deploy -> click add stage
5. Click on add action group button
6. Enter Action Name - aws-devops-ebs-java-prod-deploy
7. Action Provider - AWS elastic beanstalk
8. Region - Mumbai
9. Input artifacts - build artifacts
10. Application Name - aws-devops-ebs-java-prod
11. Enronment Name - aws-devops-ebs-java-prod

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Add Manual Approval before Prod Deploy ***
==============================================

1. Go to pipeline -> Select pipeline - aws-devops-ebs-java-pipeline
2. Click on edit
3. After dev-deploy and before aws-devops-ebs-java-prod-deploy section - > Click on Add stage
4. Enter stage Name - ManualApproval
5. Click on Add action group
6. Enter Action group name - ManualApproval
7. Action Provider - Manual Approval
8. Keep all other options default -> Done
9. Click on save and click on save again on dialog box

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Configure Notification Rule on pipeline ***
===============================================

First need to create SNS Topic
-----------------------------

1. Go to SNS service -> Click on Topics -> Create Topic
2. Type - standard
3. Name - aws-devops-ebs-java-topic
4. Display Name - aws-devops-ebs-java-topic
5. Click on Create Topic
6. Click on create subscription
7. Protocol - email
8. endpoint - Enter your valid email address
9. Click on create subscription
10. Go to your email account and verify you have recieved email. Click on Confirm subscription


Add policy to SNS Topic 
----------------------

1. Go to SNS topic -> select topic aws-devops-ebs-java-topic
2. Click on edit
3. Click on Access Policy
4. Either go to below document and copy Policy or follow below mentioned policy

https://docs.aws.amazon.com/dtconsole/latest/userguide/set-up-sns.html

<--Policy starts here, please note, replace SNS TOPIC ARN and AWS Account number at two places in below policy

{
  "Version": "2008-10-17",
  "Id": "__default_policy_ID",
  "Statement": [
    {
      "Sid": "__default_statement_ID",
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": [
        "SNS:GetTopicAttributes",
        "SNS:SetTopicAttributes",
        "SNS:AddPermission",
        "SNS:RemovePermission",
        "SNS:DeleteTopic",
        "SNS:Subscribe",
        "SNS:ListSubscriptionsByTopic",
        "SNS:Publish",
        "SNS:Receive"
      ],
      "Resource": "<SNS Topic ARN>",
      "Condition": {
        "StringEquals": {
          "AWS:SourceOwner": "<AWS Account Number>"
        }
      }
    },
    {
      "Sid": "AWSCodeStarNotifications_publish",
      "Effect": "Allow",
      "Principal": {
        "Service": "codestar-notifications.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "<SNS Topic ARN>",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "<AWS Account Number>"
        }
      }
    }
  ]
}

--Policy end here>

5. Copy above policy and paste in Access Policy section by deleting existing policy
6. Click on Save changes

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** Add Notification Rule To Pipeline ***
==========================================

1. Go to pipeline -> select pipeline -> Click on notify -> Click Create Notification Rule
2. Enter Name - aws-devops-ebs-java-pipeline-notification
3. Details type - Full
4. Select Succeded and Failed event from each coloumn
5. In Target section - Select SNS Topic as Target Type 
6. In choose target, select - aws-devops-ebs-java-topic
7. Click on Submit

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

*** End To End AWS DevOps Pipeline Verification ***
====================================================

1. Go to workstation and login with ec2-user. Switch to root - sudo su
2. Run command - cd /home/ec2-user/aws-devops-ebs-java/src/main/webapp
3. Modify index.jsp file message as "Welcome to DevOps v1..!!". Save the file
4. Run - git status
5. Run - git add index.jsp
6. Run - git commit -m "App version 1" index.jsp
7. Run - git status
8. Run - git push
9. Enter your codecommit username and password 
10. Go to pipeline and verify status of all jobs


Validation Points
----------------

1. All job status should be successful
2. Verify Dev Elastic bean URL working with new code change
Access URL as - https://URL/demo
3. Pipeline will wait for manual approval before prod release changes
4. Go to Code Pipeline and In Approval Stage - Click on review
5. Enter any comments and click on Approve
6. Verify Prod Elastic bean URL working with new code changes.
7. Verify you recieved the SNS notifications on email for pipeline successful events
8. Verify that artifacts (demo.war) are uploaded on s3 bucket - aws-devops-ebs-java-s3-artifacts - need to verify

8. Similarly, repeat the steps with new change, verify all the above mentioned validation points

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++




