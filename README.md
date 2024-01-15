동시성 이슈 학습<br>

---

<details>
  <summary>
     1 일차
  </summary>
     - Race Rondition에서 java syncronized의 문제점<br>
      syncronized 메서드는 1개의 프로세스에서 1개의 쓰레드만 접근하도록 허용한다.<br>
      즉 여러 개의 서버에서 한 번에 접근한다면 그 서버의 수만큼 접근이 가능한 것이다.<br>

- Mysql을 이용한 해결
  - Passimistic Lock
    실제로 데이터에 Lock을 걸어 데이터의 정합성을 맞춘다. <br>
    exclusive lock을 걸면 다른 트랜잭션에서는 lcok이 해제될 때 까지 접근할 수 없다.<br>
    데드락이 걸릴 수 있다.
  - Optimistic Lock
    Lock을 사용하지 않고 데이터 버전을 사용하는 방법이다. 데이터가 업데이트 될 때 읽어올 당시의 버전과 비교하여 동일하면 update 한다.<br>
    읽은 버전에서 수정사항이 생겼을 경우에 application에서 다시 조회한 후 업데이트를 실행해야한다.
  - Named Lcok
    이름을 가진 lock을 획득한 후 다른 세션은 lock이 해제될 때 까지 접근할 수 없다.<br>
    트랜잭션이 종료되어도 자동으로 lock이 해제되지 않아 별도의 명령을 수행하거나 선점 시간이 끝나야 한다.<br>
</details>

<details>
  <summary>
     2 일차
  </summary>

- Pessimistic Lock<br>
  -
    - 장점<br>
        충돌이 빈번하게 일어난다면 optimistic Lock 보다 좋은 성능을 발휘한다.<br>
        Lock을 통해 업데이트를 제어하기 때문에 데이터 정합성이 보장된다.<br>
    - 단점<br>
    Lock을 직접 걸기 때문에 성능 저하를 일으킬 수 있다.
    <br><br>
- Optimistic Lock
  - 
    - 장점<br>
      - 별도의 락을 잡지 않으므로 Pessimistic Lock 보다 성능상 이점이 있다.<br>
    - 단점<br>
      - update 실패로 인한 재시도 로직을 개발자가 직접 작성해야 한다.<br>
  
- 충돌이 빈번할 경우 Pessimistic Lock 
- 빈번하지 않을 경우 Optimistic Lock
  <br><br>
- Named Lock
  - 
  - Pessimistic Lock 과  Optimistic Lock이 엔티티에 직접 Lock을 걸었다면 Named Lock은 별도의 공간에 Lock을 걸게 된다.<br>
  - Data Source를 분리해서 사용해야함
    - 같은 데이터 소스를 사용하면 커넥션 풀이 부족해져 다른 서비스에 영향을 끼칠 수 있음.
  - 주로 분산 락을 구현할 때 사용
  - 장점 
    - 타임아웃 구현하기 쉬움
    - 데이터 삽입 정합성을 맞출 때 편리
  - 단점
    - 트랜잭션 종료시 락 해제, 세션 관리에 주의.
    - 구현 방법 복잡함.

- Redis
  - 
  - 레디스를 활용하여 동시성 문제를 활용하는 대표적인 라이브러리 2가지
      - Lettuce
        - 
        - setnx 명령어를 활용하려 분산락 구현
          - set if not exist
            - key와 value를 set 할 때 기존에 값이 없을 때만 set
          - spin lock 방식으로 retry 로직을 개발자가 구현
            - lock을 획득하려는 쓰레드가 락을 사용할 수 있는지 반복적으로 확인하면서 
            락을 획득하는 방식
      - Redisson
        - 
        - 펍섭 기반의 락 구현
          - 채널을 하나 만들고 락을 점유중인 스레드가 <br>
          획득하려는 스레드에게 해제를 알려주고<br>
          안내를 받은 스레드가 락 획득을 시도     
          - 별도의 리트라이 로직 필요 없음

- Lettuce
  - 
  - Mysql을 활용한 Named Lock과 유사하지만 세션관리에 신경을 쓰지 않는다는 점, 
   Redis를 사용한다는 점에서 차이가 있다.<br>
  - 잠점
    - 구현이 간단함
    - spring data redis를 사용하면 lettuce가 기본이기 때문에 별도의 라이브러리를 사용하지 않아도 됨
    - 단점
      - spin lock 방식으로 동시에 많은 쓰레드가 Lock 획득 대기 중이라면 redis에 부하를 줄 수 있다.
        - thread.sleep()을 통해 텀을 줄 수 있음.
- Redisson
  - 
  - Reddison은 자신이 점유하고 있는 Lock을 해지할 때 채널에  획득하려는 채널에 Lock 사용 해제 메세지를 보내 Lock을 사용할 수 있도록 한다.<br>
  - Lock 획득을 대기하던 쓰레드들은 메세지 확인 후 Lock 획득을 시도한다.<br>
  - Reddison은 Lettuce와 다르게 메세지를 수신했을 때 Lock 획득을 시도하기 때문에 Redis의 부하를 줄일 수 있다.<br>
  - 장점
    - pub-sub 기반의 구현이기 때문에 Lettuce와 비교해 부하가 적음
  - 단점
    - 구현이 복잡하고 별도의 라이브러리 필요함
    - 라이브러리에 대한 학습이 별도로 필요함
    <br><br>
- 재시도가 필요하지 않은 lock은 Lettuce 활용
- 재시도가 필요한 경우 Redisson 활용
</details>

MySql과 Redis
- 
- Mysql
  - Mysql을 사용중이라면 별도의 비용없이 사용 가능함
  - 어느정도의 트래픽까지는 문제없이 활용 가능
  - Redis 보다는 성능이 낮음
- Redis
  - 활용중인 Redis가 없다면 별도의 구축비용과 인프라 관리비용이 발생
  - Mysql 보다 성능이 좋음

