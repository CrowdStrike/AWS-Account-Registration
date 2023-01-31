# AWS Account Registration via API
  
## Get Bearer Token
```
export FALCON_CLIENT_ID=
export FALCON_CLIENT_SECRET=
export FALCON_CLOUD_API=  # Options: api.crowdstrike.com (us-1), api.us-2.crowdstrike.com (us-2), api.eu-1.crowdstrike.com (eu-1)

export BEARER_TOKEN=$(curl \
--silent \
--header "Content-Type: application/x-www-form-urlencoded" \
--data "client_id=${FALCON_CLIENT_ID}&client_secret=${FALCON_CLIENT_SECRET}" \
--request POST \
--url "https://$FALCON_CLOUD_API/oauth2/token" | \
jq -r '.access_token')
```

## Make Post Request to Register Account
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

### Resources Block Details
|Parameter|Description|Example Input|Required?|
|:-|:-|:-|:-|
|account_id|(String)Your AWS Account Id.|123456789123|Yes|
|account_type|(String)Type of AWS Account to register|commercial or govcloud|Yes|
|cloudtrail_region|(String)|us-east-1|No|
|iam_role_arn|(String)Role ARN to grant read-only permissions in your account.  Leave blank if you would like API to auto-generate rolename.|arn:aws:iam::123456789123:role/my-custom-role|No|
|is_master|(Boolean) Is this an AWS Organization Management Account?|true or false|Yes|
|organization_id|(String)|o-123edc456rfv|No|

## Successful Response Example
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
      "is_custom_rolename": true
    }
  ]
}
```