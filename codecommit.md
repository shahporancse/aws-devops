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

- **1. Create Notification**
    - Notification name: codecommit-notification
    - Detail type: full
    - Events that trigger notifications: Select all
    - Targets: SNS/AWS Slack BOT

- **2. Create trigger**
    - Trigger name: codecommit-trigger
    - Events: default
    - Events that trigger notifications: Select all
    - Service details: Choose the service to use: SNS/Lambda

- **3. Cloudwath event Rules**
    - Services Name: Codecommit
    - Event Type: default
    - Add Target: Lambda/SNS


## step 7: CodeCommit - & AWS Lambda with Slack
- https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify-lambda.html
- https://www.youtube.com/watch?v=5UC8adwht4k
- https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify-lambda.html

```
import os
import json
import boto3
import dateutil.parser
import logging
from botocore.vendored import requests

# Initialize Logger
logger = logging.getLogger()
logger.setLevel(logging.INFO)


def set_global_vars():
    """
    Set the Global Variables
    If User provides different values, override defaults

    This function returns the AWS account number

    :return: global_vars
    :rtype: dict
    """
    global_vars = {'status': False}
    try:
        global_vars['Owner'] = "Mystique"
        global_vars['Environment'] = "Prod"
        global_vars['tag_name'] = "serverless-code-commit-repo-event-notifier"
        global_vars['slack_webhook_url'] = os.environ.get("slack_webhook_url")
        global_vars['sns_topic_arn'] = os.environ.get("sns_topic_arn")
        global_vars['status'] = True
    except Exception as err:
        logger.error(f"Unable to set global variables. ERROR:{err}")
        global_vars['error_message'] = str(err)
    return global_vars


def post_to_slack(webhook_url, slack_data, record):
    """
    Post message to given slack channel/url

    :param webhook_url: The lambda event
    :param type: str
    :param slack_data: A json containing slack performatted text data
    :param type: json

    :return: key_exists Returns True, If key exists, False If Not.
    :rtype: bool
    """
    resp = {'status': False}
    slack_msg = {}
    slack_msg["text"] = ''
    slack_msg["attachments"] = []
    tmp = {}

    tmp["fallback"] = "Event Detected."
    tmp["color"] = record.get("color")
    tmp["pretext"] = f"CodeCommit Event detected in `{record.get('awsRegion')}` in Repo:`{record.get('eventSourceARN').split(':')[-1]}`"
    tmp["author_name"] = "Serverless-Repo-Monitor"
    tmp["author_link"] = "https://github.com/miztiik/serverless-code-commit-repo-event-notifier"
    tmp["author_icon"] = "https://avatars1.githubusercontent.com/u/12252564?s=400&u=20375d438d970cb22cc4deda79c1f35c3099f760&v=4"
    tmp["title"] = f"Repo Event: {record.get('eventName')}"
    tmp["title_link"] = f"https://console.aws.amazon.com/codesuite/codecommit/repositories/{record.get('eventSourceARN').split(':')[-1]}/browse?region={record.get('awsRegion')}"
    tmp["fields"] = [
        {
            "title": "Repo Name",
            "value": f"`{slack_data.get('repo_metadata').get('repositoryName')}`",
            "short": True
        },
        {
            "title": "Repo Default Branch",
            "value": f"`{slack_data.get('repo_metadata').get('defaultBranch')}`",
            "short": True
        },
        {
            "title": "Triggered-By",
            "value": f"`{record.get('userIdentityARN')}`",
            "short": True
        },
        {
            "title": "Trigger Name",
            "value": f"`{record.get('eventTriggerName')}`",
            "short": True
        }
    ]
    tmp["footer"] = "AWS CodeCommit"
    tmp["footer_icon"] = "https://raw.githubusercontent.com/miztiik/serverless-code-commit-repo-event-notifier/master/images/aws-code-commit-logo.png"
    tmp["ts"] = int(dateutil.parser.parse(record.get('eventTime')).timestamp())
    tmp["mrkdwn_in"] = ["pretext", "text", "fields"]
    slack_msg["attachments"].append(tmp)
    logger.info(json.dumps(slack_msg, indent=4, sort_keys=True))

    # slack_payload = {'text':json.dumps(i)}
    try:
        p_resp = requests.post(webhook_url, data=json.dumps(
            slack_msg), headers={'Content-Type': 'application/json'})
        resp["status"] = True
    except Exception as e:
        logger.error(f"ERROR:{str(e)}")
        resp["error_message"] = f"ERROR:{str(e)}"

    if p_resp.status_code < 400:
        logger.info(
            f"INFO: Message posted successfully. Response:{p_resp.text}")
        resp["error_message"] = f"{p_resp.text}"
    elif p_resp.status_code < 500:
        logger.error(f"Unable to post to slack. ERROR: {p_resp.text}")
        resp["error_message"] = f"{p_resp.text}"
    else:
        logger.error(f"Unable to post to slack. ERROR: {p_resp.text}")
        resp["error_message"] = f"{p_resp.text}"
    return resp


def lambda_handler(event, context):
    # Can Override the global variables using Lambda Environment Parameters
    logger.info(f"event:  {str(event)}")
    global_vars = set_global_vars()
    resp = {"status": False, "error_message": ''}

    if not global_vars.get('status'):
        resp["error_message"] = global_vars.get('error_message')
        return resp

    # Log the updated references from the event
    codecommit = boto3.client('codecommit')
    references = {reference['ref']
                  for reference in event['Records'][0]['codecommit']['references']}
    logger.info(f"References:  {str(references)}")

    # Get the repository from the event and show its git clone URL
    repository = event['Records'][0]['eventSourceARN'].split(':')[5]
    try:
        response = codecommit.get_repository(repositoryName=repository)
        event['repo_metadata'] = response.get('repositoryMetadata')
        logger.info(
            f"Clone URL: {response['repositoryMetadata']['cloneUrlHttp']}")
        # Update backup status to SNS Topic
        if global_vars.get('slack_webhook_url'):
            for record in event.get("Records"):
                post_to_slack(global_vars.get(
                    'slack_webhook_url'), event, record)
        logger.info(f"slackData:  {str(event)}")

        resp['repo_data'] = response['repositoryMetadata']
        resp['status'] = True
    except Exception as err:
        logger.error(
            f"Error getting repository: {repository}. Make sure it exists and that your repository is in the same region as this function. ERROR: {str(err)}")
        resp['error_message'] = str(err)

    resp['repo_data']['creationDate'] = None
    resp['repo_data']['lastModifiedDate'] = None

    logger.info(f"resp:  {str(resp)}")

    return resp
```
## Reference
https://www.atlassian.com/git/tutorials/using-branches


https://docs.aws.amazon.com/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-conditional-branch.html
https://aws.amazon.com/blogs/devops/refining-access-to-branches-in-aws-codecommit/

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-notify.html

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-repository-email.html )

https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-migrate-repository-existing.html


