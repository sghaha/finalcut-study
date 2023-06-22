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
- k8s에는 아래 두개의 컨트롤러 말고도 컨트롤러가 많음 (디플로이먼트 컨트롤러, 네임스페이스 컨트롤러, 엔드포인트 컨트롤러, 잡 컨트롤러 등등)
- 이 많은 컨트롤러들이 kube controller manager로 패키지화 되어있다.

### 16.1 Node Controller
- Node의 상태를 모니터링하고
- 5초마다 kube api server에 노드 상태를 물어본다 (Node Monitor period = 5s)
- 노드의 신호가 안잡히는건 40초 주기로 알 수 있음(Node Monitor Grace period = 40s)
- 파드가 다시 뜰 때까지는 5분이 걸린다. (POD Eviction Timeout = 5m)

### 16.2 Replication Controller

- Replication Set의 상태를 모니터링하고 원하는 수의 파드가 세트내에서 항상 떠있도록함
- 파드가 죽으면 다른 파드가 생김

---
## 17. Kube Scheduler
- 노드 파드의 일정 관리를 책임짐
- 스케쥴러는 어떤 파드가 어떤 노드에 들어갈지만 결정함
- 파드를 노드에 두는건 큐블렛이 할일임, 스케쥴러는 결정만함 

### 17.1 왜 스케쥴러가 필요한가?
- 알맞은 컨테이너를 알맞은 배에 실어야함
- 컨테이너와 배 크기가 다를수 있음 각각에 알맞게 배정해줘야함
- 각 배의 목적지가 다를수도있음
- k8s는 스케쥴러가 특정기준에 따라 포드를 어느 노드에 놓을지 정함

### 17.1 어떤 노드에 파드를 둘지 결정하는걸 자세히 알아보자
- 파드를 놓을 최고의 노드를 찾으려한다.
- 각파드에는 요구하는 cpu와 RAM등의 요구 사항이 있다.
- 스케쥴러는 두단계를 거쳐 적합한 노드를 찾는다.
  1) 이 파드에 맞지 않는 노드를 걸러낸다.
  2) 남은 노드중 점수를 메긴다. 더 자원이 여유로운 걸 고른다.
- 적합한 노드를 찾을때 점수먹이는건 내가 조절할 수 있다.

### 17.2 스케쥴러 옵션을 불수 있는 파일
```
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```



--- 

## 18. kubelet
- 배의 선장같은 느낌
- 마스터십과의 유일한 연락책
- 배에 컨테이너를 싣거나 내림 
- 스케쥴러가 하라는 대로한다 (물론 스케쥴러는 api server를 통해 알려줌)
- 파드와 컨테이너를 모니터링 하고 동시에 kube api 서버네 보고한다.
- 큐블렛은 다른것과 다르게 무조건 수동으로 설치해야하는 거라고한다.




---
## 19. Kube Proxy
- 파드 네트워크는 내부 가상 네트워크로 모든 포드가 연결되는 클러스터내 모든 노드에 걸쳐있다.
- 이걸 통신하게 해줌
- was파드랑 db파드가 있다 was가 db에 연결 할때 ip를 통해 연결하면 되지만 항상 이 ip가 같을 거라는 보장이 없다.
- 이걸 위해 클러스터에 걸쳐 DB를 노출할 서비스를 생성한다.
- 이제 그 서비스를 이용해서 was가 db에 접근한다.
- 서비스는 거기에 할당된 ip주소도 받는다.

### 19.1 그럼 서비스는 뭐고 ip는 어떻게 얻음?
- 서비스는 pod 네트워크에 조인할 수 없다. 
- 사실 서비스는 실제로 존재하는것이 아니기 때문에(파드 같은 컨테이너가 아니라서)
- k8s 메모리에만 존재하는 가상 구성요소임
- 그럼 어캐 파드들이 서비스에 접근 가능한거임??
- 그래서!!!! 큐브 프록시가 필요함

### 19.1 큐브 프록시 좀더 설명
- k8s 클러스터의 각 노드에서 실행되는 프로세스
- 새 서비스를 찾아줌
- 새 서비스가 생성될 때마다 각 노드에 적절한 규칙을 만들어 그 서비스로 트래픽을 전달함

### 19.2 이걸하기 위한 방법
- iptables규칙을 이용함 



---
## 20. Pods
- 파드는 앱의 단일 인스턴스
- 파드는 k8s에서 만들수 있는 가장 작은 단위
- 만약에 트래픽이 증가해서 스케일아웃을 할떄 새로운 파드를 하나더 띄우면됨
- 더 증가하면?? 근데 노드에 파드가 꽉찼다면?? 노드를 하나 더 띄워서 파드를 더 띄우면됨
- 보통 파드는 앱을 실행하는 컨테이너와 1:1 관계임
- 그런데 같은 파드안에 떠있는 컨테이너를 도와주는 컨테이너가 있을수 있다. 
- 사용자가 업로드한 파일처리 같은경우가 그 경우임, 이경우에는 컨테이너 두개를 같은 포드에 둘수 있다. (이경우는 두 컨테이너는 하나의 로컬호스트에 있다고 생각하면됨)
- 근데 이렇게 하나의 파드에 여러개 컨테이너 띄우는건 드문경우다

### 20.1 pod 배포 방법 예시
1) 파드 띄우기 run
```
kubectl run nginx --image nginx
```
* Docker hub 레포지토리에서 nginx 이미지를 다운로드 받아서 띄운다.

2) pod 목록보기
```
kubectl get pods
```

3) 여기까지 하면
* 외부에서 nginx접근은 하지 못한다. 하지만 내부적으로 액세스는 가능하다.



### 20.2 with yaml
* 크게 아래 4개의 것들을 필드로 가지고있다.
```
apiVersion:
kind:
metadata:


spec:
```

- apiVersion : k8s버전, 우린 파드 생성을 해볼거니까 일단 v1으로 하자
  1) pod : v1
  2) Service : v1
  3) ReplicaSet : apps/v1
  4) Deployment : apps/v1
- kind : 만들려는 유형, 여기서는 Pod
- metadata : 네임이나 레이블 같은거 
- spec : 스펙정보 저장,

   
- 만들어진 yaml vkdlf
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:  #이건 list혹은 Array임 그래서 바로 밑에 - 이게 있는거임
  - name: nginx-container
    image: nginx
```

### 20.3 yaml파일로 pod를 만드는법
```
kubectl create -f pod-definitaion.yml
```
* apply도 사용가능




## 21. 실습 코스 url
https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-administrator-with-practice-tests/

## 22. pod 관련 Lab 메모
- pod몇개있는지 확인해봐라
 ```
kubectl get pods
 ```
- 좀더 다양한 정보를 보고 싶으면
```
kubectl get pods -o wide
```

- nginx 이미지를 이용하여 pod를 띄워봐라  (사실 나는 yaml로 띄웠음)
```
kubectl run nginx --image=nginx
```

- 파드가 어떤 이미지를 사용했는지 알아보자
```
kubectl describe pod <pod이름>
```
여러가지 정보가 있는데 Containers 항목을 보면 된다.



- 파드가 어떤 노드에 떴는지 알아보자
```
kubectl describe pod <pod이름>
```
Node : 항목을 보면 어떤 노드에 뜬지 보인다.

- 아니면
```
kubectl get pods -o wide
```
이걸로 해도 노드 보임



- 파드 안에 몇개의 컨테이너가 있는지 보는것도 describe



- dry run 그리고 yaml 출력 : 아래와 같이 하면 대충 명령어를 yaml로 만들어 주는듯?
```
kubectl run redis --image=redis123 --dry-run=client -o yaml
```
```
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
```


---



## 23. ReplicaSets

### 23.1 Replication Controller
- 레플리카는 무엇이고 Replication Controller 은 왜 필요한가? :
- 한 파드가 고장나면 유저는 서비스를 이용할 수 없다.
- 사용자가 앱을 계속 이용할 수 있도록 고장나면 바로 다른게 실행되어야함 레플리케이션 컨트롤러가 그걸 해준다.

### 23.2 레플리케이션 컨트롤러가 필요한 다른이유
- 로드밸런싱 & 스케일링
- (레플리케이션 컨트롤러 안에 같은 파드가 여러개 띄워져 있다고 생각하면됨)
- 만약 이렇게 파드를 스케일 아웃하다가 노드에 공간이 부족하면 레플리케이션 컨트롤러의 영역이 다른 노드까지 확장됨

### 23.3 레플리케이션 컨트롤러를 생성하는 방법
- 역시나 아래 4가지 섹션이 기본 틀이다
```
apiVersion:
kind:
metadata:


spec:
```

- apiVersion : 레플리케이션 컨트롤러는 v1임
- spec : template : 사용할 pod 템플릿을 제공하기 위함, template하위에는 pod를 적어내면됨(대신 apiVersion이랑 kind는 안써도됨)



```
apiVersion: v1
kind: ReplicationController 
metadata:
  name: myaap-rc
  labels:
    app: myapp
    type: front-end
spec:
  template: 
    metadata:  #여기서부터 ---------------------------------
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx #여기까지가 pod관련 내용임 -----------
```


- 여기까지 하면 부모-레플리케이션 컨트롤러, 자식-pod의 구조가 된다.
- 이제 파드를 몇개 띄워냐 같은 정보가 필요하다



```
apiVersion: v1
kind: ReplicationController 
metadata:
  name: myaap-rc
  labels:
    app: myapp
    type: front-end
spec:
  template: 
    metadata:  #여기서부터 ---------------------------------
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx #여기까지가 pod관련 내용임 -----------
  replicas:  3 ### spec섹션의 자식임
```

- 이런식으로 pod가 뜨면 pod의 이름이 myaap-rc-11jhdf 이런식으로 뜬다는걸 기억해두자


### 23.4. 레플리케이션 컨트롤러 보는 명령어
```
kubectl get replicationcontroller
```



### 23.5 Replica Set 과 Replication Controller
- 둘이 용도는 같지만 완전 똑같은건 아니다.
- controller는 old 기술로 점점 Replica Set으로 대체 되고있다.



### 23.6 replica set 만드는 법을 알아보자
- 역시나 아래 4가지 섹션이 기본 틀이다
```
apiVersion:
kind:
metadata:


spec:
```

- apiVersion : 래플리카셋은 apps/v1임


```
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: myaap-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: 
    metadata:  #여기서부터 ---------------------------------
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx #여기까지가 pod관련 내용임 -----------
  replicas:  3 ### spec섹션의 자식임
```

- 이제 여기서 컨트롤러와 셋의 차이점이 생긴다
- 래플리카셋은 selector라는걸 넣어줘야한다.
- 셀렉터는 래플리카셋으로 놓인 pod를 식별할수 있게 해줌
- 템플릿에서 pod 정의를 했는데 왜 또함?
- 사실 래플리카셋은 스스로 만들지 않은 pod도 관리할 수 있기 때문임 ㅇㅇ
- 셀렉터와 matchLabels를 넣어주면 어떤 pod를 감시할지 알려준다.
- 만약에 matchLabels에 맞는 pod가 미리 떠있으면 새로 pod를 띄우지 않는다.
- 하지만 기존의 pod가 죽거나 할때 새로 띄울때는 template에 있는걸 보고 띄운다.

```
apiVersion: apps/v1
kind: ReplicaSet 
metadata:
  name: myaap-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: 
    metadata:  #여기서부터 ---------------------------------
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx #여기까지가 pod관련 내용임 -----------
  replicas:  3 ### spec섹션의 자식임
  selector: 
    matchLabels:
      type: front-end
```


### 23.7. 래플리카셋 보는 명령어
```
kubectl get replicaset
```


### 23.8 레플리카 갯수를 update하는 방법
1) yaml파일에 숫자 바꿔준다. 그런다음에
```
kubectl replace -f replicaset.yaml
```

2) kubectl 사용 1
```
kubectl scale --replicas=6 -f replicaset.yaml
```

3) kubectl 사용 2
- 아래의 replicaset은 타입이고
- myapp-replicaset은 name임
```
kubectl scale --replicas=6 replicaset myapp-replicaset
```


### 23.9 래플리카 이미지를 바꾸는 방법
```
kubectl edit replicaset <래플리카셋 이름>
```

- 그런다음에  기존에 있던 파드를 지움, 그러면 새로뜸
- 파드수 변경같은거도 kubectl edit으로 가능하다 이경우엔 파일 수정후 나가면 알아서 반영됨



---
## 시험팁
- alias k=kubectl 
- 하면 편함
- yaml 파일을 그냥 쓰는건 너무 힘들다.
- kubectl run 을 써보자
- 그러면 yaml 만드는걸 도와준다.
- nginx pod를 만드는 법
```
kubectl run nginx --image=nginx
```
- pod yaml만드는법 (pod를 실제로 띄우진 않음)
```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

- deployment만드는법
```
kubectl create deployment --image=nginx nginx
```
- deployment yaml 만드는법
```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```
- 파일로 저장까지
```
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

- v1.19이후부터는 --replicas옵션이 생김
```
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

- scale
```
kubectl scale deployment nginx --replicas=4
```

- redis-service라는 이름의 ClusterIP만들기
```
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

- 아니면
```
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```


- 노드 포트 만들기
```
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```
- 아니면
```
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```



- redis라는 이름의 redis:alpine이미지를 사용하는 tier=db레이블을 가지고있는 파드 생성
```
kubectl run redis --image=redis:alpine --labels="tier=db"
```


- webapp이라는 이름의 kodekloud/webapp-color 이미지를 이용하는 replicas가 3인 deployment 만들기
```
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```


- pod를 만든다. pod이름은 custom-nginx, 이미지는 nginx, 8080포트를 노출
```
kubectl run custom-nginx --image=nginx --port=8080
```


- dev-ns라는 네임스페이스를 만든다
```
kubectl create namespace dev-ns
```


- deployment 생성, 이름은 redis-deploy, 이미지는 redis, 네임스페이스 dev-ns, replicas는 2
```
kubectl create deployment redis-deploy --image=redis --namespace=dev-ns --replicas=2
```


- 데몬셋 만들기
```
kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml
```



---


## 24. Deployment
- 래플리카셋에서 배웠던건 잠시 잊자

- 파드가 여러개 있는데 업그레이드를 하거나 배포를 해야할때
- 한번에 교체하는것이 아니라, 하나씩 교체하는게 좋을때가 있다.
- 그리고 배포를 롤백하는 기능도 있으면 좋다.

- 뭐이런 류의 기능을 하는것이 디플로이먼트
- 래플리카 셋을 감싸는 개념이다.

### 24.1 deployment를 만들자
- 래플리카 셋 yaml이랑 되게 비슷하다.
- deployment를 create하면 replicaset도 자동적으로 만들어진다.
```
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: myaap-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas:  3
  selector: 
    matchLabels:
      type: front-end
  template: 
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx
```



---

## 25. Service

- 애플리케이션을 다른 애플리케이션 또는 사용자와 연결하는데 도움을 줌
- 내부의 pod까리 통신을 도와주고
- 외부로 포트를 열어줘 외부의 요청을 받을 수 있게 해줌 (노드포트 서비스라고 함)


### 25.1 서비스 타입
1)  NodePort
2) ClusterIP
3) LoadBalancer

### 25.2 NodePort (노드포트 설명 장표 그림이 있으면 좋음)
- 타겟 포트 : 실제 pod에서 사용하는 포트
- 포트 : 서비스에서 pod와 연결되는 포트
- 노트포트 : 외부에 열려있는 포트 (범위 : 30000~32767)
- 단일노드위에 단일pod든, 단일 노드위에 다중 pod든, 다중 노드위의 다중 pod든 서비스는 정확히 똑같이 생성됨.
- pod를 제거하거나 추가하면 알아서 service도 업데이트됨


### 25.2 서비스를 생성해보자

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
```
- 타겟포트를 적지않으면 그냥 port랑 같다고 생각함
- 노드포트를 적지않으면 자동으로 할당됨 (범위 : 30000~32767)
- 느꼈겠지만 서비스를 생성할때는 pod를 명시해주지 않음 걍 타겟포트만 알려줌
- 그래서 셀렉터를 추가해줌!!!

```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end
```





## 25.3 ClusterIP
- frontend / backend / DB등의 파드가 따로따로 띄어있다고 생각해보자
- 각각 pod끼리 통신해야한다.
- 각각 pod가 3개씩 있다고 할때 frontend는 어떤걸로 backend를 호출해야할까
- backend 파드 3개중 하나를 선택해야하기도하고 각 파드는 언제든 사라질수있어서 할당된 ip가 바뀔수도있다.
- 그래서 예를 들어 백엔드 파드를 위해 만든 서비스는 모든 백엔드 파드를 하나로 묶어 다른 파드가 이 서비스에 접근하도록 한다


## 25.4 만들어 보자
```
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
  - targetPort: 80
    port: 80
  selector:
    app: myapp
    type: back-end
```



## 25.5 LoadBalancer

- 파드가 여러개 있고 이걸 외부에서 접근하게 해주는 nodeport가 있다고 생각해보자
- 그럼 노드 수만큼 ip가 생성된다.
- 하지만 유저에게는 단일화된 endpoint가 필요하다.
- 이걸위해 LoadBalancer를 하나 구축하면 되는데 aws나 에저등등 클라우드 플랫폼은 그들의 로드밸런서 서비스가 있을거다
- k8s는 그것들을 지원하니 그걸 위해 구성하면 된다.
- 따라서 아래 yaml은 지원을 하는 클라우드 플랫폼에서만 작동한다.

```
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: LoadBalancer
  ports:
  - targetPort: 80
    port: 80
    nodePort:30008
```


### 25.6 타겟포트 알아보는법
```
kubectl describe svc <서비스이름>
```





---
## 26. Namespace
- 네임스페이스 지정해서 get pod할때
```
kubectl get pods --namespace=kube-system
```

- 네임스페이스 지정해서 create하고싶을때
```
kubectl create -f <yaml파일명>.yaml --namespace=dev
```


- 그냥 yaml파일에 넣어주고 싶을때는 metadata안에 넣어줌

```
apiVersion: v1
kind: Pod
metadata: 
  name: myapp-pod
  namespace: dev
  labels:
    app: myapp
     type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

### 26.1 네임스페이스를 새로 만들고싶을때
1) yaml로 생성

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
```
kubectl create -f <파일명>.yaml
```

2) kubectl 이용
```
kubectl create namespace <ns이름>
```

### 26.2 모든 네임스페이스의 파드가 보고싶을때
```
kubectl get pods --all-namespaces
```

### 26.3 네임스페이스에 리소스를 제한하고싶을떄
- Resource Quota를 이용한다.
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
```
kubectl create -f <파일명>.yaml
```




---
## 27. Imperative(명령형) VS Declarative(선언형)

---
## 28.  kubectl apply는 어떻게 동작할까?



---


# 여기서 부터 스케쥴링 섹션


## 29. 매뉴얼 스케쥴링
- 노드를 수동으로 스케쥴링하는걸 배울거임
- 스케쥴러는 어떻게 작동할까?
  - 사실 모든 pod yaml에는 nodeName항목(spec의 자식)이 있는데 기본값이 아님
  - k8s가 자동으로 추가함 (스케쥴링 알고리즘을 통해)
- 스케쥴러가 없다면? pod가 계속 팬딩상태에 머물게됨
- 그럼 그냥 수동으로 node에 pod를 할당하면 된다.
  1) 방법 1 : 생성할때 yaml에 nodeName를 지정해줌
  2) 방법 2 : 만약에 이미 띄워져 있는 파드에 node를 지정해주려면 바인딩 개체를 생성하고 포드의 바인딩 api에 요청한다. (스케쥴러가하는거임)



---

## 30. Label & Selector
- 레이블을 이용해 이것저것 분류함
- 그리고 셀렉터를 사용해서 레이블필터링을 할수있음
- 예를들어
```
kubectl get pods --selector app=App1
```




---

## 31. 테인트 & 톨러레이션

- 노드와 pod의 관계를 알아봄, 그리고 노드에 어떤 포드를 배치할지, 어떻게 제한할 수 있는지 알아봄
- 벌레가 사람한테 다가오는걸 막기 위해 스프레이를 뿌리는걸  생각하자
- 하지만 이 냄새를 잘 버티는 벌레가 있을수 있음
- 벌레가 사람몸에 앉는걸 결정하는건 두가지임
  1) 사람의 테인트
  2) 벌러의 톨러레이션 레벨
- 여기서 사람은 노드, 벌레는 pod임
- 테인트와 톨러레이션은 노드에 파드를 스케쥴링 할때 쓰임
  -.
- 1~3노드가 있고 a~d파드가 있는데 노드1에 전용리소스가 있다고 생각해보자
- 그러면 1번 노드에 테인트를 걸어둔다(스프레이 뿌린다고 생각하자)
- 그럼 a~d파드 모두 노드1에 올라갈 수 없다.
- 이때 특정 파드만 띄우게 하려면 어떻게 함?
- 만약 D를 1에 놓고싶다면 D에다가 수용력을 높이는 행동을 취함
- 그럼 D에는 1에 올라갈수 있는 톨러레이션이 생김
- 그럼 D만 1에 올라가고 나머지는 2-3에 알아서 올라감
- 그런데!!!! D를 무조건 1에 올리겠다는게 아니다. 1에는 D만 올라올수 있다는 말이다.
- D는 2,3에 들어갈 수있다.
- D를 1번에만 넣을수 있게하는건 Node Affinity라고 한다. 다른개념이다.

### 31.1 이걸 하는 방법은 어떻게됨?
- 노드를 테인트 하려면
```
kubectl taint nodes <노드이름> key=val:taint-effect
```

### 31.2 taint-effect
- 테인트를 못참았을때 어떤일이 생기는지 정해줌
- 세가지가 있다
  1) NoSchedule : 이 노드에 그 파드를 올리지 않음
  2) PreferNoSchdule : 파드를 올리지 않는걸 선호하는데 어떻게 될지 모름
  3) NoExecute : ?? 잘 이해를 못함 나중에 알아보자

- 예
```
kubectl taint nodes node1 app=blue:NoSchdule
```


### 31.3 파드에 톨러레이션 추가
- 만약 아래와 같이 테인트를 생성했다면
```
kubectl taint nodes node1 app=blue:NoSchdule
```
- POD에 톨러레이션은 아래와 같이 넣어준다.
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers: 
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

### 31.4 마스터노드도 사실은 노드다!
- 근데 왜 스케쥴러는 어느 파드도 마스터 노드에 스케줄링 하지 않을까?
- k8s가 처음 설정될떄 마스터노드에 테인트 설정이 자동으로 되기 때문! 


### 31.5 테인트 삭제
- controlplane의 
- Taints:             node-role.kubernetes.io/control-plane:NoSchedule
- 를 삭제해보자

```
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```


---
## 32. Node Selectors
- 노드가 서로 다른 크기 일수있다.
- 특정노드에서만 작동할 수 있도록 파드의 한계를 설정해야한다.
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers: 
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large #이거 Large는 노드에 레이블로 size=Large이렇게 붙여진거임
```

### 32.1 여기서 잠깐! 레이블은 어떻게 붙일까?
```
kubectl label nodes <노드이름> <레이블키>=<레이블벨류>
```

### 32.2 노드셀렉터의 한계
- 복잡한 요구사항이라면 좀 어렵다.
- 예를들어 Large or Medium을 선택해라 도 안되고
- Not Small을 선택해라 도 안된다.



---
## 33. Node Affinity
- 파드가 특정 노드에 호스트 될 수 있도록 하는것임
- 노드 셀렉터로 이걸 해보려 했지만
- 노드 셀렉터는 복잡한 조건을 못 건다.
- 노드 어피니티는 특정 노드에 파드 배치를 제한하는 고급 기능을 제공한다.

- 헐 근데 개복잡함. 따라 적기도 벅참
- 특히 Node Affinity Type이 엄청 긺
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers: 
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: Exists
```
- 다른 샘플(Lab에 나온거임)
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: color
                operator: In
                values:
                - blue
```

### 33.1 Node Affinity Type
- 겁나 길다
- 예를 들어 'requiredDuringSchedulingIgnoredDuringExecution'



---
## 34. Node Affinity VS 테인트 & 톨러레이션
- 상황설명 : 
- 노드5개 (빨노파 회회) 있고
- 파드 5개 있따 (빨노파 회회)있다
- 각각에 색깔에 맞추어 할당되어야 한다.

### 34.1 이걸 테인트&톨러레이션으로 해결해보자
1) 노드에 빨노파 표시를 한다. (회색은 Other라고 생각한다.)
2) 그다음에 파드에 톨러레이션을 빨노파 해준다.
3) 그런데 이건 꼭 같은 색깔에 들어간다는 보장이 없다.
4) (내생각: 그럼 회색에도 테인트&톨러레이션 걸어주면 되는거 아닌가??)

## 34.2 노드 어피니티로 해결해보자
1) 노드에 레이블을 붙인다. 빨노파
2) 그다음에 파드에 노드셀렉터를 붙인다.
3) 이경우 빨노파 파드가 각각의 노드에 잘 붙는다.
4) 그런데 회색(Other)파드가 빨노파 노드에 붙을수가있다.


## 34.3 이제 테인트&톨러레이션 & 노드 어피니티 모두를 이용하여 해결해보자
1) 처음엔 빨노파 노드 표시를 한다(테인트)
2) 빨노파 파드에 톨러레이션을 걸어준다.
3) 그다음에 노드 어피니티로 노드가 다른데 가는걸 막아준다고 한다. 



---
## 35. Resource Requirement and Limit
- 각각의 파드는 필요한 양의 cpu와 mem이 이있고 이를 kube-scheduler가 보고 어느 노드에 들어갈지 결정한다.
- 이때 스케쥴러는 필요한 리소스 양을 고려한다. 
- 아래와 같이 우너하는 리소스를 명시해줄 수있다.
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers: 
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - cotainerPort: 8080
    resources:
      requests:
        memory: "4Gi"
        cpu: 2
```

### 35.1 CPU 1개는 무슨 뜻일까? 정녕 무슨 단위인걸까?
- cpu의 제일 작은 단위는 1m이다 (밀리)
- 1카운트는 vCPU 하나와 같다.
- 메모리는 1G, 1M, 1K, 1Gi, 1Mi, 1Ki 등등의 단위로 적으면된다. (Ki는 1024, K는 걍 1000)


### 35.2 Limit
- 지금까지는 최소요구사항이고 이 파드가 사용하는 Limit를 정해줄수도있다.
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers: 
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - cotainerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "4Gi"
        cpu: 2
``` 

### 35.3 파드가 limit보다 자원을 많이 쓰려한다면 어떻게 될까?
- cpu의 경우 시스템이 cpu를 조절해 지정된 한도를 넘지 않도록 한다.
- 하지만 메모리는 그렇지 않다. 한도보다 많은 메모리리소스를 쓸 수 있다. 그러면 파드는 종료된다. 그리고 out of memory를 출력함

### 36.4 알아두자
- defulat는 limit을 안걸어 주는것이기 떄문에 cpu랑 메모리를 많이써서 다른 파드를 질식 시킬수있다.
- 만약에 requests는 안정하고 limit만 정하면 k8s는 자동적으로 limit값으로 할당한다.
- 사실 limit은 안정하는게 좋을수 있다. 그럴려면 모든 포드에 대한 request를 정해줘야 안전하다

### 36.5 LimitRange
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 500m
    max:
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```
```
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

### 36.6 Resource Quotas
- 네임스페이스 레벨로 리퀘스트와 리밋을 정해줄수있다.



---
## 37. 데몬셋
- 각 노드에 하나씩 파드를 띄울때 좋다.
- 노드가 추가되면 알아서 추가되고
- 노드가 삭제되면 알아서 삭제된다.
- kube-proxy같은걸 데몬셋으로 배포하면좋다

### 37.1 데몬셋 만드는 법
- 레플리카 셋 만들때랑 비슷하다
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-deamon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
    template:
      metadata:
        labels:
          app: monitoring-agent
      spec:
        containers:
        - name: monitoring-agent
          image: monitoring-agent
```



### 37.2 데몬셋은 어떻게 작동하는가? (v1.12 까지)
- 각 파드가 생성되기 전에 nodeName으로 각 노드 이름을 정해준다.
- 생성되면 각각의 노드에 할당된다.

### 37.3 그 이후에는 어떻게 작동하는가?
- 1.12부터는 노드어피니티와 디폴트 스케쥴러를 이용한다.



---
## 40. Static Pods
- 큐블렛은 컨트롤 플레인에게 명령을 받아 행동하는데
- 컨트롤플레인이 없어지면 어떻게 될까?

### 40.1 컨트롤플레인 없이 파드를 생성하는법(Static PODs)
- 파드 생성 yaml을
- 원래 컨트롤 플레인이 yaml파일을 저장하는곳에 수동으로 저장시킨다
- /etc/kubernetes/manifests : 이거 --pod-manifest-path 옵션에 저장되어있는 값이라함 (kubelet.service)
- 그럼 큐블렛이 이 파일을 주기적으로 확인하고 파드를 생성한다.
- 생성뿐만아니라 파드의 생존도 보장함
- 파일을 삭제하면 파드도 삭제됨
- 우린 이걸 static pod라고 부르기로 했어요
- 이방법으로는 pod만 생성할수 있음 다른건 생성못함
- 스태틱 파드는 파드 명에 -<노드명>이 붙는다!!!!!!!!!!!!!!!!!

### 40.1.1 static pod의 yaml파일이 저장되는곳을 알고싶다.
1) 아래 명령어를 쳤을때 나오는 yaml 경로가 있다.
```
ps -aux | grep /usr/bin/kubelet
 ```
2) 그 경로의 파일을 살펴본다
```
cat /var/lib/kubelet/config.yaml
```
3) 그럼 staticPodPath 항목에 적혀있다.


### 40.2 Static PODs는 컨트롤 플레인이 인지할수있는가?
- (이제 컨트롤 플레인이 없다는 가정을 제외함)
- 그렇다. 스태틱 파드를 만들어도 kubectl get pod하면 뜬다
- 근데 읽을수만 있다.
- 수정하거나 삭제하려면 manifests 폴더에 있는 yaml파일을 수정해야한다.


### 40.3 스태틱 파드 왜씀?
- kube-system같은 컨트롤플레인 관련된걸 꾸리기 위해한다는듯
- kube-system 네임스페이스로 띄워진 파드중에서도, 아래 명령어 날려서 name끝에 `-controlplane`이 붙은게 진짜 스태틱 파드인듯
```
kubectl get pod --all-namespaces -o wide
```




### 40.4 스태틱 파드 vs 데몬셋

스태틱 파드                                               데몬셋
큐블렛이 생성함                                        kube-api서버가 생성함(DaemonSet Controller)
컨트롤플레인의 구성요소를 배포함           모니터링 에이전트, 로깅 에이전트 등등을 배포함
둘다 :::::: kube-scheduler랑 관련없음 무시당함







---



## 41.  멀티플 스케쥴러

- 디폴트 스케쥴러 이외의 스케쥴러를 여러개 둘수 있다.








----
## 42. 스케쥴러 프로파일

- 스케쥴링 되는 순서
  1) 스케쥴링되어야하는 파드들이 스케쥴링 큐에 들어간다.
  2) 필터링 한다 (요청 스펙과 노드에 남아있는 공간등을 보고)
  3) 스코어링을 한다.
  4) 스코어링은 파드를 배치하고 난다음에 남은 공간이 얼마나 되는지를 기준으로 한다.
  5) 바인딩힌다. 가장 높은 점수를 가진 노드에 바인딩

- 이 모든 작업은 플러그인으로 이루어짐






---

# 여기부터 로깅 & 모니터링

## 43. 모니터 클러스터 컴포넌트
- 어캐 모니터링할까?
- 그럼 뭘 모니터링할까?
- 각각의 파드 개수 파드 성능, cpu등등 의 매트릭을 모니터링한다
- 사실 k8s에는 이런 기능이 없다.
- 하지만 오픈소스 솔루션이 많다. 프로메타우스, 일레스틱 스택, 데이터도그, 다이나트래이스 등등



### 43.1 메트릭서버
- 메트릭서버는 k8s클러스터당 하나가 있다.
- 노드와 포드에서 메트릭틀 회수한다. 그런다음에 저장한다.
- 매트릭 서버는 인메모리 솔루션임. 디스크에 데이터를 저장하지않음

- 큐블렛에는 cAdvisor라는 것이 있는데 이게 파드에서 메트링을 회수하고 매트릭서버한테 줌


### 43.2 설치
1) 깃 클론하고 (url을 못적었다...젠장)
2) 받은 디렉토리에 들어가서
```
kubectl apply -f .
```



### 43.2 상태 확인하는법
- 노드
```
kubectl top node
```
- 파드
```
kubectl top pod
```

## 44. Application Logs
- 파드 로그 찍기
```
kubectl logs -f <파드명>
```
- 만약 하나의 파드에 여러개의 이미지 컨테이너가 있다면
```
kubectl logs -f <파드명> <컨테이너명>
```



---
# 여기서부터 Application 라이프사이클 섹션

## 45.

### 45.1 롤아웃 커맨드
```
kubectl rollout status deployment/myapp-deploy
```

- 배포 취소
```
kubectl rollout undo deployment/myapp-deployment
```

### 45.1 배포 전략
1) 롤링 업데이트
  - 디폴트 배포전략
  - 새로뜨는거는 새 레플리카셋에 뜨는거임
  -  롤백하면 원래 있던 레플리카 셋을 씀(업데이트 하기 전부터 쓰던거)
2)




---
## 시험팁 2
- 있는 deployment수정하기
```
kubectl edit deployment <디플로이먼트명>
```

- 파드 수정하려고 
```
kubectl edit po <파드명>
```
한다음에 저장하고 나오니까 에러날때

- 아래와 같이 강제로 바꾸자
```
kubectl replace --force -f /tmp/kubectl-edit-123091823.yaml
```



---

## 46.  커맨드
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
    - sleep
    - "5000"

```


```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["pyhton","app.py"]
    args: ["--color", "pink"]
```

- command와 args가 좀 헷갈리는데 나중에 알아보자
- command는 Dockerfile의 ENTRYPOINT와,
- args는 Dockerfile의 CMD와 대응한다.


---

## 47. k8s 환경변수

### 47.1 Plain Key Value
```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    ports:
    - containerPort: 8080
    env:   ###----------------------------이게 환경변수 부분임
    - name: APP_COLOR
      value: pink
```

### 47.2 ConfigMap을 이용하는 경우

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    ports:
      - containerPort: 8080
    envFrom:   ###----------------------------이게 환경변수 부분임
      - configMapRef:
          name: <configmap 이름>
```
- 근데 이거 Lab에서는 잘 안먹히고 다른방식으로 헀다. 나중에 랩 찾아보자

### 47.3 SecretKey를 이용하는 경우

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    ports:
    - containerPort: 8080
    envFrom:   ###----------------------------이게 환경변수 부분임
    - secretRef:
        name: <시크릿이름>


```

### 47.4 configMap
- 환경변수가 늘어나면 관리하기 힘들다

1) 컨피그맵을 만들고
2) 파드에 주입한다.


### 47.5 configMap만들기

#### 1) 명령어로 만들기
```
kubectl create configmap \
<컨피그 이름> --from-literal=<key>=<value>


```
- 예시
```
kubectl create configmap \
app-config --from-literal=APP_COLOR=blue
```


####  2) 명령어로 만드는데 파일옵션이용하기
```
kubectl create configmap \
<컨피그 이름> --from-file=<path-to-file>
```


#### 3) 선언적으로 만들기
- yaml파일 만들고
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```
- 만들기 요청
```
kubectl create -f <파일명>
```




### 47.5 Secret
- 컨피그맵에 DB의 endpoint와 passwd같을걸 저장하는건 좋지 않다.
- 시크릿은 암호화 한다. 그거 빼고는 configmap이랑 비슷하다고 보면됨

- 이것도 configmap이랑 비슷하게 명령형으로 만드는게 있고 선언형으로 만드는게 있다.
- 명령형은 귀찮으니 선언형만 메모한다.

#### 1) 선언적으로 만들기
- yaml파일

```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: passwd
```

- 근데 이렇게 yaml파일을 만들면 결국 정보가 노출되니까
- 값들을 웬만하면 인코딩해야한다.


- 리눅스 명령어를 활용해서 인코딩한 값을 넣자
```
echo -n '<나의 패스워드>'  | base64
```


- 적용
```
kubectl apply -f 000.yaml
```

- 정보 추출
```
kubect get secret <시크릿이름> -o yaml
```


- 리눅스 명령어를 활용해서 인코딩한 값 디코딩하자
```
echo -n '<인코딩값>'  | base64 --decode
```

- ***중요*** 시크릿은 암호화된게 아니라 인코딩된거다 남들이 볼수있다는점을 알아야한다.
- 깃헙에 푸시할때 조심하자
- 시크릿은 etcd에 암호화되어있지 않다
- 다른 네임스페이스에서도 access가능하다 (즉 다른팀원이 볼수도있다) 그래서 롤기반엑세스컨트롤(RBAC)를 구성해야한다.
- 그냥 aws나 다른 프로바이더의 서비스를 이용하는게 더 좋다.



### 47.6 secret관련 명령어
- secret확인
```
kubectl get secrets
```






# 로그보기
```
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```


---

## 48. Init Container
- https://codecollector.tistory.com/1281
- pod의 app container들이 실행되기 전에 실행되는 특수한 container
- app image는 없는 utility, 설정 script등을 포함할 수 있습니다.


### 48.1  일반 container와 차이점
- 초기화 컨테이너는 앱 컨테이너의 리소스 상한(limit), 볼륨, 보안 세팅을 포함한 모든 필드와 기능을 지원합니다. 
- 그러나, 초기화 컨테이너를 위한 리소스 요청량과 상한은 리소스에 문서화된 것처럼 다르게 처리됩니다.
- lifecycle, livenessProbe, readinessProbe 또는 startupProbe 를 지원하지 않습니다. 
- 왜냐하면 초기화 컨테이너는 파드가 준비 상태가 되기 전에 완료를 목표로 실행되어야 하기 때문입니다.
- 만약 다수의 초기화 컨테이너가 파드에 지정되어 있다면, kubelet은 해당 초기화 컨테이너들을 한 번에 하나씩 실행합니다. 각 초기화 컨테이너는 다음 컨테이너를 실행하기 전에 꼭 성공해야 합니다. 모든 초기화 컨테이너들이 실행 완료되었을 때, kubelet은 파드의 애플리케이션 컨테이너들을 초기화하고 평소와 같이 실행합니다


```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
    name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    ports:
    - containerPort: 8080
  initContainers:
  - command:
    - sh
    - -c
    - sleep 600
    image: busybox:1.28
    name: warm-up-1
```



---


# 클러스터 Maintenance
- 어떻게 운영체제를 업그레이드 할지 이때 노드를 잃으면 어떤 결과가 나오는지 알아보고
- 클러스터 업그레이드 과정을 알아본다.
- 백업과 재난복구 시나리오를 연습해본다.


## 49. os 업그레이드
- 노드가 클러스터에서 벗어나면 그 노드의 pod에 접근이 불가능해진다. 
- 바로 복귀하면 약간의 해프닝으로 끝남
- 그러나 5분동안 벗어나면 해당노드에서 파드가 종료된다.
- 그럼 레플리카셋에 묶여있는 파드는 노드에 파드가 띄워짐, (레플리카 셋에 없었던 파드는 그냥 죽어있음)
- 이때 다시 그 노드가 돌아오면 그냥 빈 노드가 됨
- 레플리카 셋에 없었던 파드가 증발해버리는거임!

### 그래서 모든 작업의 노드를 의도적으로 드레인 할수 있음
- 아래와 같이 하면 node-1의 파드들이 다른 노드로 이동함
```
kubectl drain node-1
```
- 데몬셋 무시할때
```
kubectl drain node01 --ignore-daemonsets
```
- 그러면 이때 os업그레이드를 한다
- 다 하고 클러스터로 돌아왔을때 uncordon해야한다
```
kubectl uncordon node-1
```

- 이거 말고 cordon이라는 명령어도 있다.
- 이건 기존 파드들을 내쫒지는 않고 스케쥴링에만 제거한다.


---
## 50. k8s version 업그레이드
- kube-apiserver, controller-manager, kube-scheduler, kubelet, kube-proxy, kubectl등 각각의 버전이 따로 있다.
- 구성요소들은 각각 다른 버전일 수도있다.
- 단 kube-apiserver는 짱짱맨이라 다른것들의 버전이 kube-apiserver버전 보다 높을 수는 없다.

- kube-apiserver가 버전 x라면
- 한단계 아래인 controller-manager와 kube-scheduler는 x-1버전까지 할수있고
- 또 한단계 아래인 kubelet과 kube-proxy는 x-2버전까지 할 수 있다. 
- 하지만 kubectl은 x+1부터 x-1까지 된다.


#### 버전은 한꺼번에 올리는것보다 하나씩 올리는걸 권장한다.

## 51. 업그레이드 과정
1) 마스터 노드를 업그레이드 하고 워커노드를 업그레이드한다.   
마스터노드가 업그레이드 되는동안 api-server, 스케쥴러같은건 잠깐 다운된다.   
이때 노드에 떠있는 파드들은 원래대로 서비스 된다.
2) 마스터노드가 업그레이드 완료되면 워커노드랑 마스터노드는 버전이 한개 차이난다.   
이건 지원되는 구성임
3) 이제 워커노드를 업그레이드 할차례
4) 몇가지 전략이 있다. 첫번째 : 한번에 업그레이드 - 이건 서비스가 중단됨
5) 두번째 : 한번에 노드 하나씩 
6) 세번째 : 새 버전으로 새 노드를 추가해서 옮김


## 52. kubeadm 으로 업그레이드 해보자
- k8s공식 홈피 가는게 더 정확하다. 아래꺼 잘안됐었다...
- 아래 명령어 치면 많은 정보를 얻을 수 있음
```
kubeadm upgrade plan
```
- 현재 클러스터 버전, 현재 컴포넌트 버전, 업그레이드 할수 있는 latest stable 버전 등등을 알 수 있다.
- 참고: 큐블렛은 kubeadm이 업그레이드 해주지 않는다. 나중에 수동으로 업그레이드 해야함
- 그리고 아래와같은 커맨드를 알려줌
```
kubeadm upgrade apply v1.13.4
```

1) 일단 kubeadm을 현재 클러스터보다 딱 한 마이너 버전 높은걸로 바꿔준다.
```
apt-get upgrade -y kubeadm=1.12.0-00
```
2) 업그레이드 한다.
```
kubeadm upgrade apply v1.12.0
```
3) 이때 kubectl get node했을때 업그레이드 안된것처럼 보일수도있는데   
그건 큐블렛 버전이 안올라가서 그런것이다. (컨트롤플레인에 큐블렛이 있을수도있고 없을수도있긴함)

4) 큐블렛을 업그레이드 한다.
```
apt-get upgrade -y kubelet=1.12.0-00
```

5) 큐블렛 리스타트
```
systemctl restart kubelet
```

6) 이제 마스터노드의 버전이 올라간것으로 보일것이다.

7) 이제 워커노드를 업그레이드 해줘야한다.
8) drain으로 하나의 노드를 비워준다
```
kubectl drain node-1
```
9) 마스터 노드에서 했던것처럼 kubeadm 업그레이드 해줌
```
apt-get upgrade -y kubeadm=1.12.0-00
```
11) 큐블렛 업그레이드
```
apt-get upgrade -y kubelet=1.12.0-00
```


12) 노드 config 업그레이드 한다.
```
kubeadm upgrade node config --kubelet-version v1.12.0
```

13) 큐블렛 리스타트
```
systemctl restart kubelet
```
14) 스케쥴러에 다시 등록
```
kubectl uncordon node-1
```

15) 남은 노드에도 하나씩 수행함




## 54. 백업

### 54.1 대충 생각나는 백업해줘야하는것들
1) 리소스 컨피그 파일
2) etcd 클러스터
3) Persistent Volumes


  


