---
titl: CT Basic - Network 
tags: security, aws
categories: aws, security, basic

toc: true
toc_sticky: true

---
## AWS Network Segmentation

### VPC Segmentation

A. VPC 분리 기준: 
- 외부에서 바라 보는 라우팅 기준으로 VCP를 분리해야함(외부 접근 관점)
- 네트워크 라우팅 통제가 필요한 단위로 구분 (사내 라우팅 전파 정책 단위)
ex) 
1. PRD/DEV/STG VPC 구분 (분리 정책: PRD/ DEV 간 서로 접근 불가)
2. A 계열사/ B 계열사 VPC
3. A 비즈니스용/ B 비즈니스용 VPC (일반적으로 추가되는 기본 정책: 모든 VPC내 인터넷은 직접 인터넷 접근 불가 -WAF/FW 요건  )
4. Dedicated DX 회선 사용 VPC/ Shared DX 회선 사용 VPC
---
B. 가상으로 나눠 본다고 하면
Workload 별 VPC(DEV/STG,PROD를 prefix /16으로)
전사적 관점의 인프라용 VPC인 Shared VPC 하나 (얘는 prefix /20)
Egress VPC (내부-> 인터넷 접근 차단용, DEV/STG, PRD 각각 /20 )
Security VPC (보안 시스템 전용)

C. VPC 분리 세부 내용
- VPC 생성 시 IP 대역 정의 필요 
- prefix 지원 범위 정의 필요 (최대 /16~ 최소 /28)
- RFC 1918에 정의된 사설 IP 대역 사용 권장 
 10.0.0.0/8
 172.16.0.0/12
 192.168.0.0/16
 - 향후 확장시 사용할 대역도 미리 고려해 놓는게 좋음

참고: https://docs.aws.amazon.com/vpc/latest/ipam/planning-examples-ipam.html 


### Subnet Segmentation
A. Workload Subnet 분리 기준 (내부에서 나가는 Routing)
- VPC 내부에서 외부로 접근하는 라우팅을 기준으로 분리 고려 필요
- VPC 내부에서 외부로 접근하는 관점에서 고려 필요
- ACL 설정 단위
- AZ 이중화 고려

ex 1) Public Subnet(인터넷 가능 - IGW 통해 접근) / Private Subnet(인터넷 불가 - Public Subnet에 있는 NAT 통해서 접근해야함)
/ DB Subnet (VPC 외부 통신 불가)


B. 가상으로 나눠 본다고 하면
워크로드 VPC
 퍼블릭 서브넷:WEB-Public, APP(WAS)-Public (LB용) , FW용 서브넷
 프라이빗 서브넷: TGW, DB, APP(WAS, Internal LB용), WEB, EP(AWS Endpoint 연결용)


### 보안성 있는 Ingress/Egress Traffic Flow 구현

A. 개요
North Egress 제어:  Egress전용 VPC & Firewall & Nat Gateway & GWLB 구성

EC2 -> TGW -> (GWLBe-> GWLB) -> FW -> NAT -> IGW -> Internet -> IGW ->  GWLBe -> GWLB -> FW -> TGW -> EC2

South Ingress 제어(web): 각 VPC 내 생성된 Resource별 Security Group을 활용한 Ingress 제어 및 HTTP/HTTPS WAF(AWS or 3rd-party) & ALB 결합 (웹트래픽 방화벽 미경유)

South Ingress 제어(non web): 각 VPC 내 생성된 Resource별 Security Group을 활용한 Ingress 제어 및 Firewall & NLB 결합 (Client IP 식별)

East-West 제어: VPC 간 통신시 VPC 내부에 생성된 Resource별 Security Group/NACL을 활용한 Ingress 제어

---
### security groups 

특징

- 허용할 Rule을 반드시 지정해야 통신 가능
- 명시적 Deny Rule 지정 불가
- 각각 inbound and outbound rules 지정 필요
- EC2 instance 지정 가능, 인스턴스 당 5개까지 assign 가능
- Security Groups IP addresses, subnets, prefix-list, 또는 다른 Security Group을 assign 가능
- Security Groups는 다른 인스턴스에 적용 가능 

보안 그룹을 위한 모범 사례
- 사용하지 않거나 연결되지 않은 보안 그룹 제거 사용하지 않거나 연결되지 않은 보안 그룹이 많으면 혼란이 발생하고 잘못된 구성이 발생합니다. 사용하지 않는 보안 그룹을 제거하십시오. (PCI.EC2.341)
- 승인된 역할에 대한 수정 제한 액세스 권한이 있는 AWS Identity and Access Management(IAM)42 역할만 보안 그룹을 수정할 수 있습니다. 보안 그룹을 변경할 권한이 있는 역할의 수를 제한하십시오. (PCI DSS 7.2.143)
- 보안 그룹의 생성 또는 삭제 모니터링이 모범 사례는 처음 두 가지와 함께 작동합니다. 보안 그룹의 생성, 수정 및 삭제 시도를 항상 모니터링해야 합니다. (CIS AWS 기초 3.1044)
- 아웃바운드 또는 송신 규칙을 무시하지 마십시오. 필요한 서브넷으로만 아웃바운드 액세스를 제한하십시오. 예를 들어 3계층 웹 애플리케이션에서 앱 계층은 인터넷에 대한 무제한 액세스를 허용하지 않아야 하므로 애플리케이션의 올바른 작동에 필요한 호스트 또는 서브넷에만 액세스할 수 있도록 보안 그룹을 구성합니다. (PCI DSS 1.3.445)
- 액세스할 수 있는 인그레스 또는 인바운드 포트 범위를 제한합니다. 보안 그룹에서 열려 있는 포트를 애플리케이션이 올바르게 작동하는 데 필요한 포트로만 제한합니다. 큰 포트 범위가 열려 있으면 취약성에 노출되거나 서비스에 대한 의도하지 않은 액세스가 발생할 수 있습니다. 이것은 고위험 응용 프로그램에서 특히 중요합니다. (CIS AWS 재단 4.146, 4.247) (PCI DSS 1.2.148, 1.3.249)

## AWS Network Connectivity

### Transit Gateway (TGW)
고려사항: 
- 비용: 

