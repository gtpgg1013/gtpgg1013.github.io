---
title: "K8s 뉴비의 쿠버네티스 정복기 (2)"
excerpt: "1.K8s 개념 이해"

# permalink: /categories/kubernetes/2/

categories:
  - develop
  - kubernetes
tags:
  - k8s
  - develop
last_modified_at: 2021-04-22T00:21:00-05:00

comments: true
---

## 1. K8s 개념 이해

### Master / Node(Worker)

- Master : 클러스터 전체 컨트롤러 : HA를 위해 여러개를 두고 master node를 두고, L4 레벨에서 로드 밸런서로 묶어서 종종 사용하는듯.
- Node : 컨테이너가 배포되는 곳 (Worker Node라고도 부름)

### Object

- 기본적 구성 단위가 되는 기본 오브젝트(Basic Object) / 추가적 기능 가진 컨트롤러(Controller)로 분리

  - 오브젝트 = 리소스 ?

- Object Spec

  - 오브젝트는 모두 설정 정보를 기술한 스펙으로 정의 : yaml / json 파일로 관리

- Basic Object

  - 컨테이너화되어 배포되는 애플리케이션의 워크로드를 기술하는 오브젝트
  - Pod / Service / Volume / Namespace 4가지 존재
  - Pod : 컨테이너화된 App
    - 가장 기본적 배포단위, 1개 이상의 컨테이너 포함
    - Pod 내 컨테이너는 IP와 Port를 공유 : Pod 내 통신을 localhost:8000 이런 식으로 통신 가능
    - Pod 내 컨테이너 간에는 디스크 볼륨 공유 가능
      - 같은 파일 시스템을 공유하기 때문에, 하나의 Pod 내에서는 다른 컨테이너의 파일 읽기 가능
  - Service : 로드밸런서
    - 보통 Pod로 서비스 제공시, 그냥 Pod로 제공하지 않고 여러개의 Pod를 서비스하면서, 이를 로드밸런서로 하나의 IP와 포트를 묶어서 서비스 제공
    - Pod는 동적생성되고, 리스타트시 IP가 바뀌기 때문에 Pod 목록을 LB가 유연하게 선택 가능해야 함 => 라벨 / 라벨 셀렉터의 도입
      - 서비스 정의시 어떤 pod를 서비스로 묶을지 정의 : 라벨 셀렉터
      - 메타 정보에 라벨 정의하면 / 서비스가 라벨 셀렉터에서 특정 라벨 가지고 있는 Pod만 선택하여 서비스에 묶음
  - Volume : 디스크
    - Pod 기동시 컨테이너마다 로컬 디스크를 생성하는데, 영구적이지 못해서 위험함 : Pod 내려가면 유실
    - 컨테이너 리스타트 상관없이 파일을 영속 저장
    - 컨테이너의 외장 디스크 / Pod 기동시 컨테이너에 마운트하여 사용
    - 다양한 외장 디스크를 추상화된 형태로 제공 : 온프렘 / 클라우드
  - Namespace : 패키지명 (논리적 분리된 클러스터)
    - 쿠버 클러스트 내 논리적 분리단위
    - 접근 권한 다르게 운영 가능
    - 리소스 쿼타(할당량) 지정 가능
    - 네임스페이스별 리소스 나워서 관리 가능
  - Label
    - 리소스 선택시 사용
    - 각 리소스는 라벨을 가질 수 있고, 특정 라벨을 가지고 있는 리소스만을 선택 가능
    - 라벨 선택하여 특정 리소스만 배포 / 업데이트 할 수도 있고, 선택된 리소스만 Service에 연결하거나 특정 라벨로 선택한 리소스에만 네트워크 접근 권한을 부여하는 등의 행위 가능
    - metadata 섹션에 키/값 쌍으로 정의
    - 하나의 리소스에 여러 라벨 동시 적용 가능

- Controller

  - 위의 기본 4개 오브젝트로도 App. 설정 및 배포는 가능하지만, 좀더 편리하게 하기 위해 추상화 한 것
  - 기본 오브젝트 생성 / 관리 역할
  - Replication Controller
    - Pod 관리 역할 : 지정된 숫자 Pod 가동 / 관리
    - 크게 3 파트로 구성
      - Replica 수 : RC로 관리되는 Pod의 수 : 숫자만큼 Pod 수 유지
      - Pod Selector : 라벨 기반으로 RC가 관리하는 Pod 가지고 오는데 사용
      - Pod Template : Pod 정보 (도커 이미지, 포트, 라벨) 정의
    - 이미 Pod 도는 상태에서 RC 리소스 생성하면 해당 Pod의 라벨이 RC의 라벨과 일치하면 새로 생성된 RC의 컨트롤을 받음
      - 해당 RC 기준으로 Replica 숫자도 조정
      - 모자르면 Template 기준으로 생성
      - 기존 Pod가 Template와 달라도 그대로 유지
  - ReplicaSet
    - RC의 새 버전
    - RC는 Equality 기반 Selector 사용 / RS는 Set 기반 Selector 사용
      - Equality : 등가조건 : environment = dev
      - Set : 집합 : environment in (production, qa)
  - Deployment
    - RC와 RS의 더 상위 추상화 개념
    - 실제 운영에서는 RC, RS보다 Deployment를 더 많이 사용
    - 블루 / 그린 배포 : 새로운 RC Template로 Pod 생성 후, 서비스를 새로운 Pod로 옮김
    - 롤링 업그레이드 : Pod 하나씩 업그레이드 해감
      - 배포 중 문제가 생기면 수동으로 할 일이 생김
      - 이러한 과정을 자동화하고 추상화한 개념
    - 실질적으로 Pod 배포를 위해 RC 생성 / 관리 : 롤백 위한 기존 버전의 RC 관리 등 여러기능 포괄적 포함

출처 - 조대협님 블로그 (https://bcho.tistory.com/1255)
