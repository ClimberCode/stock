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
     - PessimisticLock<br>
      
    - 장점 <br>
      충돌이 빈번하게 일어난다면 optimistic Lock 보다 좋은 성능을 발휘한다.<br>
      Lock을 통해 업데이트를 제어하기 때문에 데이터 정합성이 보장된다.<br>
    - 단점 <br>
      Lock을 직접 걸기 때문에 성능 저하를 일으킬 수 있다.<br>
</details>
