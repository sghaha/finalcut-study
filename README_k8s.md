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







































  


