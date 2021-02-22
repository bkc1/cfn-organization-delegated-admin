# Cloudformation Stackset - Provision IAM role/policies across AWS Organizations


## Overview 

The YAML Cloudformation stackset template is intended to be a reference/example of how IAM roles/policies can be deployed from an Organization-delegated administrator account across all member accounts within the Org or within specific OUs.  It will provision an IAM Role with optional managed policies attached based on job junction and AWS support center access needs. Optionally a custom(in-line) reference IAM policy can also be associated with the role. The example in-inline IAM policy in this template is for the EC2 service, granting read access to all CloudWatch logs. 

This CFN template was borrowed and refactored from: https://github.com/1Strategy/iam-starter-templates/tree/master/iam-roles-and-policies. 


## Prereqs & Dependencies

Follow the setup in blog and/or documentation below to enable trusted access with AWS Organizations and register a delegated administrator account.

- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-enable-trusted-access.html
- https://aws.amazon.com/blogs/mt/cloudformation-stacksets-delegated-administration/

## Usage 

From the delegated admin account Cloudformation console, ceate a new 'service-managed' stackset using the CFN YAML template. 

- Give the Stackset a name (i.e. OrgIAMrole-<job-function>) 
- Configure CFN parameters:
  - Choose a managed policy based on job function (optional)
  - Set path if grouping (default /)
  - Set Name for in-line policy (i.e EC2-Cloudwatch-readonly) 
  - Set Custom Role Name (i.e. OrgDataScientistRole)
  - Enter Account ID of delegated administrator
  - Use managed policy for AWS support access or a custom policy(must pre-exist)
- Select 'Service-managed permissions'
- Select 'Deploy to Organization'
- Select region 'US East(N.Virginia)'. Note only one region is needed since IAM is a global service. 

### Configure AWS CLI AssumeRole from delegated admin account.

The CFN template sets the delegated admin account as the Principal in the Trust poilcy, allowing cross-account AssumeRole permissions.  From the CFN stack outputs, note the `RoleARN` and replace Account ID with the ID of the member account where you are assuming the role. Note that Secret access keys must be setup for the `org-admin` profile that will be used as the `source_profile` of the `member-acct`.   

```
[profile org-admin]
region = us-east-1
output = json

[profile member-acct]
role_arn = arn:aws:iam::<Account ID>:role/<Role Name>
source_profile = org-admin
region = us-east-1
output = json
```

Validate the `AssumeRole` access by running a AWScli command using the `member-acct` profile 
```
$ aws iam list-roles --profile member-account | jq '.Roles[] | select(.RoleName == "<Role Name>")'
```

### A few useful commands for reference:

Find the Org parent IDs and OU IDs:
```
$ aws organizations list-roots
$ aws organizations list-organizational-units-for-parent --parent-id r-XXXX
```

Register an Org deletgated admin(from mgmt account)
```
$ aws organizations register-delegated-administrator --account-id XXXXXXXXXX --service-principal=member.org.stacksets.cloudformation.amazonaws.com
$ aws organizations list-delegated-administrators
```


