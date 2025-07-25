## Week9 동시성 제어 부분 간단복습
### Concurrency Control (pp. 165 - 176) 

◦ Serializability 
	**여러 트랜잭션이 동시에 실행**되더라도, 그 **결과는 어떤 순서로든 차례대로 실행한 것과 동일해야 한다**는 원칙
	
◦ Transaction Isolation 
	ACID 원칙 중 I에 해당하는 원칙이다.
	**하나의 트랜잭션이 실행되는 동안 다른 트랜잭션의 영향을 얼마나 차단할 것인가**를 결정하는 속성
	
◦ Read and Write Anomalies 
	격리 수준이 완벽하지 않을 때(직렬성이 깨질 때) 발생하는 데이터 꼬임 현상을 **이상 현상** 이라고 부른다.
	동시성 제어의 주된 목적이 이 이상 현상을 막는 것이다.

◦ Isolation Levels
	DB는 위에서 설명한 이상 현상들을 막기 위해, 개발자가 선택할 수 있는 몇 가지 고립 수준을 제공한다.
	고립 수준이 높아질수록 데이터는 안전해지지만 성능은 저하될 수 있다.

---

# Week10
# Concurrency Control
- 동시성  제어는 여러 트랜잭션이 동시에 실행될 때 발생하는 상호작용을 관리하는 기술들의 집합이다.
- 동시성 제어는 여러 사용자가 동시에 같은 데이터를 수정하려고 할 때, 데이터가 엉망이 되지 않도록 '교통정리'를 해준다.

- 동시성 제어는 크게 세 가지로 나눌 수 있다.
	1. **낙관적 동시성 제어(Optimistic Concurrency Control, OCC)** : 충돌이 거의 일어나지 않을 것이라고 가정하는 방식
	2. **다중 버전 동시성 제어(Multiversion Concurrency Control, MVCC)** : 데이터의 여러 버전을 유지하여 동시성을 높이는 방식.
	3. **비관적 동시성 제어(Pessimistic Concurrency Control, PCC)** : 충돌이 자주 일어날 것이라고 가정하고, 시작부터 데이터를 잠그는 방식.

## 낙관적 동시성 제어 (Optimistic Concurrency Control, OCC)
-> 간단하게 요약하면 '일단 실행하고, 문제가 생기면 그때 해결하자' 이다.

- 트랜잭션간의 충돌이 거의 발생하지 않을거라고 가정해서 다른 트랜잭션들을 막지 않고 자유롭게 작업을 실행하게 둔다.
- 그 후 마지막에 트랜잭션을 커밋하기 전에 '충돌이나 문제가 발생했나?' 를 검사한 후, 문제가 없으면 커밋하고 충돌이 발견된다면 트랜잭션 중 하나를 중단(abort) 시킨다.
### OCC의 실행 과정 3단계
1. Read Phase(읽기 단계)
	1. 트랜잭션은 다른 트랜잭션에게 변경 내용을 공개하지 않고, 자신만의 공간에서 필요한 데이터를 읽고 모든 작업을 실행한다.
	2. 이 단계가 끝나면 이 트랜잭션이 어떤 데이터를 읽었고, 어떤 데이터를 수정했는지(read set, write set)가 명확해진다.
2. Validation Phase(검증 단계)
	1. 변경 내용을 커밋하기 전에, 내가 작업하는 동안 충돌이 있었는지를 검사한다. 
	2. 충돌이 발견된다면, 충돌이 발생한 트랜잭션은 롤백되고 처음부터 다시 시작해야 한다.
3. Write Phase(쓰기 단계)
	1. 검증 단계를 통과하면, 충돌이 없다는게 확인이 된 것이다.
	2. 이제 자신만의 공간에서 작업했던 모든 변경 내용을 실제 DB에 적용(커밋) 한다.

**OCC의 장단점**
- 충돌이 적게 일어나는 환경(대부분 읽기 작업만 하는 환경 등..)에서는 잠금을 거는 비용이 없으므로 높은 성능을 보여준다.
- 하지만 충돌이 잦은 환경에서는 재시도 비용이 커져 오히려 성능이 저하될 수 있다.


## 다중 버전 동시성 제어(Multiversion Concurrency Control, MVCC)
-> '모든 변경 내역을 보관하여, 각자 다른 버전을 보게 하자!'

- 데이터를 덮어쓰는 대신, 데이터가 변경될 때마다 새로운 버전의 데이터를 만든다.
- 각 버전에는 트랜잭션 ID나 타임스탬프를 붙여 구분한다.
- 이렇게 하면 여러 트랜잭션이 동시에 같은 데이터에 접근하더라도, 각 트랜잭션은 자신이 시작된 시점을 기준으로 일관된 버전의 데이터를 보게 된다.

### MVCC의 작동 원리
- 읽기 작업
	- 읽기 트랜잭션은 다른 트랜잭션이 데이터를 수정하더라도, 기다릴 필요가 없다.
	- 그 이유는 읽기 트랜잭션이 시작될 때, 유효했던 과거 버전의 데이터를 읽으면 되기 때문이다.
- 쓰기 작업
	- 쓰기 트랜잭션은 데이터의 새로운 버젼을 생성한다.
	- 이 작업이 커밋되면, 이후에 시작디는 새로운 트랜잭션들은 이 새 버젼을 보게 된다.

**MVCC의 장점**
- DB의 특정 시점 스냅샷을 통해 일관된 뷰를 보장한다.
- 이 방식은 읽기 작업이 매우 많은 환경에서 성능 저하 없이 높은 동시성을 제공하기 때문에 현재 많은 DBMS에서 사용중이다.

## 비관적 동시성 제어 (Pessimistic Concurrency Control, PCC)
-> '최악을 가정하고, 일단 잠그고 보자'

- PCC는 OCC와는 반대로 '트랜잭션간의 충돌은 자주 발생할 거야' 라고 비관적으로 가정하는 방식이다.
- 그래서 다른 트랜잭션이 문제를 일으키기 전에 데이터를 수정하려면 반드시 먼저 잠금(Lock)하도록 강제한다.
- 다른 트랜잭션은 이 '잠금'이 풀릴 때까지 해당 데이터에 접근할 수 없으므로, 충돌이 원천적으로 차단된다.
### PCC의 작동 원리
1. 트랜잭션이 데이터에 접근하려고 할 때, 먼저 해당 데이터에 대한 잠금을 요청한다.
2. 해당 데이터에 대한 잠금이 확인되면, 작업을 수행한다.
3. 만약 다른 트랜잭션이 이미 잠금된 상태라면, 그 잠금이 해제될 때까지 끝까지 기다리거나(blocking), 작업을 포기하고 나중에 다시 시도한다.

**PCC의 장단점**
- PCC는 데이터의 일관성을 매우 강력하게 보장해주기 때문에, 충돌이 많이 일어나는 환경(계좌 이체, 재고 관리 시스템 등)에서 안정적으로 작동한다.
- 하지만 안정성이 높은만큼 성능이 떨어질 수 있다.
- 예를들어 여러 트랜잭션이 서로의 잠금이 풀리기를 기다리면서 시스템 전체의 성능이 저하되거나
- 최악의 경우 서로가 서로를 영원히 기다리는 '교착 상태(Deadlock)'에 빠질 수 있다.

## 잠금 기반 동시성 제어 (Lock-Based Concurrency Control)
-> PCC를 구현하는 가장 대표적이고 일반적인 방법이다.

- 핵심은 2PL(Two-Phase Locking, 2PL)이란느 프로토콜을 사용하는 것이다.
### 2단계 잠금 프로토콜 (Two-Phase Locking, 2PL)

1. **확장 단계**(Growing/Expanding Phase) : 이 단계에서 트랜잭션은 잠금을 획득할 수만 있고, 어떤 잠금도 해제할 수 없다.
2. **축소 단계**(Shrinking Phase) : 트랜잭션이 단 하나의 잠금이라도 해제하는 순간, 이 '축소 단계'로 진입힌다. 이 단계부터는  새로운 잠금을 획득할 수 없고, 오직 보유한 잠금을 해제하는 것만 가능하다.

### 2단계 커밋(2PC) vs 2단계 잠금(2PL)
이름이 비슷해서 헷갈리기 쉽지만, 둘은 완전히 다른 개념이다.
	**2단계 커밋(2PC):** 여러 데이터베이스에 걸친 **분산 트랜잭션**을 원자적으로 처리하기 위한 프로토콜입니다. (13장에서 다룸)
	**2단계 잠금(2PL):** 하나의 데이터베이스 안에서 **동시성**을 제어하기 위한 잠금 프로토콜입니다.

## 교착 상태 (Deadlocks)
: 그냥 무한 대기 상태이다.

### 교착 상태 해결 방법
1. **교착 상태 감지 (Deadlock Detection)**
	1. 데이터베이스는 내부적으로 대기 그래프(Wait for Graph)를 그려서 트랜잭션 간의 대기 관계를 추적관리한다.
	2. 이 그래프에서 교착 상태 사이클이 발견된다면, 교착 상태로 판단한다.
2. **희생양 선택 및 롤백**
	1. 교착 상태가 감지되면, 데이터베이스는 트랜잭션 중 하나를 '희생양(victim)'으로 선택하여 강제로 중단시키고 롤백(rollback)시킨다.
	2. 이렇게 하면 희생된 트랜잭션의 잠금이 해제되어서 다른 트랜잭션이 작업을 계속할 수 있다.
3. **교착 상태 예방 (Deadlock Prevention/Avoidance)**
	1. `wait-die`나 `wound-wait` 같은 기법들은 트랜잭션의 타임스탬프(시작 시간)를 비교하여, 우선순위가 낮은 트랜잭션이 높은 트랜잭션의 잠금을 기다리지 못하게 하거나(중단), 높은 트랜잭션이 낮은 트랜잭션의 잠금을 빼앗아(중단시킴) 오도록 하여 원형 대기(= 교착 상태)가 생기지 않도록 한다.

## 잠금(Locks)과 래치(Latches)
이 둘은 혼동하기 쉽지만, DB내부에서 목적과 작동 방식이 완전 다르다.

1. 보호 대상
	1.  잠금은 논리적 데이터(행, 테이블, 키)를 보호.
	2. 래치는 물리적 메모리 구조(B-tree 페이지, 버퍼)를 보호
2. 목적
	1. 잠금은 트랜잭션 간의 일관성 보호
	2. 래치는 스레드 간의 데이터 구조 보호
3. 지속 시간
	1. 잠금은 지속시간이 길다(트랜잭션 전체 기간)
	2. 래치는 매우 짧다(특정한 연산 수행 동안)
4. 관리 주체
	1. 잠금은 잠금 관리자(교착 상태 감지 기능 포함)
	2. 래치는 개별 스레드
예를 들면, 잠금은 특정 서류(데이터)에 대한 접근 권한이고, 래치는 서류함(메모리 구조)을 잠시 여는 열쇠라고 생각하면 된다.

### 읽기-쓰기 잠금 (Readers-Writer Lock)

래치(또는 잠금)의 효율을 높이기 위한 방법이다. "읽는 작업끼리는 서로 방해하지 않는다"는 점에 착안했다.
- **공유 잠금 (Shared Lock, Read Lock):** 여러 스레드가 **동시에 읽기**를 원할 때 사용된다. 여러 스레드가 동시에 이 잠금을 획득할 수 있다.
- **배타적 잠금 (Exclusive Lock, Write Lock):** 스레드가 **쓰기**를 원할 때 사용된다. 오직 하나의 스레드만 이 잠금을 획득할 수 있으며, 이 잠금이 걸려있는 동안에는 다른 어떤 읽기/쓰기 스레드도 접근할 수 없다.
### Latch Crabbing(래치 크래빙)
: B-Tree와 같은 데이터 구조를 탐색할 때, 동시성을 극대화하기 위한 최적화 기법이다.

- 문제점 : B-tree에서 특정 데이터를 찾으려면, 루트 노드부터 리프 노드까지 내려가야 한다. 만약 이 전체 경로를 통채로 래치로 잠가버리면, 다른 스레드는 아무 작업도 할 수 없어 성능이 크게 저하된다.

**해결책(= 래치 크래빙 과정)**
1.  스레드가 부모 노드의 래치를 획득한다.
2.  부모 노드에서 내려갈 자식 노드를 찾는다.
3. 자식 노드의 래치를 획득한다.
4. 자식 래치를 획득하는 즉시, 더 이상 필요 없는 부모 노드의 래치를 해제한다.
tmi) 위 과정처럼 노드를 잡았다가, 놓아줬다가 하는 과정을 반복해서 게가 집게발로 잡고 놓고 하는것과 비슷한 느낌이라 래치 크래빙이란 이름이 붙었다.

- 위와 같은 과정으로 하면 스레드가 탐색하는 아주 작은 부분만 잠기기 때문에, 다른 스레드들이 트리의 다른 부분을 동시에 탐색하거나 수정할 수 있게 되어 동시성이 극적으로 향상된다.
- 단, 쓰기 작업 시에는 자식 노드가 꽉 차서 분할이나 병합이 예상될 경우, 부모 노드의 래치를 미리 해제할 수 없고 계속 유지해야 한다.
