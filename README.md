# 유니티 서버 수행

## 1. 커맨드 패턴 & JobQueue

- 커맨드 패턴이란?
  - 하나의 객체를 통해 여러 객체들에 명령(Command)을 해야 할 때 사용되는 패턴
  - 예시로는 서버에서 여러 클라 세션이 요청한 것들을 캡슐화하여 처리하게 된다.
- 코드에서는
    - IJob이라는 인터페이스를 상속받은

    ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/0b5f3290-427c-4818-9b8d-695a9bd188b8)
    - 클래스를 받는다.

    ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/4678b055-784b-4022-b5fb-84d2a05ee0b2)
    - 이 Job클래스는 Action을 받아서 JobSerializer클래스의 Flush함수로 JobQueue에 들어 있는 Job들이 실행이 되고,

    ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/267d7839-cc17-4c85-9a74-05f5c32706c1)
    - Pop함수로 없앤다.
   
    ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/27332a0d-9f54-441d-9c1c-bcef4f7b4d0b)
    
    - 이 부분에서 커맨드 패턴을 느낄 수 있다.
    - 우리가 만든 게임서버에서는 게임 룸에 클라이언트 세션들이 한 일을 보내고, 그것들을 JobQueue에 보관하여 일괄처리하는 순서이다.
    
    
      - 만약 플레이어가 게임에 들어온 상황
        - 클라이언트 세션이 들어왔음을 게임 룸의 JobQueue에 Push한다.
        
        ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/9d60ae4a-5563-44f9-afe0-6ac06b3c0ec3)
        - 여러 일이 쌓은 JobQueue를 비우기 위해 일괄 처리(Flush)한다.
          
        ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/77e38113-2550-4f99-8235-b5fc410867c8)
        - 여기서 JobQueue에 매개변수로 넣은 Action이 작동된다.

        ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/f36c6bc2-808e-4463-8b79-2fb779de0580)
        
- JobQueue 구조
  - JobQueue는 GameRoom에서 Push한 Job을 갖고 있는 데이터 구조이다.
  ![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/3ce37d3d-1204-43d6-9f39-af48608dc03d)
  
## 2. JobTimer

- JobTimer란?
  - 일정 시간이 되면 예약하는 시스템
  - 업데이트를 효율적으로

- JobTimer를 왜 쓰나?
  - JobSerializer에서 JobQueue를 만들었다. 하지만 만약, 일정 시간 이후에 실행이 되어야 하는 것이라면?
  - 그냥 무지성 구현하기에는 귀찮고 서버에 부하가 많이 걸릴 것 같다.. 
  - 그럼 일정 시간 이후에 저절로 JobQueue에 Push하는 기능이 필요할 것이다.
  - 그것이 JobTimer이다.

![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/3d442221-e0ef-42fd-bfe7-603c279840c8)

- JobTimer의 요소를 정의해 놓은 JobTimerElem 구조체이다.
- 해야하는 일(job)과 일정 시간(execTick)이 정의되어 있다.

![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/30cc9660-8410-4dc5-94ea-0640d0728071)

- 0.1초마다 GameRoom의 Update를 실행하여
![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/615e7f1a-104c-43fc-a723-0fd0a7a24886)


- Flush함수를 실행한다.
![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/0dae97a2-b960-44ee-b78c-92157fc65430)

- 그러면 JobTimer의 Flush함수가 실행되게 되는데,
![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/4e5c783e-ca5f-4b9e-86f7-a862d7d9ee03)

- JobTimerElem을 요소로 하고 있는 우선순위 큐.
  - 이러면 JobTimerElem의 execTick이 작은 순대로 큐에 쌓인다.

![image](https://github.com/daehee719/UnityServerSuhang2/assets/81199906/de9859b5-7261-4b45-b513-83f1a7b341d3)

- 그 다음, Flush 함수로 간다. 
- 우선순위 큐에서 최우선순위 요소를 뽑아서 값(execTick)을 본다.
- 만약 execTick이 없다면 바로 실행하라는 뜻이므로 바로 실행을 해준다.
- 만약 execTick이 현재의 TickCount보다 많으면 시간이 지났다는 뜻이므로 바로 실행을 해준다.
이러면 JobTimer의 기능을 하게 된다.

