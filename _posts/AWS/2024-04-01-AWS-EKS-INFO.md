---
layout:   post
title:    "EKS"
subtitle: "EKS 학습"
category: AWS
more_posts: posts.md
tags:     AWS
---
# [AWS-EKS] AWS EKS 특징과 노드 구성 방식, 배포방식

<!--more-->
<!-- Table of contents -->
* this unordered seed list will be replaced by the toc
{:toc}


## Amazon EKS란?

>Kubernetes의 컨트롤 플레인 또는 노드를 제공하는 AWS의 관리형 K8S 서비스

즉, **AWS**에서 제공하는 **탄력적으로 쿠버네티스를 관리할 수 있는 서비스**이다.

사실 쿠버네티스가 가장 주목받고, 컨테이너 오케스트레이션 툴의 1인자로써 올라선 만큼 AWS 뿐만 아닌 MS나 Goolge과 같은 대형 CSP에서도 각각 AKS, GKE라는 이름으로 관리형 쿠버네티스를 제공한다.

이러한 EKS의 노드 영역은 구성 방식에 따라서 AWS 관리형일 수도 있고, 아닐 수도 있다.

## EKS의 특징

먼저 이러한 EKS의 특징에 대해서 알아보자.
#### 1. 다수의 가용영역에 Kubernetes 컨트롤 플레인을 실행
![](/assets/img/AWS/eks/337ca019-a327-4d08-890c-e409ce9fd5a4-image.png)

즉, 우리가 아는 컨트롤 플레인의 영역에 있는 `API 서버`, `etcd`, `Contoller Manager`, `Scheduler`가 위치한 **컨트롤 플레인이 다수의 가용영역에 배치**된다는 점이다.


#### 2. 다양한 AWS 서비스와 통합하여 확장성 및 보안성 제공

또한 AWS에서 제공하는 서비스인 만큼 Kubernetes 환경에 대해서 AWS의 다양한 서비스를 통합하여 확장성있게 사용할 수 있다.

예를 들어 컨테이너 이미지 저장소로 ECR을 사용하거나, Ingress를 생성할 때 AWS ELB 타입으로 생성한다던지, VPC내에 EKS를 배치하여 VPC내 IP를 파드에 할당하는 등 밀접하게 연관되어 사용할 수 있다.
![](/assets/img/AWS/eks/f9159be2-e168-4706-9b39-d382055cef9d-image.png)

#### 3. EKS는 K8S의 최신버전을 사용한다.

![](/assets/img/AWS/eks/749f0189-dd31-4fa9-8c63-d635c0503189-image.png)

AWS EKS는 kubernetes의 최신버전을 사용하는데, 이를 통해서 kubenetes의 다양한 플러그인과 도구를 활용할 수 있다. 또한 EKS로의 마이그레이션 또한 쉽게 가능하다.

[EKS - 공식문서](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/kubernetes-versions-standard.html)

>**💡 TIP 쿠버네티스 버전 표기법**
앞에서 부터 Major, Minor, Patch를 의미한다.
![](/assets/img/AWS/eks/8dece50b-bf4d-4bd3-9254-7d8a0867040b-image.png)

---




## Amazon EKS Control Plane과 Data Plane

EKS의 **EKS Control Plane과 Data Plane**이 어떻게 배치되는지 알아보자.

### EKS Control Plane (마스터 노드)
> EKS Control Plane은 Kubernetes 클러스터의 주요 관리 기능을 담당하며, 다음과 같은 구성 요소로 이루어져 있다.

- Kubernetes API 서버: 클러스터에 대한 모든 API 요청을 처리
- etcd: 클러스터의 상태 정보를 저장하는 고가용성 키-값 저장소
- 스케줄러: 파드(pod)가 클러스터의 어느 노드에 배치될지 결정
- 컨트롤러 관리자: 클러스터의 상태를 모니터링하고, 원하는 상태를 유지하도록 조정


먼저 AWS가 관리하는 **AWS Managed VPC내 EKS Control Plane**을 살펴보자.
![](/assets/img/AWS/eks/58be5927-2d72-494e-91c7-efe5be15b4b7-image.png)

**EKS Control Plane**이 위치하는 `AWS Managed VPC`는 VPC의 세부 구성이나 네트워크 설정을 사용자가 직접 **확인하거나 변경할 수 없다.** 🙅🏻‍♂️
이는 AWS가 EKS의 컨트롤플레인을 위해 **별도로 관리하는 인프라**이기 때문이다.

일반적으로 3개의 가용영역에 각각 한개씩 총 3개가 구성되어 동작되며, 이를 통해 하나의 가용 영역에 장애가 발생하더라도 나머지 영역에서 컨트롤플레인의 운영이 계속 유지될 수 있도록 한다.

또한 고가용성 및 트래픽 분산을 위한 `ELB`나, `ASG`가 내부적으로 동작하도록 설계되어있다.

### EKS Data Plane (워커 노드)
![](/assets/img/AWS/eks/86770653-dbf0-4187-8bea-42b731073f0c-image.png)

사용자가 구성하고 볼 수 있는 영역🙆🏻‍♂️은 **워커노드의 영역인 Data Plane**이다.

워커노드가 사용자의 계정 내 VPC(그림에서의 Custom VPC)에 **직접적으로 배치**되기 때문에 **사용자가 직접 구성하고 관리**할 수 있다.


#### 컨트롤 플레인과의 통신 방식
![](/assets/img/AWS/eks/1897af60-2860-4dba-b138-3b9dfb99e166-image.png)

AWS가 제공하는 컨트롤 플레인과 사용자의 워커노드가 **서로 다른 VPC임에도 통신이 가능**한 이유는 **Cross Account ENI**를 사용하기 때문이다. 

이는 EKS owned ENI라는 네트워크 인터페이스를 통해서 AWS가 제공하는 컨트롤플레인과 사용자가 만든 Data Plane의 워커노드가 서로 다른 VPC임에도 통신이 가능하다.


#### 그럼 워커노드의 가용영역의 수는 3개가 고정인가요?

그렇다면 AWS에서는 그림처럼 **3개의 컨트롤 플레인을 제공/관리** 해주니, 우리가 EKS를 다룰 때는 **항상 최소 3개의 가용영역과, 워커노드를 구성해야할까**?

>**구성해 봤다면 알겠지만, 정답은 No다.**🙅🏻‍♂️

AWS에서 제공하는 EKS(Elastic Kubernetes Service) 컨트롤플레인의 가용 영역 개수는 워커 노드가 위치하는 노드 그룹에서 설정하는 가용 영역의 개수와는 **독립적이다.**

즉, 사용자가 워커 노드를 위한 노드 그룹에서 가용 영역을 **하나만** 설정하더라도, **AWS에서 EKS 컨트롤플레인은 여전히 여러 가용 영역에 구성**해준다.








## EKS 노드구성 방식


### 1. 관리형 노드 그룹 (Managed Node Groups)

AWS에서 **노드의 프로비저닝과 관리를 대신**해주는 방법이다.
이를 통해 사용자는 **노드의 설정, 업데이트, 스케일링 등을 쉽게 관리** 할 수 있다.
예를 들어 온라인 쇼핑몰 애플리케이션에서 관리형 노드 그룹을 사용하면 AWS가 필요한 워커 노드를 **자동으로 프로비저닝하고, 트래픽에 따라 자동으로 스케일링**해준다.

### 2. 자체 관리형 노드 (Self-managed Nodes)

사용자가**직접 EC2 인스턴스를 프로비저닝하고 관리**하는 방식이다.
더 많은 커스터마이징과 세밀한 제어가 가능하지만, 관리 부담이 높다.
만약 **특정 유형의 인스턴스를 사용하거나, 특별한 네트워크 설정이 필요하다면, 자체 관리형 노드를 사용**해야한다.

### 3. AWS Fargate

**서버리스 컴퓨팅 엔진**으로, 사용자가 서버나 클러스터의 관리 없이 컨테이너를 실행할 수 있게 해준다.
사용한 컴퓨팅 자원에 대해서만 비용을 지불하는데, 서버 관리에 신경 쓰지 않고**간단한 웹 애플리케이션 또는 마이크로서비스를 빠르게 배포**하고 싶다면, Fargate를 사용하여 운영 부담 없이 애플리케이션을 실행할 수 있다.

---

## EKS Cluster 배포 방법


#### 1. 관리 콘솔을 통한 수동 배포

![](/assets/img/AWS/eks/1fabe74d-86b8-40ad-8a7e-7566b6da366b-image.png)

관리콘솔에서 수동으로 작업하는 것은 사용자 인터페이스를 잘 보여줘서 가시성이 높지만, 설정을 손수 하나하나 해야하는 단점이 있다.

또한 이 방식은 초기에 간단한 테스트를 진행하거나, 작은 서비스에서는 운영하기 쉽지만 이후 서비스가 커지며 업그레이드나 확장 및 유지 관리가 필요한 시점에는 비 효율적이게 된다.

따라서 작은 규모의 클러스터에 적합할 수 있지만, 복잡한 인프라 및 스케일링 요구사항이 있을 때는 효율적이지 못하다.


#### 2. eksctl를 통해 배포
![](/assets/img/AWS/eks/c52f2ebc-1792-4839-b21a-1b0a41079ad6-image.png)

eksctl 명령어를 통해서 클러스터를 생성, 삭제 및 관리가 가능하다. 명령어에 대한 기본적인 숙지가 있으면 관리콘솔보다 훨씬 빠르게 배포할 수 있다. 


#### 3. Iac 방식의 배포
![](/assets/img/AWS/eks/a3c2ef7a-a2d7-41b2-a363-4039680dfcbb-image.png)

IaC(Infrastructure as Code)방식을 통해서 인프라를 **코드 형태로 손쉽게 배포할** 수 있다. 
ex.) **CloudFormation, Terraform**

AWS CloudFormation은 인프라를 코드로 관리할 수 있는 서비스다. 이를 통해 사용자는 JSON 또는 YAML 형식의 템플릿 파일을 작성하여 AWS 리소스(예: EC2 인스턴스, S3 버킷, RDS 데이터베이스 등)를 정의하고, 이를 기반으로 자동으로 리소스를 생성, 업데이트, 삭제할 수 있다. CloudFormation은 인프라를 코드로 관리하고, 버전 관리 및 반복 가능한 인프라 배포를 가능하게 하여 DevOps 및 자동화 작업을 용이하게 한다.

Terraform의 경우, HCL이라는 특수한 언어를 알아야한다는 단점이 있지만, Terraform을 사용하면 CSP에 종속받지 않는 통합된 인프라스트럭처 코드를 작성할 수 있다.
무엇보다 관리 콘솔에서는 하기 힘들었던 EKS 클러스터 뿐만 아니라 **VPC, 서브넷, 보안 그룹, IAM 그룹**,**애드온** 등 모든 리소스를 **프로그래밍적으로 관리**할 수 있다.

>💡 **애드온이란?**
Kubernetes 클러스터의 기능을 확장하거나 보강하는 추가적인 컴포넌트나 소프트웨어
(Prometheus & Grafana, Istio, EFK, Helm, ArgoCD 등등..)


💬 덧붙이자면, 사실 `1. 관리 콘솔을 통한 수동 배포`와 `2. eksctl 방식` 모두 Amazon EKS(Elastic Kubernetes Service) 배포 시 내부적으로 **AWS CloudFormation**이 사용된다.
  
---
**Reference📎** | [CloudNet@와 함께하는 Amazon EKS 기본 강의](https://www.inflearn.com/course/amazon-eks-기본-강의)