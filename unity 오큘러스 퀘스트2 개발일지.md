# unity 오큘러스 퀘스트2 개발일지

### 개발 설정 & 씬 실행하기

우선 이 분 게시글을 참고하기

https://yoonstone-games.tistory.com/102

빌드를 하지 않고 씬 실행 하는 방법을 설명하겠습니다.

1. 빌드세팅에서 안드로이드로 바꿨던 것을 다시 pc로 변경한다.
2. 컴퓨터에 USB로 오큘러스 연결
3. 접근 허용
4. 오큘러스에서 설정을 들어가서 oculus link가 떠야 함
5. 떴다면 oculus link 클릭해서 들어가면 하얀색 배경에 몇 가지 창이 생성된다.
6. unity로 돌아와서 Project 창에서 OVRPlayerController을 검색하여 프리팹을 Hierarchy 창에 갖다 놓고 main 카메라를 삭제한다.
7. play를 누르면 정상 작동한다.

### 키 입력

Unity Documentation 참고

https://docs.unity3d.com/kr/2018.4/Manual/OculusControllers.html

> OVRPlayerController 프리팹을 생성했다면 안에 OVRPlayerController 스크립트가 들어있는 것을 확인 할 수 있다. 이 부분이 움직임을 담당한다.

처음에는 왼쪽 컨트롤러로 회전을 시키고 싶어서 키 값을 알아보려 했다. Input.GetKeyDown()하려고 했다가 포기. OVRPlayerController 스크립트를 열어봤다.

OVRInput.Get()을 통해서 입력을 받는다는 것을 알게 되었다. OVRInput.Get(OVRInput.Button.PrimaryThumbstickLeft)을 이용했고 왼쪽 컨트롤러로 회전을 시킬 수 있게 되었다.

