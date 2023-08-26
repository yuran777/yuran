---
layout: single
title:  "aws IAM"
tags:
  - aws
  - iam
  - policy
categories:
  - aws
  - security
toc: true
toc_sticky: true

---

# IAM (Identity And access Management) 
거의 AWS의 끝과 시작이라 할 수 있다! 
누가 (사용자/ 어플리케이션/ Role) 무엇에(AWS 리소스) 접근할(권한) 수 있는가를 정의하는 것!!


![](https://velog.velcdn.com/images/yuran3391/post/376d8962-8ff9-42a9-8cc3-530669f02dbf/image.png)

# 1. Identity (인증)
## 누구세요?
- IAM Principal은 AWS 어카운트 내에 정의된 **요청 주체**를 말함
- aws에서의 보안 주체는 aws 어카운트내에 정의된 identity를 의미하는 것으로, 크게 Root User , IAM User와 IAM Role, Application으 로 나눌 수 있음/

#### 1.1.2 IAM 사용자 유형
- Root User
  - 모든 권한을 갖고 있음
  - 계정 생성 후 사용하지 않는 것을 권장
- IAM User
  - 장기 자격 증명 (access key / secret key)
  - 주로 IAM 그룹으로 그룹화하여 관리
- Role
  - 임시 자격 증명 (Access Key / Secret Key / Token)
  - 정의된 권한 범위 내 AWS API를 사용 가능
-  어플리케이션 
	- 사용자 대신 권한을 수행하는 경우
	- Role을 이용하여 임시자격증명을 가지고 수행

![](https://velog.velcdn.com/images/yuran3391/post/4766e841-0fcc-4910-9dcc-e1dc145b8771/image.png)

### 1.2 IAM User
- 실 사용자 기준으로 통제할 때 IAM User로 인증하며(**상시 자격증명**) 주로 IAM Group으로 관리
![](https://velog.velcdn.com/images/yuran3391/post/129858d9-d0ec-4029-861b-bc01e2cad008/image.png)

#### 1.2.1 IAM Group
- IAM Group은 보안주체가 아니며, IAM 권한을 한번에 주기 위한 용도임
- 그룹간 포함 관계는 불가 (Nested)
- 자동 소속되는 기본 그룹은 없음
- IAM 사용자는 복수개의 그룹에 속할 수 있음 (최대 10개까지, 하드리밋)
![](https://velog.velcdn.com/images/yuran3391/post/f65f16f4-c0f9-4e73-8ca5-f1523e55f299/image.png)

### 1.3 IAM Role
- 자동화된 프로세스에서
- AWS 서비스들에서
- 인증 연계된 외부 사용자들이 
**임시자격증명**으로 인증
- 코드에 하드 코딩하지 않고 실행 시에 임시(+Token, 일정 시간 이후 만료됨) 자격 증명 사용하며, 이를 Assume 라고 함

#### 1.3.1 Trusted entity type

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
- 필수사항
	- Version:정책의 버전을 나타냅니다.(변수사용을위해서는2012-10-17버전을사용하여야함) 
	- Statement: 정책에서 정의할 모든 규칙 배열(Rule array)을 포함합니다
	- Effect:규칙에서 행위(Action)를 허용 할 것인지(Allow),거부할 것 인지(Deny)를표현합니다
	- Action:규칙 평가의 대상이 되는 행위를 포함합니다
	- Resource:규칙의 대상이되는 AWS의 자원을 포함합니다. 
- 선택사항
	- Sid: 규칙배열에서 각 규칙을 구분할 수 있도록 ID를 사용할 수 있습니다
	- Principal:리소스 정책과 신뢰 정책에서만 사용되며,규칙을적용하는 주체를 포함 할 수있습니다. 
	- Condition:규칙평가 시 특정조건(들)을 통한 제약을 추가 할 수있습니다.

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

- Identity Based Policy는 요청하는 주체에 연결됨.
- Resource Based Policy는 요청을 받은 aws 자원(대상)에 연결됨.

	- 주로 사용되는 정책은 자격 증명 기반 정책과 리소스 기반 정책이 있습니다.
	- 자격 증명 기반 정책은 IAM 사용자, IAM 역할과 같이 IAM 자격 증명에 연결할 수 있는 권한 정책입니다.
	- 리소스 기반 정책은 Amazon S3 버킷과 같은 리소스에 연결하는 권한 정책입니다.


Resource Based Policy 는 자원에 할당되는 정책이니 해당 리소스를 접근할 보안 주체에 대한 지정 즉, Principal이 필수적

![](https://velog.velcdn.com/images/yuran3391/post/31b7c6d7-d21f-471f-9d3e-5c8b8272581d/image.png)
![](https://velog.velcdn.com/images/yuran3391/post/1e9c4ea9-31b4-4d5a-8db8-d31d404d3e2a/image.png)



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

