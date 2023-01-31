# Discover - AWS Account Registration Workflow via API

![image](https://user-images.githubusercontent.com/29733103/215860251-26626a0a-ba0d-4d3a-bfc8-770fffb2f14a.png)


## Prerequisites

1. Falcon API Client and Key with scope: 

|D4C Registration|Read & Write|
|-|-|

2. Access to a user account with admin rights in the AWS account (CrowdStrike never has access to these credentials)

## Gather Information

- AWS Account ID
- Account Type (Commercial or GovCloud)
- Desired Role Name (optional)

## Authenticate with CrowdStrike API and Get Bearer Token
This step can be completed in the Falcon Swagger UI or in your terminal using the following command:
```
export FALCON_CLIENT_ID=
export FALCON_CLIENT_SECRET=
export FALCON_CLOUD_API=api.crowdstrike.com  # api.crowdstrike.com (us-1), api.us-2.crowdstrike.com (us-2), api.eu-1.crowdstrike.com (eu-1)

export BEARER_TOKEN=$(curl \
--silent \
--header "Content-Type: application/x-www-form-urlencoded" \
--data "client_id=${FALCON_CLIENT_ID}&client_secret=${FALCON_CLIENT_SECRET}" \
--request POST \
--url "https://$FALCON_CLOUD_API/oauth2/token" | \
jq -r '.access_token')
```

## Example API Calls
In this example we are registering a single commercial account, and passing a custom IAM Role Name.
```
curl -X 'POST' \
  'https://api.crowdstrike.com/cloud-connect-aws/entities/account/v2' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer $BEARER_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
  "resources": [
    {
      "account_id": "123456789123",
      "account_type": "commercial",
      "iam_role_arn": "arn:aws:iam::123456789123:role/my-custom-role",
      "is_master": false
    }
  ]
}'
```
A successful response will contain important information, such as your iam_role_arn, The CrowdStrike intermediate_role_arn, external_id, your account registration status and the cloudformation_url to easily launch the stack in your AWS account.

```
{
  "meta": {
    "query_time": 0.060088499,
    "writes": {
      "resources_affected": 1
    },
    "powered_by": "cspm-registration",
    "trace_id": "123-abc-456-def-789"
  },
  "errors": [],
  "resources": [
    {
      "ID": 25080,
      "CreatedAt": "2023-01-01T01:01:01",
      "UpdatedAt": "2023-01-01T01:01:01",
      "DeletedAt": null,
      "cid": "1qaz2wsx3edc4rfv",
      "account_id": "123456789123",
      "iam_role_arn": "arn:aws:iam::123456789123:role/my-custom-role",
      "intermediate_role_arn": "arn:aws:iam::321654987321:role/CrowdStrikeCSPMConnector",
      "external_id": "1qaz2wsx3edc4rfv5tgb6yhn",
      "status": "Event_DiscoverAccountStatusProvisioned",
      "is_master": false,
      "aws_permissions_status": [],
      "cloudformation_url": "https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://cs-prod-cloudconnect-templates.s3-us-west-1.amazonaws.com/aws_cspm_cloudformation_d4c_v2.json&stackName=CrowdStrike-CSPM-Integration&param_ExternalID=1qaz2wsx3edc4rfv5tgb6yhn&param_RoleName=my-custom-role&param_CSRoleName=CrowdStrikeCSPMConnector&param_CSAccountNumber=321654987321",
      "account_type": "commercial",
      "settings": {},
      "valid": true,
      "is_custom_rolename": false
    }
  ]
}
```

In this example we are registering a single commercial account, but we are allowing CrowdStrike to generate a random role name.
```
curl -X 'POST' \
  'https://api.crowdstrike.com/cloud-connect-aws/entities/account/v2' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer $BEARER_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
  "resources": [
    {
      "account_id": "123456789123",
      "account_type": "commercial",
      "is_master": false
    }
  ]
}'
```
A successful response will now contain a role named CrowdStrikeCSPMReader with a randoml;y generated string suffix.  This role name is required for the Discover service to successfully complete registration and perform scans on the account.

```
{
  "meta": {
    "query_time": 0.060088499,
    "writes": {
      "resources_affected": 1
    },
    "powered_by": "cspm-registration",
    "trace_id": "123-abc-456-def-789"
  },
  "errors": [],
  "resources": [
    {
      "ID": 25080,
      "CreatedAt": "2023-01-01T01:01:01",
      "UpdatedAt": "2023-01-01T01:01:01",
      "DeletedAt": null,
      "cid": "1qaz2wsx3edc4rfv",
      "account_id": "123456789123",
      "iam_role_arn": "arn:aws:iam::123456789123:role/CrowdStrikeCSPMReader-abc123def456",
      "intermediate_role_arn": "arn:aws:iam::321654987321:role/CrowdStrikeCSPMConnector",
      "external_id": "1qaz2wsx3edc4rfv5tgb6yhn",
      "status": "Event_DiscoverAccountStatusProvisioned",
      "is_master": false,
      "aws_permissions_status": [],
      "cloudformation_url": "https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://cs-prod-cloudconnect-templates.s3-us-west-1.amazonaws.com/aws_cspm_cloudformation_d4c_v2.json&stackName=CrowdStrike-CSPM-Integration&param_ExternalID=1qaz2wsx3edc4rfv5tgb6yhn&param_RoleName=CrowdStrikeCSPMReader-abc123def456&param_CSRoleName=CrowdStrikeCSPMConnector&param_CSAccountNumber=321654987321",
      "account_type": "commercial",
      "settings": {},
      "valid": true,
      "is_custom_rolename": false
    }
  ]
}
```

## Provision the IAM Role
At this point you will see your account on the Cloud Security Account Registration page in Falcon, but the account will show as inactive.

![image](https://user-images.githubusercontent.com/29733103/215858141-d26adb2d-42e0-4413-b2ae-aec3056ba874.png)

To complete the registration, the cloudformation_url included in the API response can be used to automatically create a Cloudformation stack with all required parameters pre-filled to deploy the IAM role in your account.  The IAM role sets read only permissions required for the Discover Service while the intermediate_role_arn & external_id are used in the IAM role trust policy to ensure only CrowdStrike can assume the role.
  
 You may instead review the permissions required in the role and create the role according to your organization's IAM management policies.
  
The required permissions:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeImages",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeVolumes",
                "ec2:DescribeVpcs",
                "ec2:DescribeRegions",
                "ec2:DescribeSubnets",
                "ec2:DescribeNetworkAcls",
                "ec2:DescribeSecurityGroups",
                "iam:ListAccountAliases",
                "organizations:ListAccounts"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```
The required trust policy:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "intermediate_role_arn_from_api_response"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringEquals": {
                    "sts:ExternalId": "external_id_from_api_response"
                }
            }
        }
    ]
}
```

Once the role is created with the required permissions and trust policy, your account will show as 'Active' in the Falcon console.
