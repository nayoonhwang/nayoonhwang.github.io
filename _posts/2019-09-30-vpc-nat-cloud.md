---
layout: post
title:  "VPC and NAT"
date:   2019-09-30 11:22:22 +0900
categories: cloud network
---

VPC:
https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098


# VPC의 기본 컴포넌트들

## VPC
VPC는 Virtual Private Cloud의 줄임말로 "가상 IP 네트워크" 로 이해할 수 있다. 이는 클라우드 내에 가상 데이터 센터라고 할 수 있다. VPC내에 있는 서버들끼리는 연결되어있으며, 사설 IP(Private IP)를 통해 서로 통신할 수 있다. 

> Note: 사설 IP란?
> 예를 들어 '우리집 주소는 "서울시 성동구 마장로 ~" 이다' 했을때 "서울시~"는 공인 IP라 할 수 있다. 반면 집 안에서 "안방에 있는 손톱깎기 가져와" 했을 때 "안방" 은 사설 IP라고 할 수 있다. 즉 외부에서 어떤 위치를 칭하는 주소는 공인 IP이고 내부에서는 사설 IP로 위치를 찾는다. 

VPC에서 사용하는 사설 IP 주소 대역은 다음과 같다.
| 주소 | Prefix 비트 수 | 
|:--------|:--------:|
| 10.*.*.* | 8 |
| 172.16.*.* | 12 |
| 192.168.*.* | 16 |

![](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/nondefault-vpc-diagram.png)

특정 VPC는 한 Region에 속해있고, 각각의 VPC는 완전히 독립적이다.

사설 IP 주소를 선택하는거나, subnet을 만드는거, 라우트 테이블, 네트워크 게이트웨이 VPC는 클라우드 상의 다른 가상 네트워크와 논리적으로 분리되어있다. VPC 내에서 리소스를 띄울 수 있는데, 해당 VPC에 특정 IP 주소 범위를 할당하고, 서브넷을 추가하고, 보안 그룹을 연결하고 라우트 테이블을 설정할 수 있다. VPC의 장점은 프라이버시, 보안, 개인정보 데이터 유실같은 측면에서 장점을 가진다.

## Subnet
서브넷은 대형 네트워크를 소형 네트워크들로 나눈 것이라고 생각하면 된다. 당연히 가용한 IP 주소 범위도 더 작아지게 되지만, 작은 네트워크의 유지관리가 좀 더 수월하고, 다른 네트워크로부터의 보안등을 제공하기 위해 사용한다. 정 서브넷 내부에서 리소스를 띄울 수 있는데, 인터넷에 반드시 연결되어야하는 리소스들을 위해서는 Public 서브넷을 사용하고, 인터넷에 연결될 필요없는 리소스들을 위해서는 Private subnet을 사용하면 된다. 각 서브넷의 리소스를 보호하기 위해서는, 보안 그룹이나 네트워크 접근 제어 리스트(ACL)등 여러가지를 적용할 수 있다.

## Route Table
라우트 테이블은 트래픽의 방향을 정해주는 안내표이다. 즉, 발생한 네트워크 요청이 올바른 타겟으로 향할 수 있도록 한다. VPC내에 여러개의 라우트 테이블을 가질 수 있다.

## Internet Gateway
인터넷 게이트웨이는 구축한 가상 네트워크를 외부와 연결하기 위한 관문이다. 
![](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/internet-gateway-overview-diagram.png)

위 그림과 같이 Public subnet의 Route Table에 0.0.0.0/0 에서 온 요청을 igw-id로 Mapping한다. 즉, 내부(선택한 사설 IP대역)에서 온 요청이 아닌, 외부(0.0.0.0)에서 온 네트워크 트래픽은 인터넷 게이트웨이로 향하게 하도록 한다.

## NETWORK ACL and Security Group
"접근 제어 목록"인 ACL은 각 Subnet에 설정할 수 있는 접근 규칙이다. 일종의 방화벽과 비슷하다. 보안그룹은 인스턴스 수준에서 작동하는 반면 VPC는 서브넷 레벨에서 작동한다. 웹 서버냐, 데이터베이스 서버냐에 따라 이 보안 그룹의 Rule은 다를 수 있다. 예를 들어 웹서버에서는 외부의 HTTP/HTTPS 요청을 모두 허용하고 데이터베이스 서버는 HTTP대신 인바운드 MySQL 액세스를 허용하는 규칙을 사용할 수 있다. 

## NAT
Private Subnet이 외부와 통신할 때, NAT는 사설IP 주소를 공인 IP와 map해주는 역할을 한다. Public Subnet상에서 Private Subnet 상의 인스턴스의 인바운드/아웃바운드 요청을 처리한다.

Security groups: Security groups are a set of firewall rules that controls the traffic for your instance. In Amazon Firewall the only action that can be carried out is allow. You cannot create a rule to deny. The destination is always the instance on which the service security group is running. You can have a single security group associated with multiple instances.

Customer Gateway — An Amazon VPC VPN connection links your data center (or network) to your Amazon VPC (virtual private cloud). A customer gateway is the anchor on your side of that connection. It can be a physical or software appliance

NAT:

https://blog.lael.be/post/9534
