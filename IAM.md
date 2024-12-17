# IAM
Users, groups, and roles can be assigned JSON IAM policies which allow/restrict the actions they can perform in AWS.
A user can belong to many groups but groups cannot belong to other groups.
Roles can be assigned to users or groups.
Roles are also useful to assign access for a service to other AWS services e.g. allowing a Lambda function Read access to an S3 bucket.

When you assume a new role, you give up your previous permissions and have the permissions of the currently assumed role.
If you don't want a user to lose their permissions when performing a certain action, use resource-based policies to allow certain users to interact with objects rather than getting them to assume a new role.

The owner of a resource is the IAM user who authorised the creation of a resource. The root account is the only account that can perform certain account management actions. Admin users with wildcard access to all resources cannot perform all account management actions.
#### Best Practices
- Only use the root user for the account setup. Administration going forward should be performed by users with "admin" permissions.
- Assign one AWS user to one physical user.
- Give permissions to services using roles.
- Use access keys for programmatic access (CLI/SDK).
- Create a strong password policy + enforce MFA
#### Policies
Policies consist of the following:
- Version - policy language version, always set to "2012-10-17"
- (Optional) ID - identifier for the policy
- Statement - can have one or many. They consist of one or many sections that contain the following:
	- (Optional) SID - identifier for permissions grouped together e.g. S3 perms
	- Action - list of actions to include e.g. `s3:GetBucket`
	- Effect - set to allow or deny
	- Resource - for which resources does these permissions apply
	- Principal - account/user/role to apply this principal to
	- (Optional) Condition - sets when the section is active or not based on if you get a match:
		- `"NotIpAddress": { "aws:SourceIp": ["..."]}` - section is active when client IP making the API call is not within the given list.
		- `"StringEquals": { "aws:RequestedRegion": ["..."]}` - section is active the API call comes from a specified region.
		- `"StringEquals": { "ec2:ResourceTag/Project": "..."}` - section is active when your EC2 resource has a tag matching the one specified.
		- `"StringEquals": { "aws:PrincipalTag/Department": "..."}` - section is active when the user making the API call has a tag on their user matching the one specified.
		- `"StringEquals": { "aws:PrincipalOrgId": ["o-..."]}` - section is active when the user making the API call comes from a specified AWS Organisation.
		- `"BoolIfExists": { "aws:MultiFactorAuthPresent": false}` - section is active when a user making the API call doesn't have MFA set up (useful for denying certain actions).
IAM statements *must* include at least an action (list of actions to do/not do) and an effect (allow/deny).
#### Permission Boundaries
Can set permission boundaries to set the maximum allowed policies that can be added to a user. Therefore, if anyone tried to add additional permissions outside of the permissions boundary to a user/role, the permissions will not be added. This prevents people assigning arbitrary permissions to a user/role but allows users more autonomy over managing their permissions.

The evaluation logic of permissions follows as such:
![](./Pictures/IAMEvaluationLogic.png)
Any explicit denies in any of the policies being used will instantly deny the action.

## Credentials Report
Creates a CSV that summarises useful information about the users within your account such as: they have not updated their password in X days, they don't have MFA set up etc.
## Access Advisor
Review permissions assigned to a user.
## Security Token Service
Create short-term credentials with limited privileges so that you can provide access to AWS resources on an as-and-when basis rather than giving those privileges permanently.
This is useful for giving AWS permissions to EC2 instances and cross-account access.
## Cognito
Create policies for your application users (potentially in the millions) from a database of those users who may be external to AWS.
#### Cognito User Pools
- Sign in functionality for app users which integrates with API Gateway + ALB.
- Uses features such as password resets, email + phone verification, MFA, as well as using federated identities from places like Facebook/Google.
- Can take load away from backend that was previously used for identity verification by evaluating a users's access at the point of hitting your load-balancing component.
#### Cognito Identity Pools (Federated Identity)
- Provide temporary AWS creds to users so they can directly use AWS resources. 
- Can use Cognito user pools as its identity provider or a 3rd party login service.
- IAM perms can be customised for each user ID.

## IAM Identity Centre (formerly Single Sign-On)
Single sign-on for AWS accounts in your organisation/any application that has SAML2.0 enabled/EC2 Windows instance.
The identity centre is hosted in the management account for your AWS Organisation.
Has a built-in identity store but you can use a 3rd party service such as Active Directory, OneLogin or Okta.
Users in an identity store will have permissions sets assigned to them which defines which AWS accounts/apps they have access to.
The permissions sets also define which roles a user can assume upon entry to an AWS account console.
You can define attribute-based access control (ABAC) which allows you to use fine-grained permissions based on a user's attributes e.g. cost centre, title, locale etc.

1. You login in on the AWS website and your user identity is retrieved from an identity store.
2. You are taken to the identity centre which lists the AWS accounts/cloud apps you have access to.
3. You click on an AWS account/application and are then taken to the console for that account/app.

## Directory Services
Used to extend Microsoft Active Directory which is a centralised way of managing objects such as users, computer, printers, file share.
This provides you with an easier way to manage permissions and security.
Objects are organised into "trees" and a group of tree is called a "forest".
#### Managed Microsoft AD
Create your own AD in AWS, manage users locally, and has MFA support.
Establishes trust relationships with your on-prem AD.
Objects managed in both AWS Managed AD and in on-prem AD.
#### AD Connector
Directory gateway (proxy) to redirect to on-prem AD with support for MFA.
Objects solely exist in on-prem AD.
#### Simple AD
AD-compatible managed directory on AWS.
Cannot be joined with on-prem AD.
Objects solely managed in AWS.

There are a couple of ways to connect your AD to IAM Identity Centre depending on if you are hosted your directory objects in AWS or not:
- For objects hosted entirely in Managed Microsoft AD or Simple AD, the integration with IAM Identity Centre is available out of the box.
- For objects hosted partially or entirely on-prem then there are two option:
	- If you are using Managed Microsoft AD to manage some of your directory objects, you need to establish a two-way trust relationship from Managed Microsoft AD in AWS to your on-prem AD.
	- If you are using AD Connector, it will proxy any requests to your on-prem AD.



#aws #iam