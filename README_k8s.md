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



## 47-1 pod에 secret 얹기
https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#create-a-pod-that-has-access-to-the-secret-data-through-a-volume



---
<br/>
<br/><br/>
<br/>
<br/>




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

### 54.2.0 etcd클러스터 백업하는법
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

### 54.2 etcd 클러스터 버전 확인하는법
```
kubectl -n kube-system logs etcd-controlplane | grep -i 'etcd-version'
```
혹은
```
kubectl -n kube-system describe pod etcd-controlplane | grep Image:
```


### 54.3 etcd 클러스터 접근 url얻는 방
```
kubectl -n kube-system describe pod etcd-controlplane | grep '\--listen-client-urls'
```
  
### 54.4 etcd 서버 certificate file 위치 아는법 
```
kubectl -n kube-system describe pod etcd-controlplane | grep '\--cert-file'
```


### 54.5 ETCD CA Certificate file located
```
kubectl -n kube-system describe pod etcd-controlplane | grep '\--trusted-ca-file'
```

### 54.6 ETCD를 /opt/snapshot-pre-boot.db 에 백업해보자
```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db
```

- 해설
```
--endpoints: Optional Flag, points to the address where ETCD is running (127.0.0.1:2379)
--cacert: Mandatory Flag (Absolute Path to the CA certificate file)
--cert: Mandatory Flag (Absolute Path to the Server certificate file)
--key: Mandatory Flag (Absolute Path to the Key file)
```


### 53.7
- 마스터노드 업데이트 하고 온라인 상태가 되엇는데 다른 어플리케이션에 접근을 못한다고한다. 이유는 뭘까?
- 확인하는법 : 일단 디폴트 네임스파이스의 deployments와 pod를 확인해보자
- 확인해보니 아무것도 없네?
- 리스토어를 안해서 그럼

### 53.8 복원해보자
```
ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
snapshot restore /opt/snapshot-pre-boot.db
```

### 53.9 Next, update the /etc/kubernetes/manifests/etcd.yaml:
- 우리는 controlplane - /var/lib/etcd-from-backup 여기에 백업본을 가지고있다
- 그래서 etcd.yaml을 수정해줘야한다.
- 아래부분을 찾아서 수정하자
- old directory (/var/lib/etcd) to the new directory (/var/lib/etcd-from-backup).
```
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```
- 이 파일이 업데이트되면 ETCD 포드는 /etc/kubernetes/manifests 디렉터리 아래에 배치된 정적 포드이므로 자동으로 다시 생성됩니다.

```


Note 1: As the ETCD pod has changed it will automatically restart, and also kube-controller-manager and kube-scheduler. Wait 1-2 to mins for this pods to restart. You can run the command: watch "crictl ps | grep etcd" to see when the ETCD pod is restarted.

Note 2: If the etcd pod is not getting Ready 1/1, then restart it by kubectl delete pod -n kube-system etcd-controlplane and wait 1 minute.

Note 3: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.



If you do change --data-dir to /var/lib/etcd-from-backup in the ETCD YAML file, make sure that the volumeMounts for etcd-data is updated as well, with the mountPath pointing to /var/lib/etcd-from-backup (THIS COMPLETE STEP IS OPTIONAL AND NEED NOT BE DONE FOR COMPLETING THE RESTORE)
```




---


# 시큐리티

## 60. 보안
- k8s 관리자를 하다보면 인증서와 관련된 이슈에 직면한다.
- 그리고 Authentication에 대해 논의하고
- 마지막으로 네트워크 정책을 논의한다

---
## 61. k8s 보안 primitives
- 패스워드 인증은 사용불가능하다, SSH key기반 인증만 k8s host에 접근가능

### 61.1 kube-apiserver
- 사실 이걸로 모든 작업을 수행할 수 있다.
- 이게 1차 방어선임, api 서버 자체에 대한 액세스를 컨트롤 한다.
- 두가지 결정을 내려야함.
1) 누가 클러스터에 접근할 수 있을지
2) 뭘 할수 있을지

#### 61.1.1. 누가 클러스터에 접근할 수 있을까? :
- 어센티케이션 메커니즘에 의해 정의됨
- apiserver에 접근할 수 있는 다양한 방법이 있음 :
1) files- username&passwd
2) files - username&tokens
3) certificates
4) 외부 인증 프로파이더 (ex. LDAP)
5) Service Accounts

#### 61.1.2. 뭘 할 수있을까
- 뭘 할 수 있는지 정의 해주는 방법
1)  RBAC Authorization
2) ABAC Authorization (Attribute based Access control)
3) Node Authorization
4) Webhook Mode


### 61.2 TLS 인증
- api-server, 스케쥴러, etcd 클러스터 등등 모든 통신에는 TLS 암호화가 적용된다.


---

## 62. Authentication
- k8s는 리눅스처럼 유저를 만들거나 하지 않음
- 외부 파일이나, 외부 인증프로파이더 등을 이용함
- 그러나 서비스 어카운트는 k8s가 관리함 (kubectl create serviceaccount SA1 처럼 만들수 있다는 말, k get serviceaccount로 볼수도있음)


### 62.1 일반 사용자 관점에서 일단 보자
- api-server를 통해 클러스터에 접근
- kube apiserver는 요청을 처리하기 전에 인증을 함

### 62.2 그럼 kube apiserver는 어떻게 인증을 할까?
1) static passwd file
2) static token file
3) certificates
4) 외부 인증 프로토콜

### 62.3 제일 간단한 Static passwd file 부터 이야기 해보자
-  csv파일에 username과 passwd를 저장한다 (passwd, username, ID로 구성 되어있음)
-  그럼다음에 kube-apiserver.service의 옵션으로 `--basix-auth-file=user-details.csv` 을 설정해두고 재시작하면 작동함
-  kubeadm을 써서 구성했다면 yaml파일을 수정하면된다. 파일 수정되면 알아서 재시작한다.
- curl를 이용해 접근하는 법
```
curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:passworkd123"
```
- csv파일 마지막 열에 group도 지정할 수 있다.
- 그러나 권장하지 않는 방법


### 62.4 토큰 파일 방식도 된다.
- 이때의 옵션은 `--token-auth-file=user-token-details.csv`
- curl를 이용해 접근하는 법
```
curl -v -k https://master-node-ip:6443/api/v1/pods -header "Authorization: Bearer LKsdufeANDhdoalkjd"
```
- 이것도 역시 권장하지 않는방법



---


## 63. TLS
- SSL이랑 같은거임, TLS가 더 공식명칭임. 443포트 쓰는그거 ㅇㅇ


## 64. TLS를 이용한 k8s 보안
- Certificate(public key) 확장자 : *.crt, *.pem
- Private Key 확장자 : *.key, *-kye.pem


### 64.1 kube-apiserver 의 TLS (Server Certificates for Servers)
- 이건 https통신을 함 apiserver.crt과 apiserver.key가 있다.

### 64.2 etcd 클러스터의 TLS (Server Certificates for Servers)
- 여기도 etcdserver.crt과 etcdserver.key가 있다.

### 64.3 kubelet의 TLS (Server Certificates for Servers)
- 여기도 kubelet.crt와 kubelet.key가 있다.

### 64.4 kube-apiserver의 클라이언트는 뭐가 있을까
1) admin유저 : admin.crt과 admin.key가 있다.
2) kube-scheduler : scheduler.crt, scheduler.key
3) kube-controller-manager : controller-manager.crt, controller-manager.key
4) kube-proxy : kube-proxy.crt, kube-proxy.key


### 64.5 kube-apisever는 etcd서버와 통신한다.
- 사실 etcd서버와 통신하는 유일한것이 바로 kube-apiserver임
- kube-apiserver는 etcd서버의 클라이언트임
- 그래서 apiserver.crt과 apiserver.key를 클라이언트 입장으로도 씀
- apiserver-etcd-client.crt와 apiserver-etcd-client.key를 생성해서 쓰기도한다.


### 64.6 kubelet server
- 이것도 apiserver가 클라이언트일때가 있다.
- 이것도 마찬가지고 apiserver-kubelet-client.crt와 apiserver-kubelet-client.key가 만들어 질때가 있다.


### 64.7 CA(Certificate Authority)
- 지금까지 말한 모든걸 인증하기 위해 ca.crt와 ca.key가 있다.



---

## 65 인증서 생성
- /etc/kubernetes/manifests/kube-apiserver.service의 옵션을 줘서 생성하거나(이건 그냥 할당만 하는거 아닌가?)
- Kubeadm을 사용함

---
## 66. 기존 클러스터에서 인증서 보는법
- /etc/kubernetes/manifests/kube-apiserver.service 에 *.crt, *.key등등이 할당되어있는걸 볼수있음

### 66.1 /etc/kubernetes/pki/apiserver.crt에 있다는 인증서의 정보를 확인해보자
- 인증서 디코딩, 세부사항 보기
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```


### 67. 인증서 적용된거 로그 보기
- 아래 명령어 입력하면 볼수 있는듯
```
kubectl logs etcd-master
```



### 68. 각종 키파일 저장되어있는 디렉토리
``
/etc/kubernetes/pki/
``


### 69. kube-apiserver의 로그보기
- 아래 명령어 날리면 뜬 컨테이너의 상태를 볼수 있다.
```
crictl ps -a
```

- 로그를 보려면
```
crictl logs <컨테이너id>
```



## 70. 인증서 API(인증서 관리 방법 등)
- admin유저 한명당 인증서 하나, 키 하나 를 들고있다.


### 70.1 새로운 멤버가 들어왔다. 인증서를 발급해줘야한다.
- 그사람에 대한 *.csr과 *.key는 있다고 하자
- CertificateSigningRequest라는걸 생성해야함
1) 일단 아래 명령어 날린걸 복사해놓자 (마지막 숫자 0임)
```
cat <csr파일명>.csr | base64 -w 0
``` 
2) 그리고 yaml파일을 만든다
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: <아까 복사한거>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```
3) kubectl apply -f <만든파일명>.yaml







### 70.2 만들어진 인증서 상태 확인하는방법
```
kubectl get csr
```
- 아마 팬딩상태일거임

### 70.3 csr 승인
```
kubectl certificate approve <아까만든CSR명>
```





### 70.4 만들어진 csr의 yaml을 확인하는법
```
kubectl get csr <csr이름> -o yaml
```



### 70.5 csr 거부 하는법
```
kubectl certificate deny <csr이름>
```

### 70.6 아예 지우는법
```
kubectl delete csr agent-smith
```




---

## 71. kubeConfig
- 발급된 인증파일을 kubeconfig file에 등록할 수 있다.
- $HOME/.kube/config
- 여기에 이런저런 인증파일들이 지정되어있기 때문에 우리가 kubectl쓸때 이런저런 인증 옵션을 안넣을수 있었던것임





### 71.1 kubeconfig 파일 구조
1) clusters 섹션
2) context 섹션
3) user 섹션

### 71.2 kubeconfig 구조 보려면
```
kubectl config view
```


### 71.3 current-context 바꾸는 명령어
```
kubectl config use-context dddd@dkddkd
```

- --kubeconfig옵션 : 디폴트 config파일말고 파일 위치를 지정해줄수있음
```
kubectl config --kubeconfig=/root/my-kube-config use-context research
```





---


## 72. api groups
1) /metrics
2) /healthz
3) /version
4) /api   : Core group (ns, pod, rc, event, endpoind, nodes, pv, 등등등)
5) /apis : names group (/apps, /extensions, /networking.k8s.io 등등등)
6) /logs

### 72.1 api들은 쿠버네티스 공식 문서에 자세히 나온다

### 72.2 리스트 보는법
- 아래처럼 하면 paths 볼수있음
```
curl http://localhost:6443 -k
```
- 근데 사실 이건 안됨, 아래 처럼 인증해야함
```
curl http://localhost:6443 -k --key admin.key --cert admin.crt --cacert ca.crt
```

- 아니면 kubectl proxy 커맨드를 이용해도됨
```
kubectl proxy
```
- 위의 명령어를 날리면 이제 앞으로의 명령은 프록시를 태움
```
curl http://localhost:6443 -k
```
- 이제 이렇게만 날리면 된다.







---

## 73. Authorization

### 73.1 클러스터에 왜 Authorization이 필요한가??
- 우린 관리자로써 kubectl을 이용하여 별의별걸 다 할수 있다
- 이제 다른사람이 클러스터에 액세스 할거다 (여긴 사람이 아니라 모니터링 app등 시스템도 포함된다)
- 우린 모두가 같은 접근 수준을 갖길 원하지 않는다.


### 73.2 k8s가 제공하는 Authorization 메커니즘
1) Node : 
2) ABAC : 
3) RBAC : 
4) Webhook :  
5) AlwaysAllow (이게 디폴트 값임)
6) AlwaysDeny


## 73.3 현재 어떤 메커니즘을 쓰는지 확인하는법
```
kubectl describe pod kube-apiserver-controlplan -n kube-system
```
에서 --authorization-mode를 확인해보자




---

## 74. RBAC (Role Base Access Contorl)
- 룰 만드는 yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]  #파드를 list get create updelete 할수 있음
- apiGroups: [""]
  resources: ["ConfigMa"]
  verbs: ["create"]
``` 

- 룰 바인딩 : 사용자 개체를 역할에 연결
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
name: devuser-developer-bindin
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
  roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

### 74.1 롤보기
```
kubectl get roles
```

### 74.2 롤바인딩 보기
```
kubectl get rolebindings
```

### 74.3 롤 자세히 보기
```
kubectl describe role <role이름>
```

### 74.4 롤바인딩 자세히 보기
```
kubectl describe rolebinding <롤바인딩 이름>
```


### 74.5 내가 특정 커맨드에 대한 권한이 있는지 확인해보는법
```
kubectl auth can-i create deployment
```


### 74.6 특정 사용자가 커맨드에 대한 권한이 있는지 확인해보는법
```
kubectl auth can-i create pod --as dev-user
```

### 74.7 명령형으로 role만들기
```
kubectl create role <룰이름> --namespace=default --verb=list,create,delete --resource=pods
```

### 74.8 명령형으로 RoleBinding만들기
```
kubectl create rolebinding dev-user-binding --namesapce=default --role=developer --user=dev-user
```






---

## 75. 크러스터 롤, 크러스터 롤 바인딩
- 노드를 네임스페이스로 분리할수 있을까??
- 못한다. 클러스터 범위의 리소스 이기 때문이다
- 그래서 이런것들은 Role과 RoleBinding으로 auth를 못한다.
- 클러스터 롤과 클러스터 롤 바인딩으로 한다.
- 기본적으로 롤과 롤바인딩이랑 비슷한 개념이다. 




### 75.1 명령형으로 클러스터 룰 만들기
```
kubectl create clusterrole <룰이름> --verb=get,watch,list,create,delete --resource=nodes
```

```
kubectl create clusterrole storage-admin --verb=get,watch,list,create,delete --resource=persistentvolumes,storageclasses
```


### 75.2 명령형으로 클러스터 룰 바인딩 만들기
```
kubectl create clusterrolebinding <클러스터룰바인딩이름> --clusterrole=node-admin --user=michelle
```
```
kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle
```


---

## 76. 서비스어카운트
- k8s에는 두가지 유형의 계정이 있다.
1) 유저 (ex. 관리자, 개발자)
2) 서비스어카운트 (ex. 프로메테우스, 젠킨스)



### 76.1 서비스어카운트 생성
```
kubectl create serviceaccount <SA이름>
```

### 76.2 서비스어카운트 토큰
- sa가 생성되면 토큰이 자동으로 생성된다.
- describe하면 볼수있다.
```
kubectl describe serviceaccount <sa이름>
```
- 토큰은 k8s api에 인증하는 동안 외부앱이 반드시 사용해야하는 것이다.

### 76.3 토큰 시크린 둘러보기
```
kubectl describe secret <sa토큰명>
```


### 76.4 디폴트 sa
- 모든 네임스페이스에는 defualt라는 sa가 생성된다
- 그리고 만약에 파드가 생성되면 디폴트 sa와 토큰이 볼륨에 마운트 된다
- 파드를 describe를 하면 마운트된 볼륨의 위치와 토큰을 볼수있다.
- 아래 명령어로 토큰을 볼수 있다.
```
kubectl exec -it <파드명> --ls <위치>
```
```
kubectl exec -it <파드명> ls <위치>
```
- ca.crt, namespace, token이라는 파일세개를 볼 수 있다.
- token : 실제 토큰을 가진 파일
```
kubectl exec -it <파드명> cat <토큰파일 주소>
```

### 76.5 파드의 서비스 어카운트 변경
- spec 섹션 하위에 serviceAccountName: <SA명> 을 넣어주면된다.
- 그런데 이건 파드 수정으로 적용이 안된다. 파드를 삭제하고 다시 만들어야 한다.
- 그러나 디플로이먼트로 했다면 알아서 롤아웃이 발생하여되어 삭제할 필요없다.

### 76.6 1.22 1.24에서 변경된점
- 파드가 생성되면 기본 sa 토큰에 의존하지 않고 만료기한이 있는 토큰이 TokenRequestAPI를 통해 생성된다.
- sa가 생성될때 더이상 토큰이 자동으로 생성되지 않는다. 토큰을 만들어줘야한다.





---
## 80. Image

- 우리는 샘플파드를 만들때 nginx 이미지를 자주 썼는데 아래와 같이 항상 yaml에 명시해줬다.
```
image: nginx
```
- 근데 이게 어떻게 가능했던걸까
- nginx는 도커의 이미지 명명규칙을 따르는 image혹은 repository이다
- 사실은
```
image: library/nginx
``` 
- 라고 쓰는게 맞는건데 축약되어있는거다
- 첫부분은 유저 어카운트명인데 아무것도 명시하지 않으면 자동적으로 libarary라고 인식한다.
- library는 도커 공식 이미지가 저장되는 기본계정의 이름
- 근데 이것도 축약되어있는거다
```
image: docker.io/library/nginx
```
- 더 앞에 레지스트리를 넣어줘야하는건데 안넣어서 기본값인 docker.io가 들어간 효과임 ㅇㅇ
- 레지스트리 : 이미지를 저장하는 곳, 새 이미지를 생성하거나 업데이트 할때마다 레지스트리에 푸시 하는거임


---
## 81. Private Image
- 아래처럼 프라이빗 이미지를 yaml에 넣었을때 인증/로그인은 어떻게 구현할까?
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
```

- 일단 docker레지스트리 관련  시크릿을 하나 생성한다. (이건 yaml파일로 뺀다음에 apply하면 안되드라 암호화 관련된거때문인듯)
```
kubectl create secret docker-registry <시크릿명> \
	--docker-server=<private-registry.io> \
	--docker-username=<registry-user> \
	--docker-password=<registry-password> \
	--docker-email=<registry-user@org.com>
```


- 그리고 yaml에 아래와 같이 쓴다.
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: <시크릿이름>
```



---
## 82.0 강제로 바꾸기
```
kubectl replace --force -f /tmp/kubedkdfjf.yaml
```

## 83. Security Context
```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep","3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```

---
## 84. 네트워크 Policy
- 파드간의 트래픽은 기본적으로 all allow임
- 어디어디서 들어오는 몇번 포트만 허용할거다! 이런 정책이 필요함
- 일단 pod에 label을 붙이고 network policy에 podSelector를 붙인다.
- 그다음에 규칙을 만듦
- 아래는 예시
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
      matchLabels:
        name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```



### 84.1 파드 이름과 네임스페이스까지 & 조건으로 ingress를 허용함
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
      matchLabels:
        name: api-pod
      namespaceSelector:
      matchLabels:
        name: pod
    ports:
    - protocol: TCP
      port: 3306
```

### 84.2 파드 이름과 네임스페이스까지 || 조건으로 ingress를 허용함
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
      matchLabels:
        name: api-pod
    - namespaceSelector:
      matchLabels:
        name: pod
    ports:
    - protocol: TCP
      port: 3306
```


### 84.3 파드 이름과 네임스페이스, ip(외부도 가능)까지 || 조건으로 ingress를 허용함
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
      matchLabels:
        name: api-pod
    - namespaceSelector:
      matchLabels:
        name: pod
    - ipBlock:
      cidr: 2.168.0.1/32
    ports:
    - protocol: TCP
      port: 3306
```



### 84.4 egress까지
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
      matchLabels:
        name: api-pod
    - namespaceSelector:
      matchLabels:
        name: pod
    - ipBlock:
      cidr: 2.168.0.1/32
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
      cidr: 2.168.0.1/32
    ports:
    - protocol: TCP
      port: 80 
```


### 84.5 네트워크 폴리시 보기
```
k get networkpolicy --all-namespaces
```











---

## 85. 도커의 Storage
- 시스템에 도커를 설치하면 아래 네군데에 데이터를 저장한다.
1) /var/lib/docker/aufs
2) /var/lib/docker/containers
3) /var/lib/docker/image
4) /var/lib/docker/volumes
- 여기서 데이터란 docker 호스트에서 실행되는 이미지 및 컨테이너와 관련된 파일을 말함

### 85.1 도커 레이어드 아키텍처
- 아래와 같이 Dockerfile이 있다고하자
```
FROM Ubuntu

RUN apt-get update $$ apt-get -y install python

RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

- 이걸 빌드하면 
```
docker build Dockerfile -t sghaha/my-custom-app
```
- 레이어가 쌓인다.
1) 우분투 레이어
2) apt 패키지 변경
3) pip 패키지 변경
4) 소스코드
5) 엔트리포인트 업데이트

- 이걸 빌드헀고 이제 두번째 도커 파일이 있다고 하자
```
FROM Ubuntu

RUN apt-get update $$ apt-get -y install python

RUN pip install flask flask-mysql

COPY app2.py /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app2.py flask run
```
- 이것도 레이어가 쌓인다.
1) 우분투 레이어
2) apt 패키지 변경
3) pip 패키지 변경
4) 소스코드
5) 엔트리포인트 업데이트
- 근데 1번부터 3번까지는 똑같아서 빌드안해도 된다!!!!!!!!!!! 4-5번만 새로 빌드하면 된다.
- 이럼 디스크도 절약하고 시간도 절약함
- 이건 같은 파일을 rebuild할때도 비슷하게 적용되어서 rebuild할때 시간이 절약된다.

- 빌드를 다하고 이제 실행한다고 생각해보자
```
docker run sghaha/my-custom-app
```
- 그러면 6번 레이어가 쌓인다.
- 5번까지의 레이어는 `이미지레이어라`고 부르며 read-only이고
- 6번 레이어는 `컨테이너 레이어`라고 부르며 read/write임
- 앱이 실행되면 6번 레이어에 데이터를 저장할텐데 이건 휘발된다.
- 그래서 volumes이라는곳에 저장함

### 85.2 도커 볼륨
- mysql을 돌릴거고 볼륨을 마운트 해준다고 생각해보자
- 아래와 같이 볼륨을 만든다
```
docker volume crate data_volume
```

- 그럼 `/var/lib/docker/volumes/data_volume`라는 폴더가 만들어진다.
- 그리고 볼륨을 마운트 해서 도커를 실행한다
```
docker run -v data_volume:/var/lib/mysql mysql
```
- 이러면 컨테이너 안에 볼륨이 마운트됨
- 컨테이너가 파괴되어도 데이터는 남아있는다

- 만약 볼륨을 만들지 않고
```
docker run -v data_volume_2222:/var/lib/mysql mysql
```
- 이런식으로 없는 볼륨을 마운트하려고 하면 자동으로 볼륨을 만들어준다.

### 85.3 만약 볼륨 말고 다른걸 마운팅 하고 싶다면?
- 만약 /data/mysql이라는 폴더를 마운팅하고 싶다면
```
docker run -v /data/mysql:/var/lib/mysql mysql
```
- 이런식으로 하면된다. 이걸 바인드 마운팅이라 한다.
- 근데 사실 -v는 예전방법이라고 한다.
- 아래 방법이 더 권장 된다고함
```
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```





## 86. 컨테이너 스토리지 인터페이스
- 과거 k8s는 도커를 런타임 엔진으로만 사용했다.
- 그리고 도커 코드가 k8s에 내장되어있엇다
- 근데 이젠 CRI(Container Runtime Interface)라고라는게 생겨서 이거에만 만족시키면 k8s에 넣을수있다.
- 이거랑 비슷하게 CNI(네트워킹)이게 있어 이거에 맞춰 개발하면 플러그인을 만들수 있음
- 그리고 CSI(컨테이너 스토리지 인터페이스)라는게 있다.



## 87. Volumes
- 이제 k8s 파드에 볼륨을 붙인다.
- 아래는 1에서 100까지 무작위 숫자를 파일에 저장하는 파드이다
```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
```

- 이제 볼륨을 지정해보자
```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```
- 근데 이건 그 노드의 /data 디렉토리를 마운트 시키는거라, 다중 노드 시스템엔 권장되지 않는다.
- 그래서 다른 서비스를 쓰는게좋다
- 아래는 aws ebs를 쓰는 예시임
```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    awsElasticBlockStore:
      volumeID: <EBS 볼륨 ID>
      fsType: ext4
```



--- 

## 88. Persistent Volumes
- persistent volumes이 저장소라고 생각하면될듯
- 각 파드가 공간을 할당받아서 쓴다.
- PV만들기
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath: # 이건 노드의 로컬 디렉토리에서 저장소를 이용하는거라 운영계에는 사용하지 않음
    path: /tmp/data
```
- ebs쓴다고하면
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <EBS 볼륨 ID>
    fsType: ext4
```



## 89. Persistent Volume Claims
- PV를 다른 리소스에 할당 시켜주기 위한것인가봄
- PVC는 무조건 하나의 PV를 가지고있다. 1:1관계임
- 만들어 보자
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mycliam
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
-PVC를 만들면 PV중에 제일 적합한걸 찾아서 1:1 매칭 시켜줌



### 89.1 pod에서 pvc를 사용하는법
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
        - mountPath: "/var/www/html"
          name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

- pvc가 pod에 사용되고 있다면 pod에서 사용하고 있기 때문에 pvc를 지울수 없다.
- pod에 연결되어있는 pvc를 삭제 요청하면 그때는 안지워 지지만 pod가 지워지면 pvc도 같이 지워짐



---

### 90. 스토리지 클래스
- pv를 만들때 우리가 구글클라우드를 쓴다면 구글클라우스 명령어로 계속 볼륨을 만들고 pv를 만들어야한다
- 근데 스토리지 클래스를 만들면 걍 pv가 알아서 만들어진단다






---


# 네트워킹

## 91. 네트워크 관련 명령어
```
ip link
```
```
ip addr
```
```
ip addr add 192.168.1.10/24 dev eth0
```
```
ip route
```
```
ip route add 192.168.1.0/24 via 192.168.2.1
```
```
arp
```
```
netstat -plnt
```


---

## 100. CNI practice
- kubelet의 컨터이너 런타임 엔트포인트 알아보기
```
ps -aux | grep kubelet | grep --color container-runtime-endpoint
```

- CNI 바이너리는 위치 디폴트값은 /opt/cni/bin 이다

- k8s 클러스터가 쓰는 cni플러그인 보려면
```
ls /etc/cni/net.d
```



## 101. ip address management
- pod에 ip를 어캐 할당할까?
- CNI플러그인의 역할임 ㅇㅇ
- CNI에 두개의 플러그인을 내장한다. DHCP / host-local
- /etc/cni/net.d/net-script.conf에 ipam이라는 섹션에 지정된다.

## ip link 명령어
- 네트워크 인터페이스를 표시하고 수정함
- 아래는 브릿지 타입의 디바이스를 출력하라는거임
```
ip link show type bridge
```



---

## 102. service networking

- 서비스를 생성하면 ip가 할당하고 클러스터의 모든 포드에서 엑세스 할수 있다.
- 파드는 노드에 걸쳐 호스트되는 반면 서비스는 클러스터에 걸쳐 호스트된다.
- 이걸 클러스터에만 접근할수 있게 한게 clusterIP
- 외부에도 포트 뚫어주는게 NodePort


### 102.1 서비스 네트워킹 practice
- 서비스에 할당되는 ip 대역 보는법
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
```

- kube-proxy가 어떤 종류의 proxy를 쓰는지 확인하는 방법
```
k logs <kube-proxy 파드 명> -n kube-system | grep -i proxy
```



- kube-proxy는 데몬셋으로 생성된다. (아래 명령어로 보임)
```
k get ds -A
```


---

## 103. k8s의 DNS

- CoreDNS의 config 파일은 어디있을까?
- CoreDNS deploy를 자세히 보면 나옴
```
k describe deploy coredns -n kube-system | grep -A2 Args
```










## 110. kubeadm으로 설치하는 과정
1) VM들이 필요하다 (마스터 노드, 워커 노드)
2) 각각의 VM에 컨테이너 호스트를 설치한다. (like containerd)
3) 모든 노드에 kubeadm을 설치한다.
4) 마스터를 init한다
5) pod network를 설정한다.
6) 워커노드가 마스터 노드에 join한다.



## 111. 자동완성
공식문서에서
`kubectl Cheat Sheet`로 나오는거



## 112. json
$.prizes[?(@.year==2014)].laureates[*.]irstname
$.status.containerStatuses[?(@.name == 'redis-container')].restartCount






## 113. 라이트닝 랩 전체
https://junior-developer.tistory.com/m/97



## 115. 라이트닝 랩 2번문제
https://naa0.tistory.com/344



## 116. etcd backup
```
ETCDCTL_API=3 etcdctl \ 
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/etcd-backup.db
```




## 117. expose
```
kubectl expose pod-- messaging --port=6379 --name messaging-service
```
```
k expose depoly hr-web-app --name=hr-web-app --type NodePort --port 8080
```



## 118.스태틱 파드 만들기
```
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -oyaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```


## 119. 참고 블로그
```
https://daintree.tistory.com/m/13
```

## 120. 
```
k create deploy hr-web-app --image=kodekloud/webapp-color --replicas=2
```


## 121. hostPath로 PV만드는 법
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
      path: /pv/data-analytics
```


## 122. 파드에 emptyDir 엮어주는법
```
https://kubernetes.io/docs/concepts/storage/volumes/#emptydir
```


## 123. mock2 8번
```
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
```

```
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup 10-244-192-4.default.pod.cluster.local > /root/CKA/nginx.pod
```

