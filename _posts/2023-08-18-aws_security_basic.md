---
layout: single
title:  "aws 보안 기본"
tags:
  - aws
  - security
  - cloudwatch
  - cloudtrail
  - iam
  - policy
categories:
  - aws
  - security
toc: true
toc_sticky: true

---


![](https://velog.velcdn.com/images/yuran3391/post/376d8962-8ff9-42a9-8cc3-530669f02dbf/image.png)

# 1. Identity (인증)
## 누구세요?
- IAM Principal은 AWS 어카운트 내에 정의된 **요청 주체**를 말함
- aws에서의 보안 주체는 aws 어카운트내에 정의된 identity를 의미하는 것으로, 크게 IAM User와 IAM Role로 나눌 수 있음/

### 1.1 IAM User
- 실 사용자 기준으로 통제할 때 IAM User로 인증하며(**상시 자격증명**) 주로 IAM Group으로 관리
![](https://velog.velcdn.com/images/yuran3391/post/129858d9-d0ec-4029-861b-bc01e2cad008/image.png)

#### 1.1.1 IAM Group
- IAM Group은 보안주체가 아니며, IAM 권한을 한번에 주기 위한 용도임
- 그룹간 포함 관계는 불가 (Nested)
- 자동 소속되는 기본 그룹은 없음
- IAM 사용자는 복수개의 그룹에 속할 수 있음 (최대 10개까지, 하드리밋)
![](https://velog.velcdn.com/images/yuran3391/post/f65f16f4-c0f9-4e73-8ca5-f1523e55f299/image.png)

#### 1.1.2 IAM 사용자 유형
- Root User
- IAM User
![](https://velog.velcdn.com/images/yuran3391/post/4766e841-0fcc-4910-9dcc-e1dc145b8771/image.png)

### 1.2 IAM Role
- 자동화된 프로세스에서
- AWS 서비스들에서
- 인증 연계된 외부 사용자들이 
**임시자격증명**으로 인증

#### 1.2.1 Trusted entity type

![](https://velog.velcdn.com/images/yuran3391/post/ea7c0bb5-4393-4a05-8475-cf4b23453796/image.png)

- AWS Service
- AWS 다른 account
- WEB Identity :외부 서비스 관리 시스템
- SAML 2.0 :외부 서비스 관리 시스템
- Custorm Trust Policy :외부 서비스 관리 시스템

![](https://velog.velcdn.com/images/yuran3391/post/2e209961-9403-4a42-adec-c5838c6e1853/image.png)
세션 기본값은 한시간으로 변경 가능

![](https://velog.velcdn.com/images/yuran3391/post/379b7e2a-cc46-4b2a-8fc0-b7f72f908df7/image.png)
AWS 서비스에 롤 부여한 케이스

## 2. Access Management (인가)
### (인증은 받은 주체가)권한을 갖고 있는지?

### 2.1 AWS Policy
- 모든 AWS 서비스는 접근제어 정책을 기반으로 인가됨
- 매 API 호출 시, 적용된 정책을 통해 인가 수행
- 정책은 IAM 역할/사용자/그룹, AWS 리소스, 임시 자격증명 세션, OU 등에 적용할 수 있음
- AWS Root 어카운트는 기본적으로 AWS 리소스에 대한 모든 권한을 가짐
- AWS 정책은 기본 디폴트가 Deny고, 명시적 Allow 보다 명시적 Deny가 우선순위가 높음

정책을 만들때는 아래 링크를 참고하면 됨. 
https://us-east-1.console.aws.amazon.com/iamv2/home?region=ap-northeast-2#/policies/create?step=addPermissions 

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "Statement1",
			"Effect": "Allow",  / Allow | Deny 
			"Action": "s3:GetObject", / 허용 혹은 차단 하고자하는 접근 타입 
			"Resource": "arn:aws:sqs:us-west-2:123454444:queue1" /요청의 목적지가 되는 서비스
		}
	]
}

---
각 의미만 안다면 visualEditor로 쉽게 policy 생성 가능
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "VisualEditor0",
			"Effect": "Allow",
			"Action": [
				"s3:PutBucketLogging",
				"s3:PutAccessPointConfigurationForObjectLambda",
				"s3:DeleteAccessPointPolicy"
			],
			"Resource": [
				"arn:aws:s3:*:970698899539:accesspoint/*",
				"arn:aws:s3:::*",
				"arn:aws:s3-object-lambda:*:970698899539:accesspoint/*"
			],
			"Condition": {
				"Bool": {
					"aws:MultiFactorAuthPresent": "true"
				}
			}
		}
	]
}
```
![](https://velog.velcdn.com/images/yuran3391/post/0ad86ab3-185b-4a7e-83ee-b35d3020a07e/image.png)
최소 권한의 원칙에 맞게 잘 설계해야한다...........ㅠ

요런 Documents들을 잘 읽고 만들어보면 됨
https://docs.aws.amazon.com/service-authorization/latest/reference/list_awsbackup.html 


#### 2.1.1 Identity Based Policy VS Resource Based Policy

![](https://velog.velcdn.com/images/yuran3391/post/dc588da6-e874-4f5d-b7e9-74b2889bdd8b/image.png)
![](https://velog.velcdn.com/images/yuran3391/post/5b5bccf4-6caf-448d-a8fc-6b48927d16c0/image.png)

Identity Based Policy는 요청하는 주체에 연결됨.
Resource Based Policy는 요청을 받은 aws 자원(대상)에 연결됨.


Resource Based Policy 는 자원에 할당되는 정책이니 해당 리소스를 접근할 보안 주체에 대한 지정 즉, Principal이 필수적

![](https://velog.velcdn.com/images/yuran3391/post/31b7c6d7-d21f-471f-9d3e-5c8b8272581d/image.png)
![](https://velog.velcdn.com/images/yuran3391/post/1e9c4ea9-31b4-4d5a-8db8-d31d404d3e2a/image.png)

---
# 3. CloudTrail
모든 api 콜에 대해 기록
![](https://velog.velcdn.com/images/yuran3391/post/5c63bfcf-704a-49d8-a38e-53764ae11d57/image.png)
CloudTrail 로그를 s3에 저장하고 클라우드 와치 로그를 활용해서 간편하게 로그 데이터를 수집 및 검색
클라우드 와치 이벤트와 통합하여 알람 설정

![](https://velog.velcdn.com/images/yuran3391/post/8d428ddd-865a-4b23-ba8a-0d0afa462796/image.png)

클라우드트레일 이벤트 기록은 90일간 행위 이력을 무료로 조회 할 수 있고 이후 삭제 됨으로 다운 받거나 s3로 이관 필요
![](https://velog.velcdn.com/images/yuran3391/post/be4f9ec5-549a-4e6a-8501-c05211eb6ebb/image.png)
![](https://velog.velcdn.com/images/yuran3391/post/4e209186-c8ac-4c27-b955-bbb486ec710d/image.png)

# 4. CloudWatch
![](https://velog.velcdn.com/images/yuran3391/post/90da04bb-71e8-4313-bdbe-48a54e248e16/image.png)
![](https://velog.velcdn.com/images/yuran3391/post/7124747d-ef11-4d63-ad04-acc07a51084c/image.png)
![](https://velog.velcdn.com/images/yuran3391/post/68ed60ae-8899-42d2-8d5e-5869e5ac5086/image.png)


# 5. IAM Best Practice
### 1. Root 사용자 사용 금지
### 2. 권한 높은 IAM User에 대해서는 MFA 활성화
### 3. IAM User 키에 대한 주기적인 교체
### 4. IAM User에 최소 권한 할당
- Access Advisor 를 활용해 일정 기간동안 접근하지 않은 사용자에 대해 점차적 권한 제거
- Access Analyzer로 권한이 과도하게 주어진 리소스 판별
- IAM Credential report 활용
- IAM Policy Simulator로 IAM Policy 사전 검증

https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html

