# codecommit:

## Step 1: Create IAM User 

## Step 2: Install AWS Cli 

https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html

## Step 3: Configure IAM Configure with Cli

```
aws configure --profile produser
AWS Access Key ID [None]: AKIAI44QH8DHBEXAMPLE
AWS Secret Access Key [None]: je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: text
```

## Step 4: Create project with codecommit
https://us-east-2.console.aws.amazon.com/codesuite/codecommit/repositories/aws-practice-codecommit-project-1/setup?region=us-east-2

```

git add .
git commit -m "first commit"
git push origin master 
git branch -b "create_branch"
git checkout -b "MoveOneBranchtoAnotherBranch"

```


## step 5: CodeCommit - Securing the Repository and Branches

 - step 1: create a group (junior_Dev)
 - step 2: create Inline policies in the permissions 
 - step 3: Policy Name: cannotpushToMasterInCodeCommit
 ```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "codecommit:GitPush",
                "codecommit:DeleteBranch",
                "codecommit:PutFile",
                "codecommit:MergeBranchesByFastForward",
                "codecommit:MergeBranchesBySquash",
                "codecommit:MergeBranchesByThreeWay",
                "codecommit:MergePullRequestByFastForward",
                "codecommit:MergePullRequestBySquash",
                "codecommit:MergePullRequestByThreeWay"
            ],
            "Resource": "arn:aws:codecommit:u*:*:*",
            "Condition": {
                "StringEqualsIfExists": {
                    "codecommit:References": [
                        "refs/heads/master", 
                        "refs/heads/prod",
                        "refs/heads/stage"

                     ]
                },
                "Null": {
                    "codecommit:References": "false"
                }
            }
        }
    ]
}

```

 - step 4: Add junior_dev group with user. 



## step 6: CodeCommit - Triggers & Notifications
https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-repository-email.html

-**Notification**
    -Notification name: codecommit-notification

    -Detail type: full

    -Events that trigger notifications: Select all

    -Targets: SNS/AWS Slack BOT

-**Create trigger**
    -Trigger name: codecommit-trigger

    -Events: default

    -Events that trigger notifications: Select all

    -Targets: SNS/AWS Slack BOT

-**Service details**

    -Choose the service to use: SNS/Lambda

-**Cloudwath event Rules**

    -Services Name: Codecommit

    -Event Type: default

-**Cloudwath Source Target**
    -Add Target

## step 7: CodeCommit - & AWS Lambda



## Reference


https://www.atlassian.com/git/tutorials/using-branches


https://docs.aws.amazon.com/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-conditional-branch.html
https://aws.amazon.com/blogs/devops/refining-access-to-branches-in-aws-codecommit/

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify.html

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-repository-email.html )

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify-lambda.html

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-migrate-repository-existing.html


