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

## step 6: CodeCommit - Triggers & Notifications

## step 7: CodeCommit - & AWS Lambda



