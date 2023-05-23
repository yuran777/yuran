--
title: cloud security csp별 제품
---

# 벤더별 보안 서비스 정리

## AWS

| 영역 | 보안 기능 | 설명 |
| --- | --- | --- |
| Authorization| lAM | Identity and Access Management, 사용자 계정 및 권한 관리 |
|  | AWS RAM | Resource Access Management, 멀티 계정 간 리소스 공유 기능 |
|  | AWS Organization | 계정에 대한 정책 관리, 계정 그룹 생성 및 거버넌스 중앙 통제 |
| Authentication | Amazon Cognito | 웹 및 모바일 앱에 대한 인증, 권한 부여 및 사용자 관리 제공 ID/패스워드 인증 및 facebook, google과 같은 다른 서비스 계정을 통한 로그인 지원 |
|  | AWS SSO | Single Sign-On, 여러 AWS 계정과 비즈니스 애플리케이션에 대한 SSO액세스를 중앙에서 관리 |
|  | Amazon Directory Service | Microsoft AD를 이용하여 사용자, 그룹 및 디바이스 관리 |
| Protected Stores | AWS CloudHSM | Hardware security modules AWS 클라우드 자체 암호화 키를 손쉽게 생성 및 사용할 수 있도록 지원하는 클라우드 기반 하드웨어 보안 모듈 |
|  | AWS KMS | Key Management Service, 통합된 AWS 서비스 및 자체 애플리케이션에 걸쳐 일괄로 키를 관리하고 정책을 정의하는 기능 제공 |
|  | AWS Secret Manager | 코드 배포 없이 보안 정보(데이터베이스 자격증명, AP 키 등)를 손쉽게 교체, 관리, 검색하는 기능 제공 |
|  | AWS Certificate Manager | AWS 서비스 및 연결된 내부 리소스에 사용할 공인 및 사설 SSL/TLS 인증서를 손쉽게 프로비저닝, 관리 및 배포하는 서비스 제공 |
| Visibility| AWS Artifact | AWS 보안 및 규정 준수 보고서에 온디맨드 방식으로 액세스할 수 있도록 제공 |
|  | AWS Security HUB | AWS 계정 전반에 걸쳐 우선순위가 높은 보안 경고 및 규정 준수 상태를 종합적으로 확인할 수 있게 제공 (대시보드, 레포트) |
| Enforcement| Amazon Guard Duty | 악의적인 활동 또는 무단 동작을 지속적으로 모니터링하는 위협 탐지 서비스 |
|  | Amazon Inspector | AWS에 배포된 애플리케이션의 보안 및 규정 준수를 개선하는 자동, 보안평가 서비스. EC2인스턴스의 취약성 평가 및 액세스 적합성 확인 |
|  | Amazon Macie | 기계학습을 사용하여 AWS에 저장된 민감한 데이터(개인식별정보, 지적재산 등)를 자동으로 검색, 분류 및 보호하는 보안서비스 |
|  | AWS WAF | 웹 애플리케이션 방화벽 서비스 |
|  | AWS Shield | DDos 공격으로부터 서비스 보호 |


## AZURE


| 영역 | 보안 기능 | 설명 |
| --- | --- | --- |
| 일반 보안 | Azure Security Center | 네트워크를 강화하고 서비스 보안을 유지하며 보안 상태를 제어하는데 필요한 도구 제공, Agent를 이용하여 클라우드와 온프레미스의 가상서버 보호 |
|  | Azure Key Vault | 보안 비밀 저장소 |
|  | Azure Monitor 로그 | 다른 모니터링 데이터와 함께 활동 로그를 수집하여 전체 리소스 집합에 대한 심층 분석을 제공하는 로그 데이터 플랫폼 |
|  | Azure Dev/Test Labs | 재사용 가능한 템플릿과 아티팩트를 이용해서 개발과 테스트에 필요한 환경을 빠르게 구성하는 서비스 |
| Storage 보안 | Azure Storage 서비스 암호화 | Azure Storage 데이터 자동 암호화 기능 |
|  | Azure 클라이언트 쪽 암호화 | 암호화 기술을 이용하여 클라이언트 프로그램에서 데이터 업로드 시 암호화, 다운로드 시 복호화하는 서비스, Azure Key Vault와 통합 지원 |
|  | Azure Storage 공유 액세스 서명 | Azure Storage 리소스에 대한 제한된 액세스 서명 서비스 |
|  | Azure Storage 계정 키 | 키를 이용한 스토리지 액세스 제어 방법 |
|  | SMB 3.0 암호화를 사용한 Azure 파일 공유 | SMB 파일 공유 프로토콜에 네트워크 암호화 사용 기술 |
|  | Azure Storage 분석 | 스토리지 계정 사용 로깅 및 메트릭 생성 기 |
| DB 보안 | Azure SQL 방화벽 | 데이터베이스에 대한 네트워크 액세스 제어 |
|  | Azure SQL 연결 암호화 | 방화벽 규칙, 사용자 ID 인증, 특정 작업 및 데이터에 대한 사용자를 제한하는 권한 부여 메커니즘 사용한 액세스 제어 |
|  | Azure SQL 항상 암호화 | 신용카드 번호 또는 주민등록번호와 같은 중요 데이터 암호화 데이터베이스 스토리지 암호화 |
|  | Azure SQL 투명한 데이터 암호화 | 데이터베이스 스토리지 암호화 |
|  | Azure SQL Database | 데이터베이스 이벤트 및 계정의 감사로그 기록 |
| |D 및 액세스 관리 | Azure 역할기반 액세스 | 리소스 기반 액세스 제어 |
|  | Azure Active Directory | 클라우드 기반 디렉터리 및 여러 ID 관리 서비스를 지원하는 인증 리포지토리 |
|  | Azure Active Directory Domain Services | 클라우드 기반 Active Directory Domain 서비스 |
|  | Azure Multi -Factor Authentication | 복합 인증 서비스 |
| 백업 및 재해복구 | Azure Backup | 데이터 백업 및 복원 서비스 |
|  | Azure Site Recovery | 기본 사이트를 보조 위치로 복제하여 오류 시 서비스 복구가 가능하도록 하는 온라인 서비스 |
| 네트워킹 | 네트워크 보안그룹 | 네트워크 기반 액세스 제어 |
|  | Azure VPN Gateway | VPN 제공을 위한 네트워크 디바이스 |
|  | WAF | Web Application Firewall, 웹 취약점 방어 |
|  | Azure Traffic Manager | 전역 DNS 부하 분산 장치 |
|  | Azure DDOS Protection | DDoS 공격 방어 |
