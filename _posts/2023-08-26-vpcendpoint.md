---
layout: single
title:  "aws IAM"
tags:
  - aws
  - endpoint
  - vpcendpoint
  - interpafe endpoint
  - gateway endpoint
categories:
  - aws
  - security
toc: true
toc_sticky: true

---

# 1. AWS VPC endpoint 란?
vpc endpoint는 vpc 내부 Resource가 VPC 외부 AWS 서비스와 통신 할 때 외부 인터넷을 거치지 않고 AWS 백본 네트워크를 통해 서비스에 도달 할 수 있도록 지원하는 서비스 이며 Gateway endpoint와 Interface endpoint로 나눌 수 있습니다.

## 2. VPC endpoint 유형
**Gateway endpoint** 와 **Interface endpoint** 라는 두 가지 유형의 VPC 엔드포인트를 사용하여 Amazon S3에 접근할 수 있습니다. Gateway endpoint 는 AWS 네트워크를 통해 VPC에서 Amazon S3에 접근하기 위해 **route table 에 지정** 하는 Gateway 입니다. Interface endpoint 는 **Private IP 주소**를 사용하여 **VPC 피어링 또는 AWS Transit Gateway**를 사용하여 VPC 내, 온프레미스 또는 다른 AWS 리전의 VPC에서 Amazon S3로 요청을 라우팅함으로써 게이트웨이 엔드포인트의 기능 확장할 수 있습니다.

## 3. Gateway endpoint
- 게이트웨이 엔드포인트는 생성한 Region에서만 사용할 수 있습니다. 즉, us-east-1 Region에서 Amazon S3용 게이트웨이 엔드포인트를 생성하면 us-east-1 Region의 S3 버킷에만 게이트웨이 엔드포인트를 통해 액세스할 수 있습니다. 다른 Region의 S3 버킷에는 게이트웨이 엔드포인트를 통해 액세스할 수 없습니다.
- 게이트웨이 엔드포인트를 사용하려면 VPC에서 DNS 호스트 이름과 DNS 확인을 모두 활성화해야 합니다. 게이트웨이 엔드포인트는 AWS 서비스로 트래픽을 라우팅하기 위해 프라이빗 DNS 이름을 사용하기 때문입니다.
- 게이트웨이 엔드포인트의 기본 할당량은 Region당 20개입니다. 필요에 따라 더 높은 할당량을 요청할 수 있습니다.

## 4. Interface endpoint
- 인터페이스 엔드포인트는 각 가용 영역에서 최대 10 Gbps의 대역폭을 지원합니다. 인터페이스 엔드포인트는 최대 100 Gbps까지 자동으로 확장됩니다. 이는 대규모 트래픽 요구 사항을 충족하도록 엔드포인트를 확장할 필요가 없음을 의미합니다.
- VPC 엔드포인트는 VPC 내 리소스가 시작한 트래픽에만 응답합니다. 이는 VPC 엔드포인트가 악용될 위험을 줄입니다.
- 인터페이스 엔드포인트는 VPC 내 서비스와 통신하는 데 사용됩니다. 따라서 엔드포인트 네트워크 인터페이스와 VPC 내 서비스와 통신해야 하는 리소스 간의 통신을 허용하는 보안 그룹을 생성해야 합니다

---

## 실습
#### 1. s3 interface endpoint

<mxfile host="app.diagrams.net" modified="2023-08-26T07:49:31.008Z" agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36" etag="EzaVlOSwZoLYXGqv3rbj" version="21.3.2" type="github">
  <diagram name="페이지-1" id="Ubigl4BfTGSjSFXbpSqd">
    <mxGraphModel dx="1194" dy="610" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="1" pageScale="1" pageWidth="827" pageHeight="1169" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <mxCell id="Li0bfwf8Ni7NL60AEftY-6" value="yuran vpc" style="points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],[1,0.25],[1,0.5],[1,0.75],[1,1],[0.75,1],[0.5,1],[0.25,1],[0,1],[0,0.75],[0,0.5],[0,0.25]];outlineConnect=0;gradientColor=none;html=1;whiteSpace=wrap;fontSize=12;fontStyle=0;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_vpc;strokeColor=#248814;fillColor=none;verticalAlign=top;align=left;spacingLeft=30;fontColor=#AAB7B8;dashed=0;" parent="1" vertex="1">
          <mxGeometry x="30" y="30" width="548.5" height="240" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-1" value="s3 gateway &lt;br&gt;endpoint" style="text;html=1;align=center;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;" vertex="1" parent="Li0bfwf8Ni7NL60AEftY-6">
          <mxGeometry x="469.5" y="152" width="80" height="40" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-9" value="" style="sketch=0;outlineConnect=0;fontColor=#232F3E;gradientColor=none;fillColor=#B0084D;strokeColor=none;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;fontStyle=0;aspect=fixed;pointerEvents=1;shape=mxgraph.aws4.endpoint;" vertex="1" parent="Li0bfwf8Ni7NL60AEftY-6">
          <mxGeometry x="470.5" y="60" width="78" height="78" as="geometry" />
        </mxCell>
        <mxCell id="raUuo1lHQiTWlKniR-Hy-2" value="Private subnet" style="points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],[1,0.25],[1,0.5],[1,0.75],[1,1],[0.75,1],[0.5,1],[0.25,1],[0,1],[0,0.75],[0,0.5],[0,0.25]];outlineConnect=0;gradientColor=none;html=1;whiteSpace=wrap;fontSize=12;fontStyle=0;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_security_group;grStroke=0;strokeColor=#147EBA;fillColor=#E6F2F8;verticalAlign=top;align=left;spacingLeft=30;fontColor=#147EBA;dashed=0;" parent="1" vertex="1">
          <mxGeometry x="50" y="60" width="430" height="180" as="geometry" />
        </mxCell>
        <mxCell id="raUuo1lHQiTWlKniR-Hy-5" value="" style="outlineConnect=0;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;shape=mxgraph.aws3.ec2;fillColor=#F58534;gradientColor=none;" parent="raUuo1lHQiTWlKniR-Hy-2" vertex="1">
          <mxGeometry x="130" y="51" width="76.5" height="93" as="geometry" />
        </mxCell>
        <mxCell id="raUuo1lHQiTWlKniR-Hy-4" value="" style="html=1;verticalAlign=bottom;labelBackgroundColor=none;endArrow=oval;endFill=0;endSize=8;rounded=0;" parent="raUuo1lHQiTWlKniR-Hy-2" target="raUuo1lHQiTWlKniR-Hy-6" edge="1">
          <mxGeometry width="160" relative="1" as="geometry">
            <mxPoint x="204" y="97" as="sourcePoint" />
            <mxPoint x="364" y="97" as="targetPoint" />
          </mxGeometry>
        </mxCell>
        <mxCell id="raUuo1lHQiTWlKniR-Hy-6" value="" style="sketch=0;outlineConnect=0;fontColor=#232F3E;gradientColor=none;fillColor=#3F8624;strokeColor=none;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;fontStyle=0;aspect=fixed;pointerEvents=1;shape=mxgraph.aws4.bucket;" parent="1" vertex="1">
          <mxGeometry x="650" y="115" width="75" height="78" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-2" value="yuran vpc" style="points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],[1,0.25],[1,0.5],[1,0.75],[1,1],[0.75,1],[0.5,1],[0.25,1],[0,1],[0,0.75],[0,0.5],[0,0.25]];outlineConnect=0;gradientColor=none;html=1;whiteSpace=wrap;fontSize=12;fontStyle=0;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_vpc;strokeColor=#248814;fillColor=none;verticalAlign=top;align=left;spacingLeft=30;fontColor=#AAB7B8;dashed=0;" vertex="1" parent="1">
          <mxGeometry x="30" y="310" width="560" height="240" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-3" value="" style="sketch=0;outlineConnect=0;fontColor=#232F3E;gradientColor=none;fillColor=#4D27AA;strokeColor=none;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;fontStyle=0;aspect=fixed;pointerEvents=1;shape=mxgraph.aws4.endpoints;" vertex="1" parent="LuS5X9uhbVAc7QKGtAih-2">
          <mxGeometry x="446" y="93" width="59" height="59" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-4" value="s3 interface&lt;br&gt;endpoint" style="text;html=1;align=center;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;" vertex="1" parent="LuS5X9uhbVAc7QKGtAih-2">
          <mxGeometry x="432.5" y="152" width="90" height="40" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-11" value="" style="sketch=0;points=[[0,0,0],[0.25,0,0],[0.5,0,0],[0.75,0,0],[1,0,0],[0,1,0],[0.25,1,0],[0.5,1,0],[0.75,1,0],[1,1,0],[0,0.25,0],[0,0.5,0],[0,0.75,0],[1,0.25,0],[1,0.5,0],[1,0.75,0]];outlineConnect=0;fontColor=#232F3E;gradientColor=#945DF2;gradientDirection=north;fillColor=#5A30B5;strokeColor=#ffffff;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;fontStyle=0;aspect=fixed;shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.vpc_privatelink;" vertex="1" parent="LuS5X9uhbVAc7QKGtAih-2">
          <mxGeometry x="520" y="93" width="67" height="67" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-5" value="Private subnet" style="points=[[0,0],[0.25,0],[0.5,0],[0.75,0],[1,0],[1,0.25],[1,0.5],[1,0.75],[1,1],[0.75,1],[0.5,1],[0.25,1],[0,1],[0,0.75],[0,0.5],[0,0.25]];outlineConnect=0;gradientColor=none;html=1;whiteSpace=wrap;fontSize=12;fontStyle=0;container=1;pointerEvents=0;collapsible=0;recursiveResize=0;shape=mxgraph.aws4.group;grIcon=mxgraph.aws4.group_security_group;grStroke=0;strokeColor=#147EBA;fillColor=#E6F2F8;verticalAlign=top;align=left;spacingLeft=30;fontColor=#147EBA;dashed=0;" vertex="1" parent="1">
          <mxGeometry x="50" y="340" width="430" height="180" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-6" value="" style="outlineConnect=0;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;shape=mxgraph.aws3.ec2;fillColor=#F58534;gradientColor=none;" vertex="1" parent="LuS5X9uhbVAc7QKGtAih-5">
          <mxGeometry x="130" y="51" width="76.5" height="93" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-7" value="" style="html=1;verticalAlign=bottom;labelBackgroundColor=none;endArrow=oval;endFill=0;endSize=8;rounded=0;" edge="1" parent="LuS5X9uhbVAc7QKGtAih-5" target="LuS5X9uhbVAc7QKGtAih-8">
          <mxGeometry width="160" relative="1" as="geometry">
            <mxPoint x="204" y="97" as="sourcePoint" />
            <mxPoint x="364" y="97" as="targetPoint" />
          </mxGeometry>
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-8" value="" style="sketch=0;outlineConnect=0;fontColor=#232F3E;gradientColor=none;fillColor=#3F8624;strokeColor=none;dashed=0;verticalLabelPosition=bottom;verticalAlign=top;align=center;html=1;fontSize=12;fontStyle=0;aspect=fixed;pointerEvents=1;shape=mxgraph.aws4.bucket;" vertex="1" parent="1">
          <mxGeometry x="650" y="395" width="75" height="78" as="geometry" />
        </mxCell>
        <mxCell id="LuS5X9uhbVAc7QKGtAih-12" value="private link" style="text;html=1;align=center;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;" vertex="1" parent="1">
          <mxGeometry x="545" y="471" width="80" height="30" as="geometry" />
        </mxCell>
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>



https://zigispace.net/1191