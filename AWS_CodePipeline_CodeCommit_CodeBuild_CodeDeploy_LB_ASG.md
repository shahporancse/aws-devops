# AWS_CodePipeline_CodeCommit_CodeBuild_CodeDeploy_LB_ASG


## Step 1: Credetnails for AWS CodeCommit 
-  Create IAM User
-  Install AWS Cli : https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html
- Configure IAM Configure with Cli
```
aws configure --profile produser
AWS Access Key ID [None]: AKIAI44QH8DHBEXAMPLE
AWS Secret Access Key [None]: je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: text
```

## Step 2. Create Repository in CodeCommit 
 
 - Developer Tools -> CodeCommit -> Repositiories -> Create Repository 
 - Repository Name : aws_cicd
 - Description: Optional 
 - Add Tags : Optional
 
## Step 3: Prerequisites
- https://us-east-2.console.aws.amazon.com/codesuite/codecommit/repositories/das/setup?region=us-east-2
- Changed Repository Name : https://us-east-2.console.aws.amazon.com/codesuite/codecommit/repositories/repository_name/setup?region=us-east-2
```
git add .
git commit -m "first commit"
git push origin master 
git branch -b "create_branch"
git checkout -b "MoveOneBranchtoAnotherBranch"
```

## step 4: CodeCommit - Securing the Repository and Branches

 - create a group (junior_Dev)
 - create Inline policies in the permissions 
 - Policy Name: cannotpushToMasterInCodeCommit
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
