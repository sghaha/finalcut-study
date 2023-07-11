# TEKTON
- https://tekton.dev/docs/

## 파이프라인
- 크게 아래 4가지로 구성됨
  1) Task
  2) TaskRun
  3) Pipeline
  4) PipelineRun

## Step
- 작업 하나하나라고 생각하면됨

## Task
- Step의 집합

## Pipeline
- Pipeline은 task의 집합. 
- Pipeline 속 task들은 순차적으로 실행
- RunAfter구문을 통해 이전 task가 끝난 뒤 다음 task가 실행되게 할 수 있습니다.



## TaskRun
- 단일 Task를 실행시키는 역할을 합니다.
- Task를 실행시킬 때의 서비스어카운트, 사용할 리소스 정의, task들의 pod설정 등을 할 수 있습니다.

## PipelineRun
- Pipeline을 실행시키는 역할을 합니다.
- Pipeline이 실행될 때 Pipeline 내 task들은 workspace라는 이름의 볼륨을 공유하게 할 수 있습니다.
- TaskRun과 마찬가지로 각 Task가 실행될 때의 서비스어카운트, workspace정의, 파라미터, pod설정 등을 할 수 있습니다.
- PipelineRun을 실행시키면 각 Task에 해당하는 TaskRun을 자동으로 생성시켜 실행하기 때문에 별도로 TaskRun을 정의해주지 않아도 됩니다.


## Workspace
- Workspace는 Task들의 볼륨으로 사용됨
- Task별로 사용할 workspace를 지정할 수 있으며, 
- Task를 실행시키는 TaskRun이나 PipelineRun에서 실제 볼륨을 workspace에 붙여줄 수 있습니다. (PVC또는 EmptyDir)

## 참고이미지
- https://velog.velcdn.com/images%2Fsgwon1996%2Fpost%2Fae62b2db-0384-42e3-b129-bc83f2e99d4b%2FUntitled%20(25).png


## 예시
- 다음과 Pipeline이 있다고 합시다.
  1) task 1. clone git repository
  2) task 2. build repository
<br/>
<br/>
- Task별로 Pod을 따로 생성하고 
- 이 두개의 Pod은 리소스를 공유하고 있어야 합니다.
- 이 공유 스토리지가 Workspace가 됩니다.
<br/>
<br/>
- Workspace는 PVC를 붙여줄 수도 있고 EmptyDir을 붙여줄 수도 있습니다.
  - Task 하나만 있을때는 공유할 필요가 없기때문에 EmptyDir로 붙여줘도 되지만,
  - Task가 여러개이고 위 예시와 같이 선행 task의 output을 가지고 작업을 해야하는 경우 PVC를 붙여주어야 합니다.
<br/>
<br/>
- 그래서 위 예시는 다음과 같은 flow로 진행될 것입니다:
  1) PV-PVC간 Bound
  2) PVC를 Workspace로 사용하겠다고 PipelineRun에 선언 (Task에서도 어떤 workspace를 사용할 건지 지정) 
  3) Task 1번 pod이 workspace(pvc)와 Bind 
  4) 깃 레포를 Workspace에 클론 
  5) Task1번 pod unbound 
  6) Task 2번 pod과 workspace(pvc) Bind 
  7) Workspace에 접근해서 빌드 작업 수행 
  8) Task 2번 pod unbound





## 참고
- https://gruuuuu.github.io/cloud/tekton/
