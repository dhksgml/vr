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



### 텔레포트

유튜브 ExpanseVR 참고

https://www.youtube.com/watch?v=10xu4exNstQ

텔레포트를 하게 되면 바닥을 뚫고 아래로 떨어지는 상황이 발생합니다.

unity 물리엔진이 60fps로 동작하는데, 오큘러스나 다른 기기들은 새로고침 빈도가 다르기 때문이다.

Project Settings -> Time -> Fixed Timestep을 변경

그리고, LocomotionTeleport.cs 스크립트를 변경해야 한다.

```c#
character.enabled = false;
character.enabled = true;
```

콜라이더 활성화/비활성화를 명시해야 바닥으로 떨어지는 현상을 예방할 수 있다.



### 회전

OVR Player Controller 스크립트의 Rotate Around Guardian Center을 true로 변경하면 플레이어 시점 회전이 현재 위치에서 고개만 돌리는 것 처럼 된다.

### 레이캐스트

변경 전

```c#
public class raycasting : MonoBehaviour
{
    private Ray ray;
    private RaycastHit hit;

    // Update is called once per frame
    void Update()
    {
        Debug.DrawRay(ray.origin, ray.direction * 1000f, Color.red);
        if (OVRInput.GetDown(OVRInput.Button.SecondaryHandTrigger))
        {
            if(Physics.Raycast(transform.position, transform.forward, out hit, 1000f))
            {
                hit.transform.GetComponent<MeshRenderer>().material.color = Color.red;
                
            }
        }    
    }
}

```

변경 후

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class raycasting : MonoBehaviour
{
    public GameObject ray_point;
    private Ray ray;

    // Update is called once per frame
    void Update()
    {
        
        if (OVRInput.GetDown(OVRInput.Button.PrimaryIndexTrigger, OVRInput.Controller.RTouch))
        {
            Debug.DrawRay(ray.origin, ray.direction * 1000f, Color.red);
            FireRay();
        }    
    }

    void FireRay()
    {
        Ray ray = new Ray(ray_point.transform.position, ray_point.transform.forward);
        Debug.DrawRay(ray_point.transform.position, ray_point.transform.forward, Color.red);
        RaycastHit hitData;
        if (Physics.Raycast(ray, out hitData))
        {
            Debug.Log(hitData.transform.gameObject.name);
        }
    }       
}
```

전에 코드는 한쪽 컨트롤러를 내려놓았을 때 작동을 안 하는 현상이 발견되었다. 매핑 방식에 차이가 있어서 그런 것 같았다.

아래 사이트를 참고해서 문제를 해결할 수 있었다.

https://syaring92.tistory.com/2

### 물체 잡기

grabber 스크립트와 grabbable 스크립트를 이용하여 물체를 잡을 수 있다.

손에는 grabber 스크립트를 적용하고, 잡을 수 있는 물체에는 grabbable 스크립트를 적용한다.

> 멀리 있는 물체를 손으로 가져오는 방법

멀리 있는 물체를 레이캐스트를 통해 손으로 끌고 올 수 있다.

DistanceGrabHandLeft와 DistanceGrabHandRight 프리팹를 이용하는 방법이다.

이 프리팹에는 Distance Grabber 스크립트가 있는데, Grip Transform을 손 위치로 지정하고,

Player에는 OVRPlayerController을 넣어준다.

crosshair을 지정하지 않은 경우 끌어올 수 없다. 만약 crosshair을 지정하지 않고 이 기능을 사용하고 싶다면, spherecollider을 하나 만들고 Distance Grabber 스크립트에 있는 Use Spherecast를 체크해야 한다.



경고 발생

1.

m_parentHeldObject incompatible with DistanceGrabber. Setting to false.
UnityEngine.Debug:LogError(Object)
OculusSampleFramework.DistanceGrabber:Start() (at Assets/Oculus/SampleFramework/Core/DistanceGrab/Scripts/DistanceGrabber.cs:95)

2.

NullReferenceException: Object reference not set to an instance of an object
CharacterCameraConstraint.OnEnable () (at Assets/Oculus/SampleFramework/Core/Locomotion/Scripts/CharacterCameraConstraint.cs:72)