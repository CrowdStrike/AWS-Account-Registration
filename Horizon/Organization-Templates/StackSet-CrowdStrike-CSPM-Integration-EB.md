# CrowdStrike-CSPM-Integration-EB

## Create EventBridge rule in child accounts in every region to send CloudTrail events to CrowdStrike

### Permission Model
SERVICE_MANAGED

## Resources
- CrowdStrikeEventBusRule
- CrowdStrikeEventBusRuleRO

## Template
```
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "Setup script to enable CrowdStrike Falcon CSPM.",
  "Parameters": {
    "CSAccountNumber": {
      "Description": "Crowdstrike account number",
      "Type": "String",
      "MinLength": "12",
      "MaxLength": "12"
    },
    "CSEventBusName": {
      "Description": "CrowdStrike Event Bridge Name",
      "Type": "String"
    },
    "DefaultEventBusRegion": {
      "Type": "String",
      "Default": "us-east-1"
    }
  },
  "Resources": {
    "CrowdStrikeEventBusRule": {
      "Type": "AWS::Events::Rule",
      "Properties": {
        "Name": "cs-cloudtrail-events-ioa-rule",
        "EventPattern": {
          "detail-type": [
            "AWS API Call via CloudTrail",
            "AWS Console Sign In via CloudTrail",
            "AWS Service Event via CloudTrail"
          ],
          "detail": {
            "eventName": [
              {
                "anything-but": [
                  "InvokeExecution"
                ]
              }
            ],
            "readOnly": [
              false
            ]
          }
        },
        "State": "ENABLED",
        "Targets": [
          {
            "Arn": {
              "Fn::Sub": "arn:aws:events:${DefaultEventBusRegion}:${CSAccountNumber}:event-bus/${CSEventBusName}"
            },
            "RoleArn": {
              "Fn::Sub": "arn:aws:iam::${AWS::AccountId}:role/CrowdStrikeCSPMEventBridge"
            },
            "Id": "CrowdStrikeCentralizeEvents"
          }
        ]
      }
    },
    "CrowdStrikeEventBusRuleRO": {
      "Type": "AWS::Events::Rule",
      "Properties": {
        "Name": "cs-cloudtrail-events-readonly-rule",
        "EventPattern": {
          "detail-type": [
            "AWS API Call via CloudTrail",
            "AWS Console Sign In via CloudTrail",
            "AWS Service Event via CloudTrail"
          ],
          "detail": {
            "readOnly": [
              true
            ]
          }
        },
        "State": "ENABLED",
        "Targets": [
          {
            "Arn": {
              "Fn::Sub": "arn:aws:events:${DefaultEventBusRegion}:${CSAccountNumber}:event-bus/${CSEventBusName}"
            },
            "RoleArn": {
              "Fn::Sub": "arn:aws:iam::${AWS::AccountId}:role/CrowdStrikeCSPMEventBridge"
            },
            "Id": "CrowdStrikeCentralizeEvents"
          }
        ]
      }
    }
  }
}
```