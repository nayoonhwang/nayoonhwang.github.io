
# DDA 스터디

## Transaction

트랜잭션은은 여러 개의 읽기와 쓰기를 하나의 논리적인 유닛으로 묶는다.
-> 그룹으로 묶은 오퍼레이션의 부분이 실패하면 전체가 실패한다.

safety guarantee (안정성 보장) => application 입장에서는 좋다. 

=> 이런 걸 보장 : ACID 보장해야함.

**Atomicity(원자성)**
한 오퍼레이션이 더 작은 부분으로 쪼개지지 않는것. 디비에 쓰다가 반만 썼는데 "성공"하면 안되고, "실패"하고 마치 어떤 변화도 없었던 것처럼 유지해야함.

*Consistency*
같은 데이터를 바라볼때 같은 값을 가지는 것. Consistensy를 실시간으로 만족시키는것은 퍼포먼스 적으로 힘들고, 그래서 eventual consistency같은 개념이 등장하는 것.

### *Isolation*

*Serializability*
동시다발적인 요청이 들어오더라고 마치 serial한 요청이 들어온 것처럼 => 어려움.

*Durability*
: 커밋 success시엔 db crush가 나든 hardware fault가 나든 레코드를 잊아뿌면 안된다.

Single Node
- write-ahead log를 유지해서 폴트가 나도 복구할수있도록
Replicated database
- 몇개의 노드들에게 copy가 되야 함.

Multi-object Transactions
- Violating atomicity example
: 메일 보내고, unread += 1 하는 경우 메일보내고 unread올릴라그랬는데 crash 
메일은 보냈으나 상대방의 안읽음 +=1 을 못한경우 => 에러!
해결:  BEGIN TRANSACTION ~ END TRANSACTION

### Isolation
Weak Isolation 을 제공

So, 개발자가 주의해서 사용해야한다.

Dirty Write
: row-level lock을 걸어야댐.
Dirty Read
: 갱신되지않는 데이터를 보이지 않게. 커밋된 데이터만 볼수있게.
Trasaction 이 완료되기전까지는 Previous만 보여주고 Transaction이 끝났을때 current를 보여준다.

snapshot isolation and repeatable read

앨리스 계좌 조회
1번 계좌 잔액 500원 -> 600원
2번 계좌 잔액 500원 -> 400원

할떄 순간적으로 500/400으로 조회되는 상황

일반적안 계좌조회 상황에서는 새로고침하면 문제가 안되지만, backup이나 anayltic 등에서는 큰 문제가 된다.(잘못된 데이터가 반영)

스냅샷 isolation 은 일반적으로 write lock을 사용한다. 

### Implementing snapshot isolation using multi-version objects
history를 들구다닌다.

내 transaction id 보다 작은 값을 가진 history의 값을 가져가면된다.

update = delete + create

1. In progress인 transaction 을 쫙 가져오고, 얘네들에 의한 변화는 쫙 무시.
2. Aborted trancation 에 의한 변화는 무시.
3. 나보다 나중의 transaction 에 변화는 무시.
4. 나보다 먼저지만, commit 되지않은거는 무시한다.

