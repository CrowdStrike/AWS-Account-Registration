# Horizon - AWS Organization Registration Workflow via Bash

![image](https://user-images.githubusercontent.com/29733103/217326802-77edaced-d0d8-4b53-8208-b56d4503a202.png)

![image](https://user-images.githubusercontent.com/29733103/217326903-210cfc31-bcd5-4d7d-8e47-c5eb036ce4f6.png)

## Prerequisites

1. Falcon API Client and Key with scope: 

|D4C Registration|Read & Write|
|-|-|

2. Access to a user account with admin rights in the AWS Organization Management (CrowdStrike never has access to these credentials)

## Gather Information

- AWS Organization ID
- Desired Role Name (optional)

## Register Org via Bash Script
This command downloads the CrowdStrike Registration script and passes your configuration details upon execution. 
```
AWS_REGION=<your-aws-region> \
BEHAVIOR_ASSESSMENT_ENABLED=<true-or-false> \
USE_EXISTING_CLOUDTRAIL=<true-or-false> \
CUSTOM_ROLE_NAME=<your-custom-role-name> \
ORGANIZATION_ID=<your-aws-organization-id> \
AWS_PROFILE=<your-aws-cli-profile-name> \
CLIENT_ID=<your-falcon-api-client-id> \
CLIENT_SECRET=<your-falcon-api-client-key> \
/bin/bash  -c "$(curl -fsSL https://cs-prod-cloudconnect-templates.s3.amazonaws.com/install_aws.sh $@)" 
```
## DeRegister Org via Bash Script
```
AWS_REGION=<your-aws-region> \
AWS_PROFILE=<your-aws-cli-profile-name> \
ORGANIZATION_ID=<your-aws-organization-id> \
CLIENT_ID=<your-falcon-api-client-id> \
CLIENT_SECRET=<your-falcon-api-client-key> \
/bin/bash -c "$(curl -fsSL https://cs-prod-cloudconnect-templates.s3.amazonaws.com/uninstall_aws.sh $@)"
```

## Generate Bash script via Guided workflow
- Navigate to https://falcon.crowdstrike.com/cloud-security/registration
- Click Add new account and select "Register an Organization account using Bash or Terraform Script"
![image](https://user-images.githubusercontent.com/29733103/217327705-d420f52f-920a-49b6-a6ec-ed6c93bd5fbf.png)
- Now you may follow the prompts to configure the Bash or Terraform
![image](https://user-images.githubusercontent.com/29733103/217327756-29cc4157-f89d-4ba7-a562-6909b732994a.png)
![image](https://user-images.githubusercontent.com/29733103/217327836-ac514ed8-b46f-4f4e-888d-a201c7005122.png)
![image](https://user-images.githubusercontent.com/29733103/217327868-3c842780-4cce-4c4b-ab84-36c163ae386b.png)
![image](https://user-images.githubusercontent.com/29733103/217327906-fd76d172-cfd1-497b-868c-4a9b10b982df.png)


## Generate Bash script via Guided workflow
