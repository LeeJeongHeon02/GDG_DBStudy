오랜만에 다시 DatabaseInternals 공부를 시작한다.
종강을 하고, 6월 말에 일본 여행도 1주일씩 다녀오느라 7, 8주차를 pass해버렸다. pass했다고 포기하는것이 아닌, 다시 마음을 잡고 빠르게 스터디로 복귀해야겠다. ~~너가 선택한 스터디잖아!!! 책임져~~

간단하게 강의록을 보면서 7, 8주차 내용을 알아둔 후, 9주차 예습을 시작해야겠다.

---

## week8 간단학습

### 트랜잭션
트랜잭션은 분할할 수 없는 데이터베이스 동작의 논리적 단위이다.
라고 말하면 조금 추상적인 느낌이다.

A계좌에서 10만원을 인출하는 과정1.
B계좌에 10만원을 입금하는 과정2.
이 각각의 과정이 하나의 처리 과정이라고 생각해보자.

그러면 과정1이 성공적으로 마무리되었지만 과정2에서 문제가 생겼다고 해보자.

그러면 A계좌에서는 10만원이 인출되었지만, B계좌에는 돈이 안들어왔을것이다. 그럼 돈이 사라져버린것이다!!!

이와 같은 문제를 해결하기 위해서 과정1 + 과정2를 하나의 트랜잭션으로 처리하면 된다. 그러면 과정2에서 오류가 생기더라도, 하나의 트랜잭션이기때문에 연산을 되돌리면 된다.

### 트랜잭션의 4대 속성
어디선가 들어봤을것이다. ACID. 사실 이 스터디에서 들어봤다.ㅋㅋ
**A**tomicity 원자성
- 트랜잭션은 더 이상 나눌 수 없다 : 트랜잭션의 모든 작업은 모두 성공하거나, 모두 실패해야 한다.
- 위의 계좌 이체 예시가 이에 해당한다
**C**onsistency 일관성
- DB는 언제나 일관성 있는(유효한) 상태를 유지해야 한다는 원칙이다. DB에 정의된 모든 규칙을 트랜잭션이 위반해서는 안 된다.
- 예시로, 3만원이 있는데 5만원을 인출하려고 하면 '계좌의 잔액은 0원 이상이어야 한다'는 규칙을 어긴다(마통은 없다고 가정).
**I**solation 독립성
- 동시에 여러 트랜잭션이 실행되어도, 혼자서만 실행되는 것처럼 방해 없이 동작해야 한다.
- 하나의 트랜잭션이 실행중일때, 다른 트랜잭션이 끼어들어서 데이터를 변경하는것을 막아준다.
**D**urability 영속성
- 성공적으로 완료된 트랜잭션의 결과는 시스템에 장애가 발생하더라도 영구적으로 기록되어야 한다는 원칙이다.

데이터베이스는 이 ACID원칙을 지키기 위해 트랜잭션 관리자, 잠금 관리자, 로그 관리자 같은 내부 장치들을 이용해 모든 작업이 안전하고 신뢰성 있게 처리되도록 보장한다.

### 버퍼 관리
느린 디스크와 빠른 메모리 사이에서 어떻게 성능을 뽑아낼지에 대한 이야기를 다룬다.

### 페이지 캐시의 역할
**캐싱** : 디스크에서 읽어온 데이터를 메모리에 잠시 보관한다. 다음에 또 이 데이터가 필요하면, 디스크까지 가지 않고 메모리에서 바로 가져와서 속도가 빨라질 수 있다.
**버퍼링** : 데이터 변경 사항을 일단 메모리에서 처리하고, 나중에 한꺼번에 모아서 디스크에 쓴다.
**더티 페이지** : 메모리에서 내용이 바뀌었지만, 아직 디스크에는 반영되지 않는 페이지를 말한다. 

메모리는 자원이 한정되어있어서 새로운 데이터를 가져오려면 기존의 데이터 중 하나를 치워야한다(= 디스크로 돌려보내야한다). 이걸 제거(Eviction) 이라고 한다.
이때 아무거나 막 치우면 안되므로, 페이지 교체 정책(Page Replacement Policy라는 규칙을 사용한다)
LRU(Least Recently Used) : 가장 오랫동안 안 쓴 내용을 지우는, 가장 고전적이고 가장 많이 쓰이는 방법이다.
LFU(Least Frequently Used) : 가장 덜 사용한 내용을 지우는 방법
등이 있다.

결론적으로 버퍼 관리는 디스크 I/O 횟수를 최소화하여 데이터베이스의 전체적인 성능을 극대화하는 핵심 기술이다.

---
# Week9

## Recovery
이 챕터는 '데이터가 어떻게 안전하게 저장될까?', '갑자기 컴퓨터가 꺼져도 내 데이터는 괜찮을까?' 에 대한 답이 나와있는 파트이다. 데이터 안정성 기술에 대한 이야기를 한다.

### WAL(Write-Ahead Log)
: 쓰기 전용 로그(커밋 로그)
- WAL은 DB에서 일어나는 모든 변경 작업의 이력을 기록하는 일종의 장부라고 생각하면 된다.
- Write-Ahead : 데이터베이스는 실제 데이터 파일(테이블, 인덱스)을 변경하기 전에, **반드시** 어떤 데이터를 어떻게 변경할 것이다. 라는 내용을 WAL에 먼저 기록해야 한다.
- Append-Only : WAL은 한 번 기록된 내용은 절대 수정하거나 지우지 않고, 새로운 기록을 계속해서 뒤에 덧붙인다.

**WAL이 필요한 이유?**
DB는 성능을 위해, 변경 사항을 메모리에만 먼저 적용하고, 나중에 한번에 모아서 디스크에 쓴다고 배웠다. 
이때, 디스크에 쓰여지기 전에 전원이 차단되어서 메모리에 있는 정보가 날라간다면? WAL이 해결해준다.
변경 사항을 메모리에 적용하기 전에, 디스크에 있는 WAL파일에 어떤 데이터를 어떻게 변경할 것이다. 라는 내용을 기록해두면, 메모리의 내용이 사라져도 WAL에 남은 기록을 보고 복원이 가능하다.

**ARIES(Algorithm for Recovery and Isolation Exploiting Semantics)**
- **부분 롤백**: 트랜잭션 중 일부만 되돌릴 수 있다.
- **세밀한 잠금**: 레코드 단위의 잠금으로 높은 동시성 제공
- **장애 복구**: 시스템 장애 발생 시 로그를 기반으로 복구 수행
- 위와 같은 기능을 제공한다.
	- 장애 발생 후 다음과 같은 3단계의 과정으로 복구한다
	1. **분석(Analysis)** : 체크포인트 이후의 로그를 분석해 어떤 트랜잭션이 진행 중이었는지 파악
	    -  Dirty Page Table과 Transaction Table을 구성
    2. **재실행(Redo)** : 장애 발생 전의 작업을 로그를 따라 그대로 다시 실행해서 상태를 복원
    3. **되돌리기(Undo)** : 완료되지 않은 트랜잭션을 로그를 거꾸로 따라가며 되돌림
	    -  이때 보상 로그(Compensation Log Record, CLR)를 남겨서 Undo 작업도 기록함

## 로그의 작동 방식(Log Semantice)

### 로그 레코드와 LSN(Log Sequence Number)
- WAL 에 기록되는 각 항목을 로그 레코드라고 한다.
- 모든 로그 레코드는 LSN이라는 시퀀스 번호를 가진다.
- 이 번호는 계속 증가하기 때문에, 어느 작업이 먼저 일어났는지를 명확하게 알 수 있다.

### 보상 로그 레코드(Compensation Log Records, CLRs)
- 트랜잭션을 되돌리는 Undo 작업도 작업이다.
- 이 작업 중에 시스템이 다운된다면?
- CLR은 이 되돌리기 작업 자체를 로그에 기록해서, 복구 과정이 중단되더라도 문제가 생기지 않도록 보장한다.

### 체크포인트 (Checkpoints)
- WAL이 무한정 계속 쌓이기만 할 수는 없을것이다.
- Checkpoints는 '여기까지는 모두 안전하게 반영되었으니, 이 체크포인트 이전의 WAL기록은 지워도 괜찮다'를 알려주는 표시이다.

### 퍼지 체크포인트 (Fuzzy Checkpoint)
- 대부분의 데이터베이스가 이 퍼제 체크포인트를 사용한다.
- 모든 트랜잭션을 멈추고 체크포인트를 찍는 것이 아닌, 체크포인트 시작과 체크포인트 끝을 로그에 기록한다.
- 그리고 그 사이에 백그라운드에서 변경된 데이터들을 디스크에 반영한다.
- 이러게 하면 시스템 중단 없이 효율적으로 로그를 관리할 수 있다.

## Operation Log vs Data Log
: 로그에는 어떤 내용을 기록해야 할까?

### 물리적 로그 (Physical Log)
- 데이터의 이전 상태와 이후 상태를 통째로 기록하는 방식이다.
- 복구할 때, 이전 데이터롤 그냥 덮어쓰기만 하면 되므로 빠르지만, 데이터(로그)의  크기가 매우 커질 수 있다.

### 논리적 로그(Logical Log)
- 수행해야 할 작업(Operation) 자체를 기록하는 방식이다. 로그 크기는 작지만, 복구시 해당 작업을 다시 실행해야 하므로 더 복잡할 수 있다.

현실에서 많은 시스템은 이 둘을 섞어쓴다.
복구 시 시스템을 빠르게 되돌리기 위한 재실행(Redo)에는 물리적 로그를 사용하고,
동시성을 높이며 트랜잭션을 되돌리는 실행 취소에는 논리적 로그를 사용한다.

## Steal and Force Policies
: 언제 디스크에 내용을 쓸까?
DB는 메모리에 있는 변경된 페이지(더티 페이지)를 언제 디스크에 쓸지 결정하는 정책을 가진다. 이는 복구 방식에 큰 영향을 준다.

### Steal vs No-Steal
- Steal은 트랜잭션이 아직 커밋되지 않은 상태여도, 변경된 내용을 디스크에 쓸 수 있도록 허용하는 방식이다.
- No-Steal은 반대로 커밋되지 않은 트랜잭션의 변경 내용은 절대 디스크에 쓰지 못하게 하는 방식이다.

### Force vs No-Force
- Force : 트랜잭션이 커밋될 때, 그 트랜잭션이 변경한 모든 내용을 **반드시** 디크스에 즉시 쓰도록 강제하는 방식이다.
- No-Force : 트랜잭션이 커밋되더라도, 변경된 내용이 디스크에 즉시 쓰여지는 것을 보장하지 않는 방식이다.

이 Steal과 Force의 조합에 따라서 복구의 복잡성이 달라진다.
예를 들어, `FORCE` 정책은 커밋된 데이터는 항상 디스크에 있으므로 복구가 단순하지만, 커밋마다 디스크 쓰기가 발생해 성능이 매우 느리다. 반면, 대부분의 고성능 데이터베이스가 채택하는 **`STEAL`과 `NO-FORCE` 조합**은 성능은 좋지만, 커밋되지 않은 내용이 디스크에 있을 수도 있고(Undo 필요), 커밋된 내용이 디스크에 없을 수도 있어(Redo 필요) 복구 로직이 가장 복잡해진다.

## ARIES
: 바로 위에서 말한 STEAL 과 NO-Force환경을 위해 설계된, 현대 데이터베이스 복구의 표준과 같은 알고리즘이다. 
(뒤에 나와있는줄 모르고 위에 정리했지만, 다시한번 자세하게 정리하자.)

### 복구 과정의 3단계
1. **분석 단계**
	- 마지막 체크포인트부터 WAL을 읽는다.
	- 시스템이 다운될 떄, 어떤 페이지들이 메모리에서 변경 중이었는지(더티 페이지 확인), 어떤 트랜잭션들이 완료되지 못했는지의 목록을 만든다.
2. **재실행 단계**
	- 분석 단계에서 파악한 지점부터 로그 끝까지 모든 기록을 다시 한번 그대로 실행한다.
	- 이는 시스템의 상태를 정확하게 다운 직전의 순간으로 되돌리는 과정이다.
	- 이 단계에서는 모든 변경을 다시 적용한다.
3. **실행 취소 단계**
	- 재실행 단계에서 다운 직전 상태로 만들었다.
	- 이때 1. 분석 단계에서 찾아낸 '완료되지 못한 트랜잭션' 들의 기록을 역순으로 따라가며 하나씩 되돌린다(롤백)
	- 이 과정을 통해 DB는 모든 작업이 원자적으로 처리된, 완벽하게 일관된 상태로 돌아가게 된다.

---

# 동시성 제어(Concurrency Control)
atm을 생각해보자.
돈을 입금하고 인출할 때, 이 과정들이 제대로 통제되지 않는다면 잔액이 엉망이 될 것이다.
데이터베이스도 이처럼 수많은 사용자가 동시에 데이터를 읽고 쓰려고 할 때, 데이터가 꼬이지 않고 정확성을 유지하도록 제어하는 기술이 **동시성 제어** 이다.
이는 트랜잭션을 안전하게 처리하면서, 최대한의 성능을 내는것이다.

## 직렬성(Serializability)
직렬성은 **여러 트랜잭션이 동시에 실행**되더라도, 그 **결과는 어떤 순서로든 차례대로 실행한 것과 동일해야 한다**는 원칙이다.
- 예시 // 잔액은 1000원으로 설정
- A과정은 계좌에 100원을 입금
- B과정은 계좌에서 50만원을 출금
- 위 두 과정이 동시에 실행된다고 가정해보자.
- 직렬성이 보장된다면 최종 결과는 아래 두 가지중 하나와 같아야한다.
- A실행 후 B 실행 : 잔액 1050원
- B실행 후 A실행 : 잔액 1050원
- 이처럼 중간에 데이터가 꼬여서 이상한 잔액이 나오지 않게 보장하는것이 **직렬성** 이다.
- 이처럼 완벽한 데이터 보호를 제공하지만, 성능을 위한 비용이 비쌀 수 있다.

## 트랜잭션 고립성(Transactiom Isolation)
고립성은 ACID 원칙 중 I에 해당하는 원칙이다.
하나의 트랜잭션이 실행되는 동안 다른 트랜잭션의 영향을 얼마나 차단할 것인가를 결정하는 속성이다.
- DB는 이 격리 수준(Isolation Level)을 여러 단계로 나눠서 제공한다.
- 격리 수준이 높을수록 데이터는 안전해지지만, 다른 트랜잭션이 끝나기를 기다리는 시간이 길어져 성능이 떨어질 수 있다.

## 읽기 및 쓰기 이상 현상(Read and Write Anomalies)
격리 수준이 완벽하지 않을 때(직렬성이 깨질 때) 발생하는 데이터 꼬임 현상을 **이상 현상** 이라고 부른다.
동시성 제어의 주된 목적이 이 이상 현상을 막는 것이다.

### Dirty Read(더티 리트)
: 아직 커밋(확정)되지 않는 다른 트랜잭션의 변경 데이터르 읽는 현상
예시
- 트랜잭션 A가 상품 가격을 100원에서 120원으로 바꾸고, 아직 커밋하지 않은 상태이다.
- 이때 트랜잭션 B가 이 임시 가격인 120원을 읽어간다.
- 그런데 트랜잭션 A를 취소해버리면, 가격은 트랜잭션이 시작되기 전인 100원이 된다.
- 그러면 트랜잭션 B는 존재하지 않는 Dirty Read를 하게 된 것이다.

### Non-Repeatable Read(반복 불가능한 읽기)
: 한 트랜잭션 안에서 같은 데이터를 두 번 읽었는데 결과가 다른 현상
예시
- A가 상품 재고를 읽었을 때 10개였다.
- 잠시 후 B가 들어와 재고 3개를 팔고 커밋했다.
- 그 후 A가 재고를 읽어보니 재고가 7개였다.
- 그러면 처음의 10개라는 읽기 결과가 더 이상 반복되지 않는 상황이다.

### Phantom Read(유령 읽기)
: 한 트랜잭션 안에서 일정 범위의 데이터를 두 번 읽었는데, 첫 번째에는 없던 유령 같은 데이터가 두 번째에 나타나는 현상
예시
- A가 '20대의 회원'을 검색하니 5명이 나왔다.
- 그 사이 B가 25살인 신규 회원을 추가하고 커밋했다.
- A가 다시 '20대의 회원'을 검색하니, 아까는 없던 회원 한명이 추가되어서 6명이 검색된다.
- 이 회원이 바로 '유령'이다.

### Lost Update(손실된 업데이트 - 쓰기 이상 현상)
: 두 개 이상의 트랜잭션이 **같은 데이터를 동시에 수정**할 때, 나중에 덮어쓴 트랜잭션 때문에 **이전 트랜잭션의 수정 내용이 사라지는 현상**
예시
- 현재 한 게시글의 좋아요 수가 100개이다.
- A, B가 동시에 좋아요 버튼을 누른다.
- A가 좋아요 수 100을 읽어간다.
- B가 좋아요 수 100을 읽어간다.
- A는 100 + 1해서 101로 저장한다.
- B도 100 + 1해서 101로 저장한다.
- 결론적으로 좋아요 수는 102이어야 하지만, 101로 남게 된다.
- A의 업데이트가 B에 의해 덮어쓰여 데이터가 손실된것이다.

### Isolation Levels(고립 수준)
DB는 위에서 설명한 이상 현상들을 막기 위해, 개발자가 선택할 수 있는 몇 가지 고립 수준을 제공한다.
고립 수준이 높아질수록 데이터는 안전해지지만 성능은 저하될 수 있다.
