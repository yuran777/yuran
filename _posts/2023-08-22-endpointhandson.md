---
layout: single
title:  "aws endpoint 실습 -  AWS Session Manager를 통한 Bastion Server 대체 및 s3 Endpoint 생성을 통한 안전한 private 환경 접근  "
tags:
  - aws
  - ssm
  - s3endpoint
  - vpcendpoint
categories:
  - aws
  - security
toc: true
toc_sticky: true

---


# **SSM 이란?**

AWS Systems Manager(SSM)의 기능으로, 포트가 아닌 IAM 권한을 가지고 통신하는 시스템이다. AWS EC2를 생성하여 이용하다 보면 ssh 접속을 위해 pem key를 발급받거나 기존에 사용하던 pem key를 사용한다. 이런 방식은 pem key가 탈취될 경우 누구든지 AWS EC2에 접속하게 될 수 있기 때문에 보안상 굉장히 위험할 수 있다. 이런 보안 위험 상황을 방지하기 위해서 SSM 서비스를 사용한다.

## **SSM 사용 이점**

- SSH나 RDP를 사용하는 통신이 아니기 때문에, 보안그룹을 모두 막아놓아도 AWS 콘솔에서 EC2에 접속해 관리 가능
- 배스천 호스트를 둘 필요없음
- 인스턴스 별로 Secret Key 관리를 할 필요 없음
- Private Instacne에 바로 접속할 수 있음

## 사전 조건

기본적으로 SSM Agent가 셋업되어있는 운영체제 타입은 아래와 같다.

- 2017년 9월 이후의 Amazon Linux Base AMI
- Amazon Linux 2
- Amazon Linux 2 ECS 최적화 기본 AMIs
- 아마존 리눅스 2023 (AL2023)
- Amazon EKS 최적화 Amazon Linux AMIs
- macOS 10.14.x(Mojave), 10.15.x(Catalina), 11.x(Big Sur)
- SUSE Linux Enterprise Server(SLES) 12 및 15
- Ubuntu Server16.04, 18.04, 20.04 및 22.04
- Windows Server 2008-2012 R2 AMIs는 2016년 11월 이후에 게시되었습니다.
- Windows Server 2016, 2019, 2022

이 외의 운영체제를 사용한다면 SSM Agent를 수동으로 설치해주어야 한다.

[Linux용 EC2 인스턴스에 수동으로 SSM Agent 설치 - AWS Systems Manager](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/sysman-manual-agent-install.html)

## 진행 순서

1. Private Subnet 에 EC2 Instance (Amazon Linux) 생성
    1. No public IP assigned
    2. No key pair
2. IAM Role생성후EC2연결(SSM/S3) 
    1. **AmazonSSMManagedInstanceCore**
        1. 노드가 Systems Manager 서비스 코어 기능을 사용하도록 허용하는 인스턴스 신뢰 정책
        2. 이 AWS 관리형 정책을 사용하면 인스턴스가 Systems Manager 서비스 핵심 기능을 사용할 수 있음
    2. S3Fullaccess
3. VPC Endpoint 설정
    1. ssm
        1. Systems Manager 서비스의 엔드포인트
    2.  ssmmessage 
        1. Session Manager를 사용하여 보안 데이터 채널을 통해 인스턴스에 연결하는 경우에만 필요
    3.  ec2messages 
        1. Systems Manager는 이 엔드포인트를 사용하여 SSM Agent에서 Systems Manager 서비스를 호출
        
4. S3 Endpoint 설정
	1. gateway로

5.  VPC Endpoint Policy **(Option)**
    1. 여러 S3 bucket 중 생성한 VPC Endpoint 에서는 특정 S3 Bucket 만 접근할 수 있도록 적용
    
    ```json
    {
    	"Version": "2008-10-17",
    	"Statement": [
    		{
    			"Effect": "Allow",
    			"Principal": "*",
    			"Action": "*",
    			"Resource": [
    				"허용하는 버킷",
    				"허용하는 버킷/*"
    			]
    		}
    	]
    }
    ```
