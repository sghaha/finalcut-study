## 1. 핵심 컨셉
- 클러스터 아키텍처
- API primitives
- Services & Other Network Primitives


## 2. k8s의 목적
- 자동화된 방식으로 우리의 응용프로그램을 컨테이너 방식으로 호스팅

## 3. 워커노드
- 컨테이너를 로딩할 수 있는 배라고 생각하면됨

## 4. 마스터 노드(컨트롤 플레인)
- 워커노드를 관리하고 컨트롤함
- 클러스터를 관리
- 노드에 대한 정보를 저장
- 어떤 컨테이너가 어디에 있을지 계획
- 노드와 컨테이너를 모니터링
- 등등

## 5. ETCD cluster
- 어디에 어떤 컨테이너가 있고, 언제부터 적재되었고, 같은 정보를 키-밸류로 저장
- "앳,씨티" 정도로 읽음

## 6. kube-scheduler

## 7. Controller-Manager (Node-Controller / Replication-Controller)
- 컨테이너가 손상되거나 파손되면 새 컨테이너를 준비해주거나 함
### 7.1. Node-Controller
- 노드를 관리, 사용 불가능하거나 파괴되는 상황을 처리
### 7.2 Replication-Controller
- 원하는 컨테이너 수가 Replication 그룹에서 항상 실행되도록

## 8. kube-apiserver
- k8s 주요 관리 구성요소
- 클러스터내에서 모든 작업을 오케스트레이션함
- 주기적으로 kubelet으로부터 상태 정보를 가져옴 (컨테이너를 모니터링 하기 위해)

## 9. 컨테이너를 적재하기 위해 모든 노드에는 도커가 설치되어있음
- 사실 무조건 도커일 필요는 없긴하다
  - 여담 : v1.24 부터 dockershim지원을 종료했다는데 궁금하면 나중에 더 알아보자

## 10. kubelet (큐블렛)
- 모든 노드에는 선장이 있다.
- 선장은 각 노드와 통신을 맡는다.
- 클러스터의 각 노드에서 실행되는 에이전트
- kube api 서버의 지시를 듣고 행동을 취함(컨테이너를 배포하거나 파괴)

## 11. kube-proxy
- 하나에 노드에 was가 있고 다른 노드에 db가 있다고 생각해보자 어떻게 통신할까? 이걸 해주는게 kube-proxy service
- 워커 노드에 필요한 룰이 실행되도록 합니다.
- 서로가 통신할 수 있도록


## 12. 마스터 노드에 있는것
- etcd클러스터 : 클러스터에 관한 정보를 저장
- kube-scheduler : 노드의 응용프로그램이나 컨테이너의 스케즁을 짜는 역할
- kube-apiserver : 서버내에서 모든 작업을 오케스트레이션 함
- controller-manager

## 13. 워커노드에 있는것
- kubelet : kube api 서버의 지시를 듣고 컨테이너와 kube-proxy를 관리
- kube-proxy : 클러스터 내부의 서비스간 통신을 가능캐함
- 컨테이너 엔진(도커같은거)



---
## 14. ETCD
- 분산되고 신뢰할 수 있는 키 밸류 스토어
- "엣,씨디"정도로 발음
- 실행하면 기본적으로 2379 포트에서 동작

### 14.0. etcd가 저장하는 정보
- nodes
- PODs
- Configs
- Secrets
- Accounts
- Roles
- Bindings
- Others

### 14.1. key-value store
- key-value 장점 : 
  1) 컬럼을 하나 만들때 모든 row에 대해 적용하지 않아도 된다.
  2) 다른 문서를 업데이트 하지 않고 추가적인 세부사항을 넣을 수 있다.
  
### 14.2. ETCD Controller client
* 저장하려면
```
./etcdctl set key1 value1
```
* v3에서는
```
./etcdctl put key1 value1
```
* 가져오려면
```
./etcdctl get key1 
```


---
## 15. kube-apiserver
* pod를 생성한다고 생각하자
  1) 사용자가 kubectl 명령을 내리면
  2) kube-apiserver가 그걸 받아서 인증 처리하고
  3) 요청을 검증하고
  4) 기존 데이터를 etcd에서 확인하고
  5) etcd를 업데이트 하고
  6) kube-schduler는 api서버를 지속적으로 관찰하고 노드에 할당되지 않은파드가 있다는걸 알아치리고
  7) 이때 스케쥴러가 올바른 노드에 새 파드를 넣어주고 api서버와 통신을함
  8) 이제 api서버가 kubelet에게 알려주고
  9) 큐블렛은 파드를 생성하고 컨테이너 런타임 엔젠에 지시해서 배포함
  10) 완료되면 큐블렛은 상태를 api서버로 보냄
  11) api서버는 다시 etcd데이터를 업데이트 함

- 클러스터에서 변경을 위해 수행해야하는 모든 작업의 중심에 있다.
- 요청의 인증과 유효성을 확인하고 데이터 스토어에서 데이터를 검색하고 업데이트함
- kube-apiserver는 사실 저장소(etcd)와 상호작용하는 유일한 구성요소임

---
## 16. kube controller manager
관제소 같은 느낌임
- 상태를 살피고
- 상황을 조정하기 위해 조치를 취함

### 16.1 Node Controller
- Node의 상태를 모니터링하고
- 5초마다 kube api server에 노드 상태를 물어본다 (Node Monitor period = 5s)
- 노드의 신호가 안잡히는건 40초 주기로 알 수 있음(Node Monitor Grace period = 40s)
- 파드가 다시 뜰 때까지는 5분이 걸린다. (POD Eviction Timeout = 5m)