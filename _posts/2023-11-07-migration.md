---
layout: single
title:  "[IT 기본] 마이그레이션 시 주의헤야할 점"
tags:
  - basic
  - Migration

categories:
  - basic
    
toc: true
toc_sticky: true
---

### 이관 우선 순위 파악
 
- 개발 or 운영 서버인지
  - 개발 서버만 우선적으로 이관
- Unix OS (HP-UX, HPE-OpenVMS, Oracle Solaris SPARC, AIX 등) 골라내어 -> U2L 대상인지 확인
- Appliance 장비인지 -> Retain 시 대안을 제시해야함
- VDI 환경을 위한 서버 -> 제외!
- 7R 패턴에 따른 분석 (Application 재설치를 통한 Clean install의 경우)
  - 내부 재개발 계획 확인
- I/F의 복잡성 확인 (내부, 외부 연동 서비스) 등등
- Decision Tree 작성을 통해 우선 순위 정의함

  
### 이관 시 주의 사항 파악

- 해당 서비스와 연동 되어 있는 부분은 반드시 파악되어야 합니다
- Data Migration 시 데이터 전송 테스트를 통해 전송시간을 필히 측정하여 Migration 예상 시간을 예측해야합니다.
- DNS IP 변경 시 변경 매뉴얼 작성하고 각 서비스 담당자에게 사전 배포해야합니다.
- Oracle DB에 대한 라이선스 확인을 통해 라이선스 지원 여부와 EC2 Include 및 RDS or BYOL로 갈지 미리 확인해야 합니다.
- 서버 OS에 백신설치 시 Domain 설치 방식인지, IP 설치방식인지 반드시 확인해야합니다.
- 데이터 파일이 크고 대량일 경우 AWS의 Disk IOPS 항목을 확인해야합니다.
- 테스트 시 접속 속도 및 기능속도를 비교하기 위해 기존 서비스에 대한 접속속도 및 기능속도를 미리 측정해 놓아야 합니다.

### 이관 시 주의 사항 파악 - Oracle
- 기존 Oracle 라이선스에 대한 서비스 종료 및 유지보수 계약 되어 있는지 확인 필요
- 기존 Oracle 라이선스가 Cloud로 이전 시 Upgrade가 필요한지 확인 필요
- 라이선스 비용 이슈로 Oracle EE -> SE로 낮추는 방안 있는지 확인 필요
- 기존 Oracle SE 버전은 이전 시 AWS RDS BYOL과 Include license 모두 적용 가능하므로 가격 비교 필요
*변경시 기존 CPU, Memory 사용률 확인하고 낮을 경우 Instance 사양을 낮추어 비용을 Save함
*SE의 경우 Oracle RDS Include license가 BYOL 보다 저렴함 (BYOL License의 List price 가격의 25% 할인율 적용시)
*BYOL : Standard Edition Two + Enterprise Edition / Include license : Standard Edition Two
- EE가 Partition Table을 사용할 경우 SE와 Migration 하기 어려워 분리가 필요
**Oracle 계정(User) 분리할 경우 API 형태로 제공 고려해야함
**시스템 중단 최소화 하기 위해선 CDC 솔루션을 사용해야함(가격 비쌈)
: CDC 솔루션을 사용할 경우 기존 Oracle 버전이 지원되는지 확인해야함


### 이관 시 주의 사항 파악 - Oracle License

- Oracle Database 10g Express Edition
CPU 개수와 상관 없으나, 단일 사용자만 사용 가능
사용자 데이터 저장 용량이 4GB까지
1GB까지의 메모리를 사용
무료 사용 가능 

- Oracle Personal Edition:
CPU 개수와 상관 없으나, 단일 사용자만 사용 가능
모든 Oracle DB 제품군과 호환이 가능
서버급 이하인 Desktop 장비일 경우 Personal 라이선스 구입 가능

- Oracle Standard Edition One(SE1):
최대 2CPU까지 확장 가능한 서버에 설치 가능
Entry Level 서버를 위한 2 Processor 미만, 최소 User는 5명
RAC 지원 되지 않음

- Oracle Standard Edition(SE):
최대 4CPU까지 확장 가능한 서버에 설치 가능
클러스터링을 지원하는 4 Processor 미만의 서버를 위한 것임
최소 사용자는 5명이며, RAC가 포함되어 있음
최대 4 Processor(CPU)나 8 Core까지의 서버에서만 사용 가능함 

- Oracle Enterprise Edition :
4 CPU 이상 확장 가능한 서버에 설치 가능
최고의 성능과 확장성, OLTP 상의 안정성, 의사결정지원 기능
Processor 수는 제한이 없음
최소 사용자는 25명이며, RAC는 옵션임.
주로 대기업 등의 강력하고 안정적인 DB가 요구 되어지는 경우 사용함

### 이관 시 주의 사항 파악 - CDC 작동 방식
1. 초기 IDC Full Data 백업
2. 백업 파일 전송 (S3)
3. Cloud RDS에 Full Data 적재
4. (변경분 적재) '1' 이 수행 된 이후부터 변경분 데이터 로그를 Target DB에 적용
5. (시스템 중단 및 재기동) 양쪽 DB가 동기화 되었을 때 IDC 서비스 중단
6. Cloud에서 변경된 서비스로 재기동 
