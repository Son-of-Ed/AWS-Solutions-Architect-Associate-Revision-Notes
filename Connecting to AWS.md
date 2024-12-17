# Connecting to AWS
There are a few methods for your to make changes in your AWS account:
- AWS CLI - perform commands from your terminal
- CloudShell - perform commands from a terminal in the AWS console *NOTE: limited to certain regions* 
- AWS SDK - interact with AWS through your applications e.g. using boto3 in Python.

Best practice for connecting to AWS dictates that you use a federated credentials with an identity provider so that you are using temporary credentials to access AWS.
Password policies are set at the account level for all users.

#aws