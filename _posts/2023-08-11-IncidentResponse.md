본 실습과정에서는 NIST CSF 프레임웍의 Detect, Respond, Recover 및 AWS CAF 프레임웍의 Detective, Responsive 영역에 해당하는 시나리오를 진행하게 됩니다.

 NIST CSF 프레임웍의 Detect, Respond, Recover
 Cybersecurity Framework Components
 
 ![](https://velog.velcdn.com/images/yuran3391/post/cac36a0d-6b38-40a4-80aa-94336faac3b9/image.png)


Amazon GuardDuty  - Amazon GuardDuty는 AWS 계정과 EC2 워크로드를 보호하기 위한 지능형 위협 탐지 및 지속적인 모니터링 서비스를 기계 학습(Machine Learning)기반으로 제공합니다.
![](https://velog.velcdn.com/images/yuran3391/post/08c6bf6e-a0c5-444f-8db1-455b296aab9a/image.png)


Amazon Macie  - Amazon Macie는 S3 상의 민감한 중요 데이터를 발견하고 분류 및 경보 기능을 기계 학습(Machine Learning)기반으로 제공하는 서비스입니다.
![](https://velog.velcdn.com/images/yuran3391/post/23c8f5e0-14a5-458f-9eca-8f88d64e88ae/image.png)


Amazon Inspector  - Amazon Inspector는 EC2 상에서 운영되는 고객 애플리케이션 환경에 대해 취약점을 찾아주는 기능을 제공합니다.
![](https://velog.velcdn.com/images/yuran3391/post/9c59ca94-ffa9-4d2e-8b99-e716932450cb/image.png)

Amazon Detective  - Amazon Detective는 GuardDuty의 탐지 내역이나 VPC Flow Logs, CloudTrail 상의 의심스러운 행위에 대한 직접적인 원인을 빠르게 식별하고 분석(Root Cause Analysis)할 수 있는 환경을 제공합니다.
![](https://velog.velcdn.com/images/yuran3391/post/89cd51a7-b320-43d3-818c-90dfa7b5727c/image.png)

AWS SecurityHub  - AWS Security Hub는 멀티 어카운트 환경에서 주요 보안 경보 및 규정 준수 상태에 대한 광범위하고 통합적인 뷰를 제공합니다. GuardDuty, Inspector, Macie 같은 AWS 보안 서비스뿐만 아니라 다른 파트너 제품과의 연동도 지원하며 각 서비스들이 탐지한 내역들에 대한 요약 및 통합 대쉬보드를 통해 상시 감사체계를 구축할 수 있도록 해줍니다.
![](https://velog.velcdn.com/images/yuran3391/post/7e989079-d393-4217-9baf-af88c61b06fd/image.png)


AWS Config  - AWS Config는 AWS 리소스의 구성 변경 내역을 기록하고 이것이 규정에 맞는지 감사할 수 있는 기능을 제공하는 서비스입니다.
![](https://velog.velcdn.com/images/yuran3391/post/5e12b50e-ddfb-42b1-8f71-1314238f7952/image.png)

Amazon CloudWatch  - Amazon CloudWatch는 AWS 리소스와 AWS상의 고객 애플리케이션들을 모니터링하기 위해 메트릭 수집, 로그 파일 모니터링, 경보 설정, 변경에 대한 자동화된 대응 등의 기능을 제공합니다.
![](https://velog.velcdn.com/images/yuran3391/post/4b3e89d9-c225-42bd-bc61-1be0c677172b/image.png)

AWS CloudTrail  - AWS CloudTrail은 AWS 어카운트 상의 거의 모든 API 호출 이력을 로깅하는 기능을 제공합니다.
![](https://velog.velcdn.com/images/yuran3391/post/21d47410-c923-4199-812c-2afb92c3b6b5/image.png)


AWS IAM  - AWS IAM(Identity and Access Management)는 AWS 사용자 및 그룹을 만들고 관리하며, 권한을 사용해 AWS 리소스에 대한 액세스를 허용 및 거부할 수 있는 권한 관리 기능을 제공합니다.
![](https://velog.velcdn.com/images/yuran3391/post/703d69c0-c62d-4931-a07e-5f6a0632ead5/image.png)

### Overall Architecture

![](https://velog.velcdn.com/images/yuran3391/post/d6f41d1b-afe3-42f0-9c5b-8ed27bd95811/image.png)

1. 실습 환경 구성 및 설정
전체 환경을 구성해 주는 CloudFormation 템플릿을 서울 리전에서 실행하게 됩니다. 이 과정에서 시뮬레이션에 필요한 VPC와 공격자/타겟의 역할을 하는 EC2 서버 2대, 데이터 유출 대상 S3 버킷, 통지용 SNS 토픽 등이 마련됩니다. 그 다음, 공격 탐지에 필요한 GuardDuty, Macie, SecurityHub, Inspector, Detective 서비스들을 활성화 시킵니다. 마지막으로 공격자 역할의 EC2에 배정되었던 EIP(Elastic IP)를 GuardDuty에 악성 IP로 등록합니다.

실습 최소 48시간 전에 Amazon GuardDuty를 활성화 하지 않았다면, Amazon Detective를 활성화 시킬 수 없습니다. 이로 인해 일부 실습 과정(3-2 Detective를 통한 RCA(Root Cause Analysis))의 진행이 안되지만 나머지 과정은 진행할 수 있습니다.

2. 공격 시뮬레이션
Session Manager를 이용하여, 공격자 역할의 EC2에서 타겟 EC2를 대상으로 일련의 공격 행위들을 시뮬레이션 하게 됩니다. 공격의 목표는 타겟 EC2를 해킹해서 해당 EC2에 부여된 IAM 역할을 획득하고, 이를 이용하여 S3에 있는 중요 정보를 유출하고 해당 어카운트 전체를 탈취하는 것입니다. 이러한 과정들은 몇 개의 스크립트 및 AWS 명령어를 직접 실행하는 방식으로 시뮬레이션하게 됩니다.

3. 공격 탐지 및 조사
앞 단계에서 발생한 행위들은 GuardDuty, Macie, Inspector 등에 의해 탐지 되고, 설정된 이메일로 통지됩니다. 실습자는 GuardDuty, Macie, SecurityHub를 통해 탐지된 내역들을 확인한 뒤, 추가적으로 Detective를 통해, 정확한 원인분석(Root Cause Analysis)을 수행하게 됩니다.

이벤트 엔진을 통해 실습을 진행하는 경우, 이벤트 엔진에서 Amazon Detective를 지원하지 않는 관계로 일부 실습 과정(3-2 Detective를 통한 RCA(Root Cause Analysis))은 진행이 안됩니다.

4. 대응 및 경감 조치
시뮬레이션 과정에서 이미 몇가지 상황들에 대해서는 CloudWatch Event와 Lambda를 통해 자동으로 대응 조치들이 수행됩니다. 실습자는 가이드에 따라 이 과정의 결과들을 확인하게 됩니다. 또한, 추가적으로 수작업 조치 및 복원해야 될 부분들을 가이드를 따라가면서 진행합니다.

5. Security Incident Response Playbook 작성
온라인 실습과정을 마치고, 실습 시나리오를 토대로 Security Incident Response Playbook을 각자 작성해 보고, 공유 및 토론하는 시간을 갖습니다.

---
출처:https://catalog.us-east-1.prod.workshops.aws/workshops/dfa6471a-32be-4b06-96c3-ea5e650fefda/ko-KR
