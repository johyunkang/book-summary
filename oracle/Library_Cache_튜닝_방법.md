## **SQL의 작성과 파싱**

------

데이터베이스에서 어떤 결과값을 얻기 위해 SQL문을 작성할때에도 개발자 개개인마다 이를 표현하는 방법은 다릅니다. 데이터베이스는 같은 결과를 얻는 경우에도, SQL문의 표현이 다르면 새롭게 파싱해야합니다. SQL문을 실행하면, 데이터베이스는 아스키 값으로 계산합니다. 즉, 대문자, 띄어쓰기, 주석에 따라 아스키 값이 다르므로 표현이 달라지면 다른 SQL문으로 인식하게됩니다.

 

같은 결과를 얻지만 표현이 다른 SQL문은 실행할때마다 library Cache에서 하드파싱됩니다. 하드파싱은 SQL문의 검색과 공간확보를 위해 Shared Pool Latch와 Library Cache Latch를 필요로 합니다. 잦은 하드파싱은 리소스를 과도하게 사용하고 래치를 오래 점유하므로 SQL문의 수행에 지연이 발생합니다. 따라서 SQL문을 재사용하는 소프트 파싱이 필요합니다. 이를 위해 SQL문 작성시, 대문자, 띄어쓰기, 주석에 대한 원칙을 새웁니다.

 

그런데, 표현방법이 같은 SQL문이여도 WHERE절의 조건값만 다른 경우는 조건으로 들어가는 값도 아스키 값으로 계산하므로 값이 달라지면, 다른 SQL문으로 인식되어 하드파싱됩니다. 이때 만약, 바인드 변수를 사용하면 소프트파싱의 가능성이 높아집니다. 바인드 변수를 사용하는 SQL문은 변수에 들어가는 값에 관계없이 파싱합니다.

 

파싱이 끝나면 바인드값을 대입하게 되는데, 적용되는 값에 상관없이 SQL을 공유할 수 있게 되는 것입니다.

 

SQL을 수행하게되면 Library Cache에서 해당 래치를 획득하고 수행하려는 SQL 실행정보(LCO:Library Cache Object)가 있는지 검색합니다. SQL이 있으면 LCO의 생성과정을 거치지 않고 바로 실행할 수 있습니다. 이것을 소프트파싱이라고 합니다. 그런데, SQL이 Library Cache에 존재하지 않는 새로운 SQL이라면 LCO를 만들어 실행정보를 저장합니다. Shared Pool 래치를 획득하여 저장할 공간을 확보합니다.

 

공간이 마련되면 SQL의 LCO가 생성되고, 여기에 SQL문과 실행계획 등의 정보를 저장합니다. 이렇게 만들어진 LCO를 통해 SQL이 수행됩니다. 이와 같이 SQL문이 Library Cache 내에 존재하지 않아 LCO를 만들고, 여기에 실행정보를 저장하는 과정을 하드 파싱이라고 합니다. 하드파싱과 소프트파싱은 Library Cache 내의 SQL 존재유무에 따라서 구별이 됩니다. 최초로 수행되는 SQL은 하드파싱을 피할 순 없습니다. 그러나 두번째 수행부터는 소프트파싱을 하는게 보다 빠르고 적은 리소스를 사용하니 효율적이게 됩니다.

 

 

*** Child LCO 생성**

SQL이 parsing이 되면, LCO를 만듭니다. LCO도 같은 SQL이라고해서 하나로 같이 쓰는것이 아니라, 유저가 다르고, 환경이 달라도 따로 LCO를 만들어서 관리합니다. 조건이 다르면 전부 다르게 LCO를 만들게됩니다. child LCO가 다르게 만들어지는 이유는 V$SQL_SHARED_CURSOR란 뷰를 보면 확인이 가능합니다.

 

오라클 프로시저나 테이블과 같은 객체에 대해서는 스키마명을 항상 같이 저장하기 때문에 유일성이 보장됩니다. 하지만, SQL 문장의 경우에는 SQL 텍스트 자체가 이름으로 사용되기 때문에 유일성이 보장되지 않습니다. 따라서 오라클 SQL 텍스트를 이름으로 갖는 부모 LCO를 생성하고 실제 SQL 커서에 대한 정보는 자식 LCO에 저장합니다. 가령 두 개의 다른 스키마 A, B에서 텍스트는 동일하지만, 실제로 참조하는 객체는 다른 SQL문장을 수행한 경우, 오라클은 SQL 텍스트에 해당하는 부모 LCO와 스키마 A가 수행한 SQL 커서에 해당하는 자식 LCO, 스키마 B가 수행한 SQL 커서에 해당하는 자식 LCO, 총 세개의 LCO를 생성합니다.

 

 

## **LATCH: library cache**

------

latch: library cache 대기 이벤트는 libarary cache 래치를 획득하는 과정에서 경합이 발생하여 나타나는 대기 이벤트입니다. Shared Pool 래치가 프리 청크를 찾기 위해 프리리스트를 스캔하고, 적절한 청크를 할당하는 작업을 보호한다면, library cache 래치는 SQL을 수행하기 위해 libarary cache 메모리 영역을 탐색하고 관리하는 모든 작업을 보호합니다. **이 때, libarary cache 래치는 CPU count 보다 큰 소수(Prime Number) 중 가장 작은 수만큼 자식 래치(child latch)를 가집니다.**

 

*** Wait Time**

이벤트의 대기시간을 기하급수적으로 증가한다.

 

*** Parameter**

P1(프로세스가 대기하고 있는 래치의 메모리 주소), P2(래치 번호), P3(래치를 획득하기 위해 프로세스가 시도한 횟수)

 

*** 일반적인 문제상황 및 대처방안**

 \- 원인: 파싱이 과다한 경우

 \- 진단방법

   latch: library cache 대기가 높은 시점의 파싱에 소요된 시간(parse time elapsed)

   발생한 파싱 횟수(parse count(total), parse count(hard), SQL 수행 횟수(execute count)를 확인)

 \- 개선방법

   바인드 변수 사용, Web Application Server의 경우, Statement Cache 기능 사용

   애플리케이션 수정, Static SQL을 사용

   session_cached_cursors 파라미터의 조정

 

 \- 원인: 버전 카운트(Version count)가 높은 경우

 \- 진단방법: V$SQLAREA 뷰에서 latch: libarary cache의 보유 시간이 긴 SQL의 VERSION_COUNT 칼럼 값을 확인

 

 \- 원인: SGA(System Global Area) 영역의 페이지 아웃(Page out)이 발생하는 경우

 \- 진단 방법: latch: library cache 대기가 높은 시점 O/S에서 스왑(Swap) 발생

 \- 개선 방법

   Memory 과다 사용 프로세스 검출

   HP-UX, AIX: LOCK_SGA 파라미터값을 TRUE 값으로 변경(DEFAULT = FALSE)

   SonOS: _USE_ISM 파라미터 값이 TRUE 인지 확인(DEFAULT = TRUE)

 

 

**버전 카운트(Version Count)**

```sql
Scott: select * from emp where empno = 1;
Mary: select * from emp where empno = 1;
John: select * from emp where empno = 1;
```

위의 세 SQL 문장은 Text가 완전히 동일하므로 동일한 해시 값을 갖습니다. 따라서 동일한 해시 체인(Hash Chain)의 동일한 핸들에 할당됩니다. 하지만 emp 테이블이 모두 스키마가 다른 테이블이므로 실제로는 다른 SQL문장입니다. 이 경우 오라클은 Text에 해당하는 부모 LCO를 두고 그 밑에 세 개의 자식 LCO를 만들어 개별 SQL 정보를 관리합니다. 세 개의 자식 LCO는 실제로는 익명 리스트(Anonymous List)라고 하는 별도의 리스트에 저장됩니다. 세 개의 자식 LCO를 가지므로 V$SQLAREA 뷰의 VERSION_COUNT(버전 카운트) 칼럼 값이 자식 LCO의 개수와 같은 3의 값을 가지게 됩니다. 버전 카운트가 높다는 것은 자식 LCO 탐색으로 인해 library cache를 탐색하는 시간이 그만큼 증가한다는 것이며, 이로 인해 library cache 래치 경합이 증가할 수 있다는 것을 의미합니다. 만일 특정 SQL 문장에서 library cache 래치 경합이 많이 발생한다면 해당 SQL의 버전 카운트 값을 확인해 볼 필요가 있습니다. 오라클의 버그로 인해 버전 카운트가 불필요하게 높아지는 경우가 있기 때문입니다.

 

**SGA 영역의 페이지 아웃(Page Out)**

Shared Pool이 디스크로 페이지 아웃된 경우, 해당 영역에 대한 스캔이 발생할 때 다시 디스크의 내용을 메모리로 불러들이는 과정(페이지 인)동안 대기해야 하므로 library cache 래치에 대한 대기시간이 증가할 수 있습니다. 만일 latch: library cache 대기가 높은 시점에, O/S에서 스왑현상이 발생한다면, 페이지 아웃에 의한 성능 저하일 확률이 높습니다.

 

**SESSION_CACHED_CURSORS**

SESSION_CACHED_CURSORS 파라미터 값이 세팅되어 있으면 오라클은 세 번 이상 수행된 SQL 커서에 대한 정보를 PGA(Program Global Area) 내에 보관합니다. 사용자가 SQL을 수행 요청할 때 오라클은 PGA에 캐싱된 정보가 있는지 확인하고, 만일 캐싱된 정보가 있다면 캐싱된 정보를 이용합니다. 따라서 library cache 영역을 탐색하는 시간이 줄어들어 상대적으로 library cache 래치를 보유하는 시간이 줄어들게 됩니다. SESSION_CACHED_CURSORS 파라미터의 기본값은 버전마다 다릅니다. 만일 기본 값이 작다면 되도록이면 50 이상의 값을 설정하는 것이 바람직합니다.

 

 

## **LATCH: cache buffers chains**

------

latch: cache buffers chains는 cache buffers chains 래치를 획득하는 과정에서 경합이 발생하여 나타나는 이벤트입니다. 버퍼 캐시를 사용하기 위해 해시 체인을 탐색하거나 변경하려는 프로세스는 반드시 해당 체인을 관리하는 cache buffers chains 래치를 획득해야하는데, 이 과정에서 경합이 발생하면 latch: cache buffers chains 이벤트를 대기하게 됩니다.

 

*** Wait Time**

이벤트의 대기시간은 기하급수적으로 증가한다.

 

*** Parameter**

P1(프로세스가 대기하고 있는 래치의 메모리 주소), P2(래치번호), P3(래치를 획득하기 위해 프로세스가 시도한 횟수)

 

*** 일반적인 문제상황 및 대처방안**

 \- 원인: 비효율적인 SQL문장 사용

 \- 진단 방법: cache buffers chains 래치 대기가 발생하는 시기에 V$SQLAREA 뷰를 통하여 SQL을 확인, TRACE를 통하여 과다한 처리범위를 발생시키지 않는지에 대한 여부를 확인

 \- 개선 방법: SQL문장 튜닝

 

 \- 원인: 핫 블록(HOT Block)에 의한 문제

 \- 진단 방법: V$LATCH_CHILDREN 뷰에서 cache buffers chains 래치에 해당하는 특정 자식 래치의 CHILD#과 GETS, SLEEPS 값이 높은지 확인 

   V$SESSION_WAIT 뷰에서 래치의 주소를 얻어 과다하게 중복된 주소가 있는지 확인

 \- 개선 방법: PCTFREE를 높게 주거나 작은 크기의 블록을 사용

​         파티셔닝 적용, 해당 블록의 로우들에 대해서만 삭제 후, 재삽입 작업 수행

 

 

**HOT BLOCK 여부 판단**

```sql
select * from
(select child#, gets, sleeps from v$latch_children
         where name = 'cache buffers chains'
         order by sleeps desc
) where rownum <= 20;
```

V$LATCH_CHILDREN 뷰에서 자식 cache buffers chains 래치에 해당하는 CHILD#과 GETS, SLEEPS 값을 비교하여, 특정 자식 래치에 사용하는 횟수와 경합이 집중되는지 판단하여 Hot block 여부를 알 수 있습니다. 다음 명령문을 이용해서 SLEEPS 회수가 높은 자식 래치를 얻습니다. **만일 특정 자식 래치의 GETS, SLEEPS 값이 다른 자식 래치에 비해서 비정상적으로 높다면 해당 래치가 관장하는 체인에 핫 블록이 있는 것으로 추측할 수 있습니다.**

 

 

**V$BH 뷰로 어떤 블록들이 HOT BLOCK인지 판단**

```sql
select hladdr, obj,
(select object_name from dba_obejcts where
(data_object_id is null and object_id = x.obj) or
 data_object_id = x.obj and rownum = 1) as object_name,
       dbarfil, dbablk, tch from x$bh x
where hladdr in
('C0000000CDFF24F0', 'C0000000CE3ADDF0', 'C0000000CDF18A98')
order by hladdr, obj;
```

X$BH 뷰를 이용하면 정확하게 어떤 블록들이 핫 블록인지 확인할 수 있습니다. X$BH 뷰로부터 1) 사용자 객체(Table, Index)에 해당하며, 2) Touch Count가 높은 블록을 기준으로 핫 블록을 추출할 수 있습니다.

 

 

## **LATCH: cache buffers lru chain**

------

latch: cache buffers lru chain은 cache buffers lru chain 래치를 획득하는 과정에서 경합이 발생하여 나타나는 이벤트입니다. Working Set(lru + lruw)을 탐색하거나 변경하려는 프로세스는 항상 해당 Working Set을 관리하는 cache buffers lru chain 래치를 획득해야 하는데, 이 때 경합이 발생하면 latch: cache buffers lru chain 이벤트를 대기하게 됩니다.

 

*** Wait Time**

이벤트의 대기시간은 기하급수적으로 증가한다.

 

*** Parameter**

P1(프로세스가 대기하고 있는 래치의 메모리 주소), P2(래치번호), P3(래치를 획득하기 위해 프로세스가 시도한 횟수)

 

*** 일반적인 문제상황 및 대처방안**

 \- 원인: 비효율적인 SQL문장 사용

 \- 진단 방법: cache buffers lru chain 래치 대기가 발생하는 시기에 V$SQLAREA 뷰를 통하여 SQL을 확인, TRACE를 통하여 과다한 처리범위를 발생시키지 않는지에 대한 여부를 확인

 \- 개선 방법: SQL문장 튜닝

 

 \- 원인: 버퍼 캐시 크기가 너무 작은 경우

 \- 진단 방법: 버퍼 캐시의 히트율을 확인하기 위해서는 V$SYSSTAT, V$SESSTAT을 확인하고, 버퍼 캐시 영역을 분석하기 위해서 V$BUFFER_POOL, V$BUFFER_POOL_STATISTICS를 확인

 \- 개선 방법: 버퍼 캐시의 크기를 충분히 크게 한다.

 

 \- 원인: 체크 포인트 주기가 지나치게 짧은 경우

 \- 진단 방법: 버퍼 캐시의 히트율을 확인하고, FAST_START_MTTR_TARGET 또는 LOG_CHECKPOINT_TIMEOUT 파라미터를 통해 체크 포인트 주기를 확인

 \- 개선 방법: FAST_START_MTTR_TARGET 파라미터를 조정하여, 체크 포인트 주기를 합리ㅏ적으로 지정

 

 

**cache buffers chains 래치와 cache buffers lru chain 래치 경합간의 차이**

cache buffers chains 래치와 cache buffers lru chain 래치 경합간의 차이점에 대해서 이해라 필요가 있습니다. 만일 동일 테이블이나 인덱스를 여러 세션이 동시에 스캔하는 경우라면, cache buffers chains 래치 경합이 발생할 확률이 높습니다. 동일 체인에 대한 경합이 발생하기 때문입니다. 하지만, 다른 테이블이나 인덱스들을 여러 세션이 동시에 스캔하는 경우라면 cache buffers lru chain 래치 경합이 발생할 확률이 높습니다. 여러 세션들이 모두 다른 블록들을 메모리에 올리는 과정에서 프리 버퍼를 확보하기 위한 요청이 많아지고 이로 인해 Working Set에 대한 경합이 발생할 확률이 높아집니다. 특히 데이터의 변경이 빈번해서 더티 버퍼의 개수가 많고 이로 인해 DBWR가 체크 포인트를 위해 lruw 리스트를 탐색하는 횟수가 잦다면 cache buffers lru chain 래치의 경합은 더욱 심해집니다. cache buffers lru chain 래치 경합의 또다른 중요한 특징은 물리적 I/O를 수반한다는 것입니다. **비효율적인 인덱스 스캔에 의한 문제라면 db file sequential read 대기와 lru chain 래치 경합이 함께 발생하게 되고,** 불필요한 풀테이블스캔이 많다면 db file scattered read 대기와 lru chain 래치 경합이 함께 발생하게 됩니다.

실제로는 cache buffers chains 래치 경합과 cache buffers lru chain 래치 경합이 같이 발생하는 경우가 많은데, 복잡한 애플리케이션들에서는 위에서 언급한 패턴들이 복합적으로 사용되기 때문입니다.

 

 

**버퍼 캐시 크기 증가 여부 판단**

```sql
select to_char((sum(decode(name, 'consistent gets', value, 0)) + 
        sum(decode(name, 'db block gets', value, 0)) - 
        sum(decode(name, 'physical reads', value, 0)) - 
        sum(decode(name, 'physical reads direct', value, 0))) / 
        (sum(decode(name, 'consistent gets', value, 0)) + 
         sum(decode(name, 'db block gets', value, 0))) *
        100, '999.99') || ' %' "Buffer Cache Hit Ratio"
from v$sysstat;
```

다수의 비효율적인 SQL로 인한 프리버퍼를 과도하게 요청하는 경우 버퍼 캐시 히트율을 떨어뜨리는 주요 원인이 됩니다. 그리고 버퍼 캐시의 크기가 지나치게 작을 경우 또한 히트율을 떨어뜨리는 주요 원인입니다. 버퍼 캐시 크기 증가를 고려하기 위해 버퍼 캐시 히트율을 확인해 볼 필요가 있습니다. 또한 free buffer waits, buffer deadlock, buffer busy waits 이벤트 다수 발생 시에도 버퍼 캐시 크기 증가를 고려해 볼 필요가 있습니다.

 

 

## **library cache pin**

------

library cache pin 이벤트 대기는 **Library Cache Object의 실행정보를 바꾸거나 참조하는 과정에서 경합이 발생할 때 관찰됩니다.** 가령 특정 SQL에 대해 최초로 하드파싱을 수행하는 세션은 해당 Library Cache Object에 대해 library cache pin을 Exclusive 모드로 획득합니다. 하드파싱이 이루어지는 동안 같은 SQL을 수행하고자 하는 세션들을 library cache pin을 Shared 모드로 획득하기 위해 대기해야 합니다. 이때 library cache lock 이벤트를 대기합니다.

 

*** Wait Time**

PMON 프로세스는 1초까지 대기하며, 다른 프로세스들은 3초까지 대기합니다. 해당 대기시간 후에도 핀을 획득하지 못할 경우 반복적으로 대기합니다.

 

*** Parameter**

 참고) library cache pin 대기 이벤트는 대기 파라미터를 사용하지 않습니다.

 P1(핀(pin) 대기와 관련된 오브젝트의 메모리 주소), P2(핀(pin)의 메모리 주소), P3(모드(mode)와 네임스페이스(namespace))

 

*** Common Causes and Actions**

 \- 원인: 현재 많이 사용되는 오브젝트에 대한 DDL 명령을 수행

 \- 진단 방법: library cache lock/pin 경합 중 x$kglk, x$kglpn, x$kglob, v$session을 통하여 Holder Session 확인, library cache lock/pin 경합 종료 후 사후 분석 시 DBA_HIST_ACTIVE_SESS_HISTORY 뷰를 통하여 Blocking Session을 확인

 \- 개선 방법: 업무시간 중 과도한 오브젝트의 변경을 제한

 

 

**library cache lock과 library cache pin의 상관관계**

```
select a.sid, kglpnmod "Mode", kglpnreq "Req"
from  x$kglpn p, v$session s
where p.kglpnuse = s.saddr
and kglpnhdl='$P1RAW';
```

 

library cache pin 이벤트의 P1=handle address, P2=lock address, P3=mode*100+namespace로 어떤 객체에 대해 어떤 모드로 락을 획득하는 과정에서 경합이 발생했는지 파악할 수 있습니다.

library cache pin은 library cache lock을 획득한 후, library cache 객체에 대해 추가작업이 필요할 "때 획득하게 됩니다. 가령 특정 프로시저나 SQL 문장을 수행하고자 하는 프로세서는 library cache lock을 Shared 모드로 획득한 후에 library cache pin을 Shared 모드로 획득해야 하며, 프로시저를 컴파일(alter procedure ... compile ...)하는 경우에는 library cache pin을 Exclusive하게 획득해야 합니다. 핀(pin)이라는 용어의 의미는 LCO에 핀을 꽂는다는 것으로, 핀이 꽂혀있는 동안 LCO의 값이 변동되지 않도록 보장받는 역할을 합니다. 한 가지 기억할 사실은 하드파싱이 발생하는 경우, 하드파싱이 이루어지는 동안 해당 SQL 커서에 대해 library cache pin을 Exclusive하게 획득한다는 것입니다. 해당 이벤트가 발생할 경우, 위의 SQL을 수행하여 핀을 점유하고 있는 세션 및 모드를 확인할 수 있습니다.

 

 

**업무시간 중 DDL의 수행을 피하라**

library cache lock 대기에 의한 성능저하 현상은 대부분 부적절한 DDL(create, alter, flush 등)에 의해 발생합니다. 따라서 트랜잭션이 왕성한 시스템에 대해서 DDL을 수행할 때는 이 내용을 충분히 수행해야 합니다. 간혹 하드 파싱이 많은 시스템에서 Shared Pool 메모리 고갈을 피하기 위해(ORA-4031 에러를 피하기 위해) flush를 수행하는 경우가 있으나 시스템에 악영향을 주는 경우가 많습니다. 하드 파싱도 나쁘지만, 하드 파싱이 발생하는 도중에 DDL을 수행하는 것은 피해야 합니다.

 

 

## **LATCH: shared pool(bind mismatch)**

------

Shared Pool 래치는 Shared Pool의 기본 메모리 구조인 힙을 보호하는 역할을 합니다. 프리 청크를 찾기 위해 프리 리스트를 탐색하고, 적절한 청크를 할당하고, 필요한 경우 프리 청크를 분할(Split)하는 일련의 작업들은 모두 Shared Pool 래치를 획득한 후에만 가능합니다. Shared Pool 래치를 획득하는 과정에서 경합이 발생하면 latch: shared pool 이벤트를 대기합니다.

 

*** Wait Time**

 이벤트의 대기시간은 기하급수적으로 증가한다.

 

*** Parameter**

 P1(프로세스가 대기하고 있는 래치의 메모리 주소), P2(래치번호), P3(래치를 획득하기 위해 프로세스가 시도한 횟수)

 

*** 일반적인 문제상황 및 대처방안**

 \- 원인: 동시에 여러 세션이 청크를 할당 받아야 하는 경우

​      Shared Pool 단편화가 일어날 경우

​      Literal SQL로 인한 Hard Parsing의 과다 수행

 \- 진단 방법: Hard Parsing 추이를 확인하기 위하여 V$SYSSTAT 뷰를 통하여 parse count(hard),

​      parse time cpu, parse time elapsed 지표 값 확인

​      Hard Parsing이 높게 나타난 세션에 수행된 SQL의 Literal SQL 여부 확인

 \- 개선 방법: 바인드 변수 사용

​      _KGHDSIDX_COUNT 히든 파라미터를 이용하여 서브풀 생성

​      Shared Pool의 크기 감소 후 dbms_shared_pool 패키지 사용

​      Cursor Sharing 기법 사용

​      Prepared Statement의 사용을 통해 JDBC PKG 내의 Literal SQL을 제거

 

 

**서브풀의 사용**

오라클 9i 이상부터는 shared Pool을 여러 개의 서브풀로 최대 7개가지 나누어서 관리할 수 있습니다. _KGHDSIDX_COUNT 히든 파라미터를 이용하면 서브풀의 개수를 관리할 수 있습니다. 오라클은 CPU 개수가 4 이상이고, Shared Pool의 크기가 250M 이상인 경우 _KGHDSIDX_COUNT의 값만큼 서브풀을 생성해서 Shared Pool을 관리합니다. 서브풀은 그 자체가 독립적인 Shared Pool로 관리되며 독자적인 프리리스트(Freelist), lru 리스트, Shared Pool 래치를 가집니다. 따라서 shared Pool의 크기가 큰 경우에는 서브풀로 쪼개서 관리함으로써 Shared Pool 래치 경합을 줄일 수 있습니다.

 

**Shared Pool 크기 감소**

하드파싱에 의해 Shared Pool 래치 경합이 발생하는 경우 또 다른 해결책은 Shared Pool의 크기를 줄이는 것입니다. Shared Pool의 크기가 줄어든 만큼 프리리스트에 딸린 프리 청크들의 개수도 감소하고 따라서 프리리스트 탐색에 소요되는 시간이 줄어들기 때문입니다. 하지만 이 경우 ORA-4031 에러가 발생할 확률이 높아지며 Shared Pool에 상주할 수 있는 객체의 수가 줄어들어서 부가적인 하드파싱이 유발될 수 있다는 단점이 있습니다. 이 단점을 해소하기 위해서 dbms_shared_pool.keep 프로시저를 이용해서 자주 사용되는 SQL 커서나 패키지, 프로시저 등을 shared Pool에 영구 상주시키는 방법을 사용할 수 있습니다. dbms_shared_pool.keep을 이용해 지정된 객체들은 Shared Pool에 영구적으로 상주하게 되며, alter system flush shared_pool 명령으로도 내려가지 않습니다. 요약하면, Shared Pool의 크기를 줄이고 동시에 dbms_shared_pool 패키지를 이용해 자주 사용되는 객체를 메모리에 상주시키는 것이 또하나의 방법이 됩니다.

 

**Cursor Sharing 사용**

Cursor Sharing 기법을 사용합니다. Cursor Sharing이란 상수(Literal)을 사용한 SQL 문장을 자동으로 바인드 변수를 사용하게끔 치환해서 커서가 공유되도록 해주는 기능을 말합니다. Curosr Sharing 기능은 기존의 Literal SQL을 바인드변수로 변환할 간적 여유가 없는 경우에만 사용하는 것이 바람직합니다.

 

 

## **kksfbc child completion**

------

SQL Cursor 객체는 Parent/Child의 관계로 이루어져 있습니다. Parent Cursor 객체가 여러개의 Child Cursor 객체를 거느립니다. Server Process가 현재 생성(Built) 중인 Child Cursor를 사용하려면 Cursor 생성이 완료될 때까지 기다려야 합니다. 이때 보고되는 이벤트가 kksfbc child completion 이벤트입니다.

 

*** 과도한 Hard Parse는 kksfbc child completion 대기 이벤트의 가장 중요한 원인**

  \1. Hard Parse가 많은 경우(Parse count 통계값 및 V$SQLAREA.VERSION_COUNT 값 참조)

  \2. Parse failure가 많은 경우

  \3. 병렬 실행이 발생하는 경우에도 Child Cursor를 공유하는 과정에서 대기가 발생하는 경우

 

*** kksfbc child completion 대기 이벤트 해소 방법(Hard Parse의 최소화)**

  \1. Dynamic SQL을 Static SQL로 변환한다.

  \2. Literal SQL 사용을 최소화하고 Bind Variable을 사용한다.

  \3. Bind Mismatch에 의한 Hard Parse가 발생하는지 확인한다.

 

 

## **CURSOR: pin s wait on x**

------

오라클 10g 이후부터, Shared Cursor Operation에 대해 기본적으로 Mutex가 동기화 객체로 사용되면서 나타나는 Wait Event입니다. 일종의 library cache pin 이벤트 발생과 성격이 비슷합니다.

 

*** Shared Cursor Operation이란?**

 \- library cache latches, library cache pin latches, library cache pinsd를 의미합니다.

 

*** 뮤텍스(Mutual exclusion)란?**

 \- 다수의 프로세스가 동일한 리소스를 공유할 때 동시 사용을 피하기 위해 사용되는 알고리즘입니다.

 \- 프로그램이 요청한 리소스를 위한 mutext를 하나 생성합니다.

 \- 시스템이 고유 ID를 부여합니다.(no wait mode의 latch와 비슷함)

 \- latch와 달리 mutext는 시스템이 관장합니다.

 \- latch의 spin을 수행하지 않기 때문에 가벼울 수 있습니다.

 \- 문제발생 oracle의 해결범위를 넘어섭니다.

 \- mutext의 경우 복구가 안됩니다.

 

뮤텍스의 기능을 이용함으로, Mutex pin을 사용하는 과정에서 이와 같은 현상이 발생하고, cursor pin s wait on x 대기가 발생합니다. 이는 library cache pin 개념과 동일합니다.

 

 

**Mutex Holder 찾기**

Oracle 10g부터 cursor: pin ...과 같은 이름의 대기 이벤트가 많이 관찰됩니다. Mutex가 동기화 객체로 사용되면서 나타난 현상인데, 이 Mutex의 문제는 Holder를 관찰하기가 쉽지 않습니다.

 

하지만 아래 Metalink Note를 읽어보면 의외로 쉽게 Mutext Holder를 찾을 수 있다는 것을 알 수 있습니다.

 ```sql
 -- 64비트 시스템에서는 8자리, 32비트 시스템에서는 4자리를 취한다.
 
 SELECT p2raw, to_number(substr(to_char(rawtohex(p2raw)), 1, 8), 'XXXXXXXX') sid
 FROM v$session
 WHERE event = 'cursor: pin S wait on X';
 
 P2RAW                 SID
 ---------------------------------- -------------
 0000001F00000000         31
 ```

31은 첫8자리 0000001F 값의 10진수 값입니다. 즉, 현재 Holder가 31번 세션이라는 것을 의미합니다. 10GR2 부터는 V$SESSION.BLOCKING_SESSION 컬럼에 Holder 정보가 기록되어 더욱 손쉽게 Holder Session을 찾을 수 있습니다.

 

 

## **latch free(simulator lru latch)**

------

DB Cache Advisor 기능이 사용하는 메모리 영역을 보호하는 latch입니다.

 *** 일반적인 문제상황 및 대처방안** 

 \- 원인: 큰 크기(수 GB이상)의 Buffer Cache를 사용 시 simulator lru latch 경함

 \- 진단 방법: DB Cache Advisor 기능이 활성화되어 있는지 확인

 \- 개선 방법: DB_CACHE_ADVICE 파라미터를 통하여 DB Caiche Advisor 기능 비활성화

 

 *** Latch란?**

Latch는 특성상 획득할 때까지 게속해서 CPU를 점유하면서 스핀하여 Latch 획득 시도를 하기 때문에 다수의 세션이 Latch 대기를 하게되면 그만큼 CPU 사용률이 증가하게 도비니다. 그래서 Latch를 해결해야하는데 Simulator lru latch는 쿼리가 수행됐을 때, 해당하는 쿼리에 대해서 일종의 Simulation을 수행해보는 것이 아니라 latch free(simulator lru latch)를 해소하기 위해서는 DB_CACHE_ADVISOR 기능을 OFF 시키면 해당 이벤트 대기현상이 해소됩니다. 만약 운영시스템이 CPU에 민감하다면 이 기능을 OFF하는 것이 좋습니다.

 

**DB_CACHE_ADVICE**

oracle 9i 부터는 SGA 영역 크기를 온라인 상태에서 바꿀 수 있습니다. 이를 Dynamic SGA 기능이라고 합니다. 이렇게 바꿀 수 있는 메모리 영역은 Shared Pool, Buffer Cache, Large Pool 이렇게 세 가지입니다. 이 중 Buffer Cache 크기를 조절했을 때의 성능을 예측하는 Advisory 기능을 DB_CACHE_ADVICE 파라미터를 통하여 제공합니다. DB_CACHE_ADVICE = ON인 경우 Buffer Cache Advisory 기능이 enable 되며 V$DB_Cache_Advice 뷰를 통하여 내용을 확인할 수 있습니다. V$DB_Cache_Advice View에는 buffer cache 별로 현재 크기의 10%에서 200%까지 20개의 크기에 대한 simulation 정보를 기록합니다. 각 크기별로 기존 block 참조 정보를 이용해서 예상되는 물리ㅏ적 일기 수를 제공합니다.

 

 *** Buffer Cache Advisory 기능 사용은 다음 두가지의 오버헤드를 일으킵니다.**

  \1) Advisory 기능은 buffer cache 별로 bookkeeping을 위한 아주 약간의 CPU 오버헤드가 필요하다.

  \2) MEMORY: Advisory 기능은 buffer block 당 Shared Pool에서 약 700 byte 정도의 메모리를 할당한다.

 

 *** parameter는 ON, OFF, READY 세 가지 값을 가질 수 있는데, 각 상태의 의미는 다음과 같습니다.**

  \1) OFF: Advisory 기능이 disable 되고, CPU나 MEMORY 오버헤드가 없음

  \2) ON: Advisory 기능이 enable 되고, CPU나 MEMORY 오버헤드가 발생

  \3) READY: Advisory 기능은 disable되나, Shared Pool의 메모리는 할당

 

READY나 ON의 경우, Shared Pool의 Contention이 발생하므로 오버헤드가 될 수 있습니다. 충분한 여유공간을 확인한 후 작업해야 합니다.

 

 

**V$DB_CACHE_ADVICE 뷰**

 id: Buffer Cache의 id(1~8)

 name: Buffer Cache의 이름

 block size: Buffer Cache의 block 크기

 advice_status: Buffer Cache Advisory 기능의 상태(ON or OFF: Ready 상태도 OFF로 표시)

 size_for_estimate: simulation에 사용한 Buffer Cache의 크기(KB)

 buffers_for_estimate: simulation에 사용한 Buffer Cache의 개수(blocks)

 estd_physical_read_factor: 물리적 읽기 예상#/Buffer Cache 일기#

 estd_physical_reads: 물리적 읽기 예상치

 

estd_physical_read_factor는, 실제 Buffer Cache Advisory 기능을 enable 시킨 이후, Buffer Cache에 실제 발생한 physical read number 대비, Buffer Cache의 크기를 V$DB_CACHE_ADVICE 뷰의 row에 나와 있는 크기로 조정했을 때 예상되는 physical read number(estd_physical_reads)의 비율을 의미합니다.



출처: https://12bme.tistory.com/313?category=749950 [길은 가면, 뒤에 있다.]