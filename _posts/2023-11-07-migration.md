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

### 이관 전 확인해야하는 부분
1. 서비스 특징
   - 서비스별 특징에 맞게 이관 시 유사한 서비스와 함께 이관하기 위해 필요하다.
서비스 중에서는 사용자가 접근하여 사용하는 서비스도 있는데, 일반 사용자가 아
닌 시스템 연계를 위해 구성 된 서비스도 많다. 이경우에는 연계된 서버도 함께 파
악하여 이관 시 연동 접속 정보를 (내부 ip로 연동 되어 있는지, 내부 ip 로 연동 되
었다면 클라우드 이관 시 해당 ip를 사용해야하는지, 신규 도메인 구성해서 연동 해
도 되는지 ) 파악하여 조건에 맞게 수정 해주는 등의 작업이 필요해 질문함.

2. 서비스 접속 정보
- 내부 사용자와 외부 사용자 접속 방식 (아이피/ 포트)이 다른 경우 반드시 확인 필
요하다. (이런식으로- http://192.168.10.191:8080/login) 클라우드 이관 시 1. 오
픈 해야하는 포트 번호 파악을 위해 2. 망을 나누기 위해서 필
요하다고 볼 수 있다.

3.피크시간:  부하가 증가하는 경우가 많다면 오토스케일링에 대한 고려도 필요하다. 오토스케일
링에 대해 고민할때에는 솔루션 라이선스 제약이 있는지 사전 확인이 꼭 필요하
다.

4. 서비스 URL 별도의 접속 도메인이 없고 IP를 통해 접근했었다면 도메인 설정 필요하기에 질문 필요
5. 이중화 여부 : 기존에 이중화 되어 있지 않은 서비스를 클라우드를 이관 할 때 이중화 하려고한다
면, 배치 작업 이 있다면 따로 배치 서버를 두는 등의 리팩토링 필요할 수도 있음.
6. WEB/WAS : WAS → EJB*를 사용하는 경우 Apache tomcat 사용 불가, JBOSS/ WebLogic /
Websphere 중 하나 골라야 한는데, 비용이 저렴한 JBOSS 우선 적용 여부 검토하는
게 편함
*EJB:
참고- JSP:
https://rainbow97.tistory.com/entry/JAVA-EJBEnterprise-JavaBeans

7. 첨부파일 파일 스토리지- S3 / EFS (FSx)
S3: 저렴 ( efs 대비 11배 저렴) , s3 라이프 사이클 정책을 적용할 수 있음 하지만
s3에 저장하고 로드하기 위해 데이터와 코드 수정 필요
EFS/FSx: 비쌈 OS에 마운트 되는 형태로 사용할 수 있어 기존 파일스토리지 사용방
법과 다르지 않고 코드 변경도 필요 없음

8. 이관 및 테스트 가능 시간을 확인하기 위해
aws 에서 배치 구성하는 방법으로는 1. aws 배치 사용 (https://aws.amazon.com/ko
/batch/) 2. ec2 서버에 스프링 스케줄러나 스프링 쿼츠 등을 이용해 배치 잡 실행
및 관리하기 3. 람다 사용 ( 실행 시간이 900초 미만인 경우)

9. 개발 언어
만약 Oracle 19c RDS를 사용하고자 한다면, OJDBC*8 혹은 10 필요. OJDBC8은
JDK 8/9/11 OJDBC10은 JDK 10/11 사용이 필요. 그렇기 때문에 JDK 6이나 7 버전
혹은 그 이하 버전을 사용하는 어플리케이션은 필수적으로 버전 업그레이드 필요.!
*JDBC: Java + DB + Connectivity 데이터베이스에 접속할 수 있도록 하는 자바 API
이다. JDBC는 데이터베이스에서 자료를 쿼리하거나 업데이트하는 방법을 제공한
다.
*OJDBC: 오라클 전용 JDBC 라이브러리
https://learn.microsoft.com/ko-kr/java/openjdk/reasons-to-move-to-java-11

10. 프레임워크
스프링 프레임워크 3.x.x 은 공식적으로 JDK 8을 지원하지 않음. JDK 8과의 호환성
을 위해 스프링 프레임워크 4.3.x (EOL 2020/12/31) 이상으로 업그레이드 필요.
참고) Spring FrameWork 3.2.x → JDK 5
Spring FrameWork 4.5.x → JDK 6/7/8 - EOL 2020/12/31
https://spring.io/projects/spring-framework#support
iBatis/ MyBatis 사용하는 경우
iBatis 사용시 MyBatis 프레임워크로 변경 필요 (2010년에 iBatis가 EoS 됨)
* Mybatis는 Java에서 관계형 데이터베이스를 사용하는 개발을 더욱 수월하
게 해주는 프레임워크로, 기존의 JDBC를 사용한 코드를 별도의 XML 파일로
분리하여 자바 코드상의 SQL을 모두 제거하고 관리를 편하게 해준다는 장점
을 갖는다.
