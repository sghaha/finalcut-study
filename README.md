# finalcut-study
파이널컷 공부 기록


## * 모든 작업파일은 외장하드에 넣자

## * file(파일) - new(신규) - library(보관함) 
라이브러리를 만들기 전에는 아무것도 할수 없다.   
이거 만들때도 외장하드에 하자
 

## * import media
파일을 프로그램 안으로 불러옴.   
임포트를 하기 직전에 파일 클릭하고 오른쪽 탭을 보면   
1. 편집에 복사
2. 파일 제자리에 두기

둘중에 하나 선택하게 되어있는데 취향에 따라 갈린다.   
나는 파일 제자리에 두기로 하겠다

트랜스 코드는 : 최적화된 미디어 생성   


이제 import하자


## * New Project
 이름을 정해주고   
 왼쪽 아래 버튼을 눌러주면 상세 세팅이 있다.   
 이것도 세팅하고 만들어 주면    

 타임라인이 만들어 진다. 

## * 타임라인에 영상 파일 넣기
영상 클릭하고 e누르면 타임라인 뒤로 들어간다.

## 자르기
단축키 : b
단축키 : 커맨드+B -> 이건 현재 기준 바로 잘림 

옵션 + [ : 플레이헤드 왼쪽 삭제   
옵션 + ] : 플레이헤드 오른쪽 삭제

## 선택하기 툴
단축키 : a
 

## 스키머
마우스 커서 있는곳을 바로 보여주는 기능   
모드 켜기/끄기 단축키 : s   

이걸 아예 영상을 가져올때 쓰면 유용하다   
위쪽에 있는 영상에서 i-o으로 범위를 지정하고 (R눌러서 레인지 셀렉션 해도된다. R모드에서는 스키머아 알아서 켜진다.  )  
e를 누르면 타임라인 뒤쪽으로 들어간다.

i-o잡힌거 지우는거 : 표시 - 선택범위 지우기 (옵션X) 



## 단춬키

* 컨트롤 +- : 타임라인 확대 축소  
* 쉬프트 + z : 타임라인 크기를 현재 화면에 맞게   
* 쉬프트 + 스크롤 : 타임라인 끌어서 이동    
* 방향키좌우 : 한프레임 이동   
* 쉬프트 + 방향키 : 10프레임 이동   
* 방향키 상하 : 잘려있는 자리로 이동    
* N : 스냅핑 on/off   
* V : View 
* 옵션 누르고 드래그 하면 클립이 복사된다.
* 클립 위로 올리기 : 클립 클릭하고 옵션 + 커맨드 + 위


## 자막

왼쪽위에 T처럼 생긴 아이콘 누르면 된다.   
자막 종류 클릭하고 Q누르면 얹힌다
 
자막변경은 오른쪽 탭 (인스펙터)   


## 드랍 쉐도우
불투명도 95%  
블러 3
디스턴스 10



## 자막 슬그머니 들어가게 하고 싶다
오른쪽 아래에 나비모양 트랜지션

 



## 파일 추출하기
파일 - 공유 - 파일 내보내기
단축키 : 커맨드+E


## 카메라 움직임이 있을때는 움직임 전후로 1초 여유를 주는것이 좋다. 

## 트림 커서에 이것저것 기능이 많다.
트림 커서로 영상 가생이를 늘렸다 줄여보고   
가운데를 움직여보고(드래그)   
옵션 누른채로 움직여 보자(드래그)


## 두줄편집
두줄편집할떄 윗쪽 라인도 마그네틱으로 바꾸고싶을때   
클립 - 스토리 라인 생성 (커맨드 + G)    
지붕이 하나 씌워짐



## 트랜지션
메인 스토리라인에 넣을때는 끝부분 클릭하고 트랜지션 더블클릭하면 들어간다.   
그냥 클립을  클릭하고 트랜지션 더블클릭하면 앞뒤로 들어간다.   
커맨드 + T : 디폴트 트랜지션을 넣는다.

끝부분 클릭헀을때 빨간색이면 트랜지션이 못들어간다는 말이다.    
영상이 아예 끝이라 그럼    


트랜지션 길이 바꾸기 : 클릭하고 컨트롤+D 

* 트랜지션 더블클릭하면 상세 조절 가능함

## 디폴트 트랜지션 바꾸기
트랜지션 오른쪽 클릭하고 기본값으로 설정 클릭


## 자막에도 트랜지션을 줄수있다
* 자막을 옆에서 들어오게 하려면 어떻게 해야하나?
  1) 트랜지션 - 무브먼트 - 슬라이드
  2) 이걸 하면 알아서 왼쪽에서 가운데로 들어온다
* 근데 밀어내기(Push트랜지션을 활용하여) 자막만 올리고싶다
  1) 자막에 컴파운드 클립을 만들자
  2) 더블클릭하면 화면이 전환된다
  3) 타이틀을 클릭하고 커맨드+옵션+위 를 하면 스토리라인 위로 올라간다
  4) 여기다가 푸시 이펙트를 넣는다
  


## 검은 화면 집어넣기 
편집 - 생성기삽입 (옵션 + W )   
커스텀 넣으려면 옵션 + 커맨드 + W





## 이펙트
트랜지션 옆에 아이콘 클릭 하면 여러가지 이펙트가 보인다   
제일 중요한 이펙트는 컬러 이다 나중에 자세히 다룰거임

이펙트 씌우면 오른쪽 탭(인스펙터)에 설정 나옴


클릭한거랑 현재 플레이홀더가 달라서   
효과 적용해도 안보여서 헷깔릴때가 있다   
그럴때 옵션을 누르고 클릭하면 따라간다


크로마키(키잉-키어)


## 마스크
원하는 공간만 투명하게 할수 있다.   
드로우 마스크로 마음대로 그린다음에 여러가지를 할 수 있음

## 이펙트를 옆 클립으로 복사하기
적용되어있는 클립 클릭후 복사하고    
적용하고 싶은 클립 클릭후 붙여넣기하면 뭐 붙여넣고 싶은지 나온다.

## 인스펙터

## 클립 복사하면서 드래그로 끌고오기
옵션 + 드래그

## 크롭
켄번은 크롭 시작점과 끝점을 지정할 수 있다.   
스틸 사진 영상에 많이 넣는다고함


## 크롭해서 머물다가 그 배율로 다른쪽을 보여줄때
트랜스폼을 이용하는것보다 크롭을 이용해서 시점을 옮기는게 더 편할 수있음   
트랜스폼은 회전이나 다른 기능을 쓸때 쓰는경우가 더 좋다.
줌인/줌아웃 같은건 크롭이 더 편함

* 4분할 화면도 크롭이 편함


## 혹시 버벅거릴것같으면
수정 - 모두 랜더링 




## 클립간 효과 복사
커맨드C - 커맨드 쉬프트 V

## 효과 프리셋 저장
카테고리도 만들어주면 좋다.   
이펙트 창에 생성된다.

## adjustment layer
* 타이틀 창에 있는건데 처음가면 없을 거다
* 무료 플러그인을 깔자
```
https://www.rippletraining.com/products/free-plugins/rt-adjustment-layer
```
* adjustment layer에 효과를 줄 수 있다
* 그럼 효과 레이어를 얹히는 느낌으로 할수있다. 


* 그런데 adjustment layer 밑에 있는 모든 클립에 적용된다.
* 그래서 블랜드 모드가 있다
* 적용되면 안되는 클립을 상위로 올리고 인스펙터 - 혼합모드 - 뒤에 를 클릭하면 밑으로 내려가면서 어드저스트 적용안됨



## 글씨에만 뒷 배경이 보이고 나머지는 까맣게 처리
* 타이틀 블랜드 모드에 : 스탠실 광도(stencil luma)
* 근데 깜깜한게 싫다 : 생성기 - 텍스처 - 페이퍼를 맨위에 깔고 블랜드 모드를 behind로 하자





## 키프레임 찍혀있는거 보기
클립 우클릭 - 비디오 애니메이션 보기 (컨트롤 v)



## 속도
커맨드 R

## 속도를 다르게 할건데 너무 이질적으로 속도가 바뀌는걸 방지하기 위해
쉬프트 B를 눌러 속도만 블레이드 해서 속도를 세팅하면 알아서 사이를 보기 좋게 해준다.

## 속도 느리게 할떄 프레임 떄문에 이슈있다
속도 아이콘 클릭 - 비디오 품질 - 광학흐름 으로 해놓으면 조금 나음

## 정지 화면
파일 - 정지 프레임 추가 : 단축키 : 옵션 F
아니면
쉬프트 H 도 가능


## 원하는 시간 입력해서 움직이고 싶을때
컨트롤 p

## psd파일을 불러오고 클립 더블클릭하면 레이어 별로 보임

## 현재 사진 스크린샷
파일 - 공유 - 세이브 커런트 프레임


## 기본자막 단축키
컨트롤T


## 오디오랑 비디오랑 떼어내기
클립 - 오디오분리


## 오디오 키프레임
오디오 파형 부분에
옵션키 누르고 클리하면 키프레임이 찍힌다.   
만약 어느 부분만 소리를 낮추려면
키프레임을 4개찍고 가운데부분을 낮춰야한다.



오 아니면 레인지 셀렉션하고
잡아 내리면 된다.


## 녹음
윈도우 - 레코드 보이스 오버



## 작업 마쳤을때
파일 - 생성된 클립 파일 삭제
(랜더링 한거, 옵티마이즈한거, 프록시 한것 등을 지울수 있따.)

파일 - 보관함 미디어 통합
(이거하면 라이브러리 파일만 들고 편집할 수 있따.)





## 미싱 파일 되었을때
file - relink files(파일다시연결)


## 미디어 임포트 할때
* optimize를 하는게 좋을까 프록시를 하는게 좋을까, 아니면 다 안하는게 좋을까, 다 하는게 좋을까?   
멀티카메라를 사용한다면 프록시가 좋을수도?   
그냥 보통의 경우에는 옵티마이즈드만 선택하자


## 라이브러리 파일에 다 묶는 방법
파일 - 이벤트 미디어 통합

## 인덱스
인덱스 클릭 (타임라인 왼쪽 위)하면 여러 인덱스를 볼수 있고,    
검색도가능해서 같은 효과등을 한꺼번에 볼수 있다.

그래서 같은 효과를 찾아서 모두 선택한다음에 효과를 바꾸면 한꺼번에 바꿀수도있다.  
 

아랫쪽에 title을 누르면 자막만 볼 수도 있다.



## 마커
인덱스 화면 - 태그 탭 - 마커 클릭    
마커 생성 : 타임라인에서 m 누르면 생성됨   
* 마커는 타임라인에 찍히는게 아니라 클립에 찍힌다   
 
마커 더블 클릭 하면 설정 바꿀수 있음(해야할일은 빨간색, 챕터는 주황색으로 표시된다.)   
(근데 챕터는 별로 쓸일이 없다는듯, DVD 챕터용으로 쓴다고 한다.)   
할일을 완료로 바꾸면 초록색으로 변한다


* 음악 박자에 맞춰서 뭔가를 넣을때 마커를 사용하면 유용함
* 마커 사이를 움직이는거 : 컨트롤 +;, 컨트롤 +'

## Roles
파컷에만 잇는 개념임   
* 클립의 역할
* 클립을 임포트 할때 롤 지정해줄수 있음

## 플레이 헤드
* 타임라인 현재 시간 표시해주는걸 플레이 헤드 라고함
* 바로보기 화면(플레이헤드 있는곳) 선택된 클릭이랑 다른경우가 많다.
* 이를 위해 클릭을 선택할때는 옵션+클릭으로 하자   
   
* x : 플레이 헤드가 있는 클립이 선택되고 구간선택된다
* c : 플레이 헤드가 있는 클립이 선택만 된다.


* 바로 뒤 클립선택하기(커맨드도 따라옴) : 커맨드 오른쪽/왼쪽
* 위 클릭 아래 클립 선택하기 : 커맨드 위/아래

* 클립선택하고 [ 누르면 왼쪽 경계 선택, 
* 클립선택하고 ] 누르면 오른쪽 경계 선택

* 원하는 자리를 시간을 입력해서 이동하고 싶을때 : 가운데 시간 클릭하고 시간입력하면됨

## 클립을 교체하고 싶을때
* 영상을 끌어다가 해당 클립에 올려놓으면됨
* 바로 아래꺼랑 엮어서 하면 유용함***


## 클립을 위로 올리고싶은데 밑에는 빈걸로 채우고 싶을떄
* p를 눌러서 포지션으로 바꾸고 클립을 올리자
* 아래로 내릴수도있다

## 색감
파이널컷에는 각자 영상을 이용하여 알아서 색감 맞춰주는게 있다. 나중에 찾아보자

