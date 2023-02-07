# CrowdStrike-CSPM-Integration

## Configures CSPM resources in child accounts.

### Permission Model
SERVICE_MANAGED

## Resources
- CrowdStrikeCSPMRole
- CrowdStrikeEventBridgeRole

## Template
```
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "Setup script to enable CrowdStrike Falcon CSPM.",
  "Parameters": {
    "RoleName": {
      "Description": "Read-only role for CrowdStrike Falcon CSPM",
      "Type": "String",
      "MinLength": "1"
    },
    "ExternalID": {
      "Description": "External ID for created role",
      "Type": "String",
      "MinLength": "2"
    },
    "CSRoleName": {
      "Description": "Name of Crowdstrike assuming role",
      "Type": "String",
      "MinLength": "10"
    },
    "CSAccountNumber": {
      "Description": "Crowdstrike account number",
      "Type": "String",
      "MinLength": "12",
      "MaxLength": "12"
    },
    "EnableIOA": {
      "Type": "String",
      "AllowedValues": [
        "true",
        "false"
      ],
      "Default": "true"
    }
  },
  "Conditions": {
    "ShouldCreateIOAResources": {
      "Fn::Equals": [
        true,
        {
          "Ref": "EnableIOA"
        }
      ]
    }
  },
  "Resources": {
    "CrowdStrikeCSPMRole": {
      "Type": "AWS::IAM::Role",
      "Properties": {
        "AssumeRolePolicyDocument": {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Effect": "Allow",
              "Principal": {
                "AWS": {
                  "Fn::Join": [
                    "",
                    [
                      "arn:aws:iam::",
                      {
                        "Ref": "CSAccountNumber"
                      },
                      ":role/",
                      {
                        "Ref": "CSRoleName"
                      }
                    ]
                  ]
                }
              },
              "Condition": {
                "StringEquals": {
                  "sts:ExternalId": {
                    "Ref": "ExternalID"
                  }
                }
              },
              "Action": [
                "sts:AssumeRole"
              ]
            }
          ]
        },
        "RoleName": {
          "Ref": "RoleName"
        },
        "ManagedPolicyArns": [
          "arn:aws:iam::aws:policy/SecurityAudit"
        ],
        "Policies": [
          {
            "PolicyDocument": {
              "Version": "2012-10-17",
              "Statement": [
                {
                  "Effect": "Allow",
                  "Resource": "*",
                  "Action": [
                    "backup:ListBackupPlans",
                    "backup:ListRecoveryPointsByBackupVault",
                    "ecr:GetRegistryScanningConfiguration",
                    "eks:ListFargateProfiles",
                    "elasticfilesystem:DescribeAccessPoints",
                    "lambda:GetFunction",
                    "sns:GetSubscriptionAttributes"
                  ]
                }
              ]
            },
            "PolicyName": "cspm_config"
          }
        ]
      }
    },
    "CrowdStrikeEventBridgeRole": {
      "Type": "AWS::IAM::Role",
      "Condition": "ShouldCreateIOAResources",
      "Properties": {
        "RoleName": "CrowdStrikeCSPMEventBridge",
        "AssumeRolePolicyDocument": {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Action": "sts:AssumeRole",
              "Principal": {
                "Service": "events.amazonaws.com"
              },
              "Effect": "Allow",
              "Sid": ""
            }
          ]
        },
        "Policies": [
          {
            "PolicyName": "eventbridge-put-events",
            "PolicyDocument": {
              "Version": "2012-10-17",
              "Statement": [
                {
                  "Action": [
                    "events:PutEvents"
                  ],
                  "Resource": {
                    "Fn::Sub": "arn:aws:events:*:*:event-bus/cs-*"
                  },
                  "Effect": "Allow"
                }
              ]
            }
          }
        ]
      }
    }
  }
}
```