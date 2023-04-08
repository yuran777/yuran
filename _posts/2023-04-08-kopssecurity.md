---
layout: single
title:  "[PKOS STUDY #5] - 쿠버네티스 보안"
---

![](https://velog.velcdn.com/images/yuran3391/post/c1548ed2-6563-49f5-b738-0be5bb4d7feb/image.png)

> **이정훈님의 ‘24단계 실습으로 정복하는 쿠버네티스’ 기반으로 작성된 포스팅 입니다.**
도서 링크 : http://www.yes24.com/Product/Goods/115187666
![](https://velog.velcdn.com/images/yuran3391/post/429d18dd-2f0f-40c7-a1f8-789f8171ea5b/image.png)

## 실습환경 배포
- kops 인스턴스 t3.small & 노드 c5a.2xlarge (AMD vCPU 8, Memory 16GiB) 배포 : 이번주 실습에서 성능을 요구하는 파드를 사용함

```
# YAML 파일 다운로드
curl -O https://s3.ap-northeast-2.amazonaws.com/cloudformation.cloudneta.net/K8S/kops-oneclick-f1.yaml

# CloudFormation 스택 배포 : 노드 인스턴스 타입 변경 - MasterNodeInstanceType=t3.medium WorkerNodeInstanceType=c5d.large
aws cloudformation deploy --template-file kops-oneclick-f1.yaml --stack-name mykops --parameter-overrides KeyName=kp-gasida SgIngressSshCidr=$(curl -s ipinfo.io/ip)/32  MyIamUserAccessKeyID=AKIA5... MyIamUserSecretAccessKey='CVNa2...' ClusterBaseName='gasida.link' S3StateStore='gasida-k8s-s3' MasterNodeInstanceType=t3.xlarge WorkerNodeInstanceType=t3.xlarge --region ap-northeast-2

# CloudFormation 스택 배포 완료 후 kOps EC2 IP 출력
aws cloudformation describe-stacks --stack-name mykops --query 'Stacks[*].Outputs[0].OutputValue' --output text

# 13분 후 작업 SSH 접속
ssh -i ~/.ssh/kp-gasida.pem ec2-user@$(aws cloudformation describe-stacks --stack-name mykops --query 'Stacks[*].Outputs[0].OutputValue' --output text)

# EC2 instance profiles 에 IAM Policy 추가(attach) : 처음 입력 시 적용이 잘 안될 경우 다시 한번 더 입력 하자! - IAM Role에서 새로고침 먼저 확인!
aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --role-name masters.$KOPS_CLUSTER_NAME
aws iam attach-role-policy --policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy --role-name nodes.$KOPS_CLUSTER_NAME
```

워커 노드 1대 EC2 메타데이터(IMDSv2) 보안 제거 ⇒ 적용 및 재생성 5분 시간 소요

```
#
kops edit ig nodes-ap-northeast-2a
---
# 아래 3줄 제거
spec:
  instanceMetadata:
    httpPutResponseHopLimit: 1
    httpTokens: required
---

# 업데이트 적용 : 노드1대 롤링업데이트
kops update cluster --yes && echo && sleep 3 && kops rolling-update cluster --yes
```

### EC2 IAM Role & 메타데이터

- 파드에서 EC2 메타데이터 사용 가능 확인

```
# netshoot-pod 생성
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netshoot-pod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: netshoot-pod
  template:
    metadata:
      labels:
        app: netshoot-pod
    spec:
      containers:
      - name: netshoot-pod
        image: nicolaka/netshoot
        command: ["tail"]
        args: ["-f", "/dev/null"]
      terminationGracePeriodSeconds: 0
EOF

# 파드 이름 변수 지정
PODNAME1=$(kubectl get pod -l app=netshoot-pod -o jsonpath={.items[0].metadata.name})
PODNAME2=$(kubectl get pod -l app=netshoot-pod -o jsonpath={.items[1].metadata.name})

# EC2 메타데이터 정보 확인
kubectl exec -it $PODNAME1 -- curl 169.254.169.254 ;echo
kubectl exec -it $PODNAME2 -- curl 169.254.169.254 ;echo

# 파드1에서 EC2 메타데이터 정보 확인
kubectl exec -it $PODNAME1 -- curl 169.254.169.254/latest ;echo
kubectl exec -it $PODNAME1 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/ ;echo
kubectl exec -it $PODNAME1 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/nodes.$KOPS_CLUSTER_NAME | jq

# 파드2에서 EC2 메타데이터 정보 확인
kubectl exec -it $PODNAME2 -- curl 169.254.169.254/latest ;echo
kubectl exec -it $PODNAME2 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/ ;echo
kubectl exec -it $PODNAME2 -- curl 169.254.169.254/latest/meta-data/iam/security-credentials/nodes.$KOPS_CLUSTER_NAME | jq

# 메타데이터 메모해두기
{
  "Code": "Success",
  "LastUpdated": "2023-04-08T05:50:29Z",
  "Type": "AWS-HMAC",
  "AccessKeyId": "ASIA6EARJYBJ3JXA2ZG2",
  "SecretAccessKey": "vZc4jsUv6ypUOG0mAbFerrNFobFw8Xd8GoKbdTlL",
  "Token": "IQoJb3JpZ2luX2VjEG4aDmFwLW5vcnRoZWFzdC0yIkYwRAIgDwgcHiD14mDZT56micaxCIANdKLVavbc5Ea7IpPiUVwCIDbSHSHn+XpWxk0EZpfD2Nyu13cBcCwWLQVnJsJAq7j9KssFCFcQAhoMOTcwNjk4ODk5NTM5Igwa7mfQh2VHO6iKrBAqqAWGxP216vqKjCqpQIcsZsrbkb7h6PaC8vjFlZLy2DU0SJAr7Ep2xNI2hcvTMFlScCMUU3FhTvKXTot6Z1E41Kl04dUEYuWF5tL+GhBrnlhKK6QSMAtfMbw+mjVHcWozEbus+ZsVZm/KfTZqapWEx97FJiPa0Z9LQNZAsFNC7TxyyEsBDBKlnrIdWpnoIBdyrZaj/GAHxqpMnGpQDVnKgHSObSUAW4aYe9kZ6XYTIh5VM4dsIsExl0zF5sw5o/zD7Un1061DuI+sdnwzu37r0NNxgGuNumo3AjG8ULUNZVw8Z60jOGMgqlrO6oBuUdyloxvKOTCBIv2m8wLUCo+KLNA4W9pPNMkXR1X4y+k3vT33jWnfkHw2GJziZO0YzA0OTl6pS7Zp0mx1kNqyAbqISC4xroyIcZiD9xpeitp30Wf5Lh5PcqHeHN45ADA+JaBcIPp4P4gSU5bFOQCh5zZoXVwR+5Bwv7uslaY2eTLIRCOQ2ACxG+RPFeNnUVAPSAIJu7gR/B7k2rgvA2Uhwkvg8Lf0+P9Wfur+shQD2fVcVvP0OpS2R02K5LjAQaqzEakeVWOzrwekcyJJSTaYLLqASQ75c8rMIbp+UW5uyaQ15gEMOSnn9/VNw5qTD07R4i+DrTcXETOePvseAXspmnYq0NTwa+0NFDFyLIvieZAd1gzMNhkjIyd7gbipAPosDTqr/6Zx5gdh97eDYxdv29sIAi5zf/eVSEt2MEnZpfYnlKLh3elh/qprKkrWIk0YJ+T3zCz5a/Uo2OCaEj9ph52cxxybdO0EYJ6H4cdTTAi/jhDz9VUv7h/1DnoBdkatg7Yed+pyZPhUKv8eolKhWUR152ghKjTpndaVG+8+LYA1oeubdjlrulaRRKPPkep3xUbY9KOmwNdstV89MjCSgcShBjqyAZuAv5bT0lCkbKBCXrP2ygliwu8Em5pDprArfG2qYnI/0uSa0AZP2EjWIvNHHOcmQ5sV1KNMaSL82jqQOgrJB/BK31EWx98VnhNQ6/k5pEJVtvrFznrO1kK49b7ji5+sLHVEqP43VPlUZOnTTf8WbiqQ5yNPEO2mjfr0Y3XIsa4qWihEP9AojFbu8kY5HHR0Xq6pGCx0xAfPj4ZXiIJfLeeKCEYIYP87zVN10FaosvXmkuY=",
  "Expiration": "2023-04-08T12:25:10Z"
}
```


## kubescape 
### 소개

Kubescape는 보안 권고 사항 기반 현재 쿠버네티스 클러스터(YAML, Helm chart)의 취약점을 점검하는 도구입니다.



![img](https://www.armosec.io/wp-content/uploads/2023/01/Group-1000005089.svg)

Kubescape에 대한 소개를 간략하게 번역하여 정리해보았습니다.

- Kubescape는 IDE, CI/CD 파이프라인 및 클러스터를 위한 오픈 소스 Kubernetes 보안 플랫폼입니다. 위험 분석(risk analysis), 보안(security), 규정 준수(compliance), 잘못된 구성(misconfiguration) 스캔이 포함되어 있어 Kubernetes 사용자와 관리자의 귀중한 시간, 노력, 리소스를 절약할 수 있습니다.
- Kubescape는 클러스터, YAML 파일 및 헬름 차트를 스캔합니다. 여러 프레임워크(including [NSA-CISA](https://www.armosec.io/blog/kubernetes-hardening-guidance-summary-by-armo/?utm_source=github&utm_medium=repository), [MITRE ATT&CK®](https://www.microsoft.com/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/) and the [CIS Benchmark](https://www.armosec.io/blog/cis-kubernetes-benchmark-framework-scanning-tools-comparison/?utm_source=github&utm_medium=repository))에 따라 잘못된 구성을 감지합니다.
- Kubescape는 [ARMO](https://www.armosec.io/?utm_source=github&utm_medium=repository)에서 만들었으며 CNCF(Cloud Native Computing Foundation)의 샌드박스 프로젝트입니다.



### 설치 및 사용

설치 및 사전 구성은 다음과 같은 절차를 따릅니다.

1) 설치 스크립트를 다운받고 기본적으로 제공하는 보안 프레임워크 및 통제 정책을 확인합니다.
2) 이후 클러스터 스캔 명령을 통해 취약점을 분석합니다.



- 바이너리 설치

```bash
# 설치
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
```

- 아티팩트 다운로드 및 확인

```bash
# Download all artifacts and save them in the default path (~/.kubescape)
kubescape download artifacts
[success] Downloaded. artifact: framework; name: ArmoBest; path: /Users/hyungwook/.kubescape/armobest.json
[success] Downloaded. artifact: framework; name: cis-v1.23-t1.0.1; path: /Users/hyungwook/.kubescape/cis-v1.23-t1.0.1.json
[success] Downloaded. artifact: framework; name: cis-eks-t1.2.0; path: /Users/hyungwook/.kubescape/cis-eks-t1.2.0.json
[success] Downloaded. artifact: framework; name: NSA; path: /Users/hyungwook/.kubescape/nsa.json
[success] Downloaded. artifact: framework; name: MITRE; path: /Users/hyungwook/.kubescape/mitre.json
[success] Downloaded. artifact: framework; name: DevOpsBest; path: /Users/hyungwook/.kubescape/devopsbest.json
[success] Downloaded. artifact: framework; name: AllControls; path: /Users/hyungwook/.kubescape/allcontrols.json
[success] Downloaded. attack tracks: attack-tracks; path: /Users/hyungwook/.kubescape/attack-tracks.json
[success] Downloaded. artifact: controls-inputs; path: /Users/hyungwook/.kubescape/controls-inputs.json
[success] Downloaded. artifact: exceptions; path: /Users/hyungwook/.kubescape/exceptions.json
```

- 아티팩트 확인

```bash
tree ~/.kubescape
/Users/hyungwook/.kubescape
├── allcontrols.json
├── armobest.json
├── attack-tracks.json
├── bin
│   └── kubescape
├── cis-eks-t1.2.0.json
├── cis-v1.23-t1.0.1.json
├── controls-inputs.json
├── devopsbest.json
├── exceptions.json
├── mitre.json
└── nsa.json

2 directories, 11 files

cat ~/.kubescape/attack-tracks.json | jq
```



```bash
# 모니터링
watch kubectl get pod -A

# 클러스터 스캔
# Deploy Kubescape host-sensor daemonset in the scanned cluster. Deleting it right after we collecting the data. 
# Required to collect valuable data from cluster nodes for certain controls. 
# Yaml file: https://github.com/kubescape/kubescape/blob/master/core/pkg/hostsensorutils/hostsensor.yaml
kubescape scan --help
kubescape scan --enable-host-scan --verbose
+----------+-------------------------------------------------------+------------------+---------------+---------------------+
| SEVERITY |                     CONTROL NAME                      | FAILED RESOURCES | ALL RESOURCES |    % RISK-SCORE     |
+----------+-------------------------------------------------------+------------------+---------------+---------------------+
| Critical | Disable anonymous access to Kubelet service           |        0         |       0       |  Action Required *  |
| Critical | Enforce Kubelet client TLS authentication             |        0         |       0       |  Action Required *  |
| High     | Forbidden Container Registries                        |        0         |      21       | Action Required *** |
| High     | Resources memory limit and request                    |        0         |      21       | Action Required *** |
...
+----------+-------------------------------------------------------+------------------+---------------+---------------------+
|          |                   RESOURCE SUMMARY                    |        47        |      204      |        8.78%        |
+----------+-------------------------------------------------------+------------------+---------------+---------------------+
FRAMEWORKS: AllControls (risk: 9.17), NSA (risk: 11.82), MITRE (risk: 6.94)
```



- 결과값 이미지

![img](https://raw.githubusercontent.com/hyungwook0221/img/main/uPic/FsSzzz.jpg)



## polaris
- `소개 및 설치` : 오픈소스 보안 점검 도구, Polaris is an open source policy engine for **Kubernetes** that **validates** and **remediates** resource **configuration** - [helm](https://artifacthub.io/packages/helm/fairwinds-stable/polaris)
    - It includes 30+ built in configuration policies, as well as the ability to build custom policies with JSON Schema.
    - When run on the command line or as a mutating **webhook**, Polaris can automatically remediate issues based on policy criteria.



![](https://velog.velcdn.com/images/yuran3391/post/9babd22d-6279-4c58-abdd-f815322c5bd0/image.png)


- **Score** : 모범 사례 대비 전체 클러스터 구성 내역 점수, 권고 사항 만족이 많을 수록 점수 상승
- **Passing/Warning/Dangerous** Checks : 위험 등급 분류, 위험 단계 취약점은 조치를 권고
- **측정 범위** : Efficiency, Reliability, Security
    - Security 보안성 : 보안 관련 구성 확인 - [링크](https://polaris.docs.fairwinds.com/checks/security/)
    - Efficiency 효율성 : CPU,Memory 리소스 사용 관련 - [링크](https://polaris.docs.fairwinds.com/checks/efficiency/)
    - Reliability 신뢰성 : 안정성 구성 여부 확인 - [링크](https://polaris.docs.fairwinds.com/checks/reliability/)
- 검사 항목 상세 - [링크](https://github.com/FairwindsOps/polaris/tree/master/checks)

보안 취약점 점검
polaris 웹 접속 → 중간 Filter by Namespace 에서 default 네임스페이스 클릭 후 Apply
![](https://velog.velcdn.com/images/yuran3391/post/317aee3e-23b5-4d5d-9696-fe5cf6750d85/image.png)

 Namespaces : default 아래 netshoot-pod 부분 ⇒ 아래 항목의 ? 클릭으로 상세 정보 확인
![](https://velog.velcdn.com/images/yuran3391/post/70d79e79-ab28-48dc-a1da-71e675384c60/image.png)



- image tag 조치

```
# 기존 netshoot-pod 삭제
kubectl delete deploy netshoot-pod

# netshoot-pod 1대 생성
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netshoot-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: netshoot-pod
  template:
    metadata:
      labels:
        app: netshoot-pod
    spec:
      containers:
      - name: netshoot-pod
        image: nicolaka/netshoot:v0.9
        command: ["tail"]
        args: ["-f", "/dev/null"]
      terminationGracePeriodSeconds: 0
EOF
```

  대시보드 새로고침 후 확인
  
  ![](https://velog.velcdn.com/images/yuran3391/post/9547e194-ba41-498c-994a-add5b64575c1/image.png)

```
kubectl scale deployment netshoot-pod --replicas 2
```
![](https://velog.velcdn.com/images/yuran3391/post/65f490fa-e4a7-484e-b81d-32d2d903d414/image.png)

**보안 모범 사례 적용 후 재점검**
netshoot-pod 에 보안 모범 사례 적용

```
# 삭제
kubectl delete deploy netshoot-pod

# netshoot-pod 생성
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netshoot-pod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: netshoot-pod
  template:
    metadata:
      labels:
        app: netshoot-pod
    spec:
      containers:
      - name: netshoot-pod
        image: nicolaka/netshoot:v0.9
        command: ["tail"]
        args: ["-f", "/dev/null"]
        imagePullPolicy: Always
        resources:
          limits:
            cpu: 150m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 128Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          privileged: false
          readOnlyRootFilesystem: true
          #runAsNonRoot: true
      terminationGracePeriodSeconds: 0
EOF

```

![](https://velog.velcdn.com/images/yuran3391/post/28837e49-88c9-401a-b2c0-05b0e4cb841b/image.png)

### K8S 인증/인가 & RBAC



