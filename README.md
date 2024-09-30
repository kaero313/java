<h1>MQ를 이용한 대량의 데이터 처리</h1>

<h3>1. 도입 목적</h3>

- 비동기, 확장성, 보증성
- 부하 분산 처리
- 데이터 손실 방지

<br/>

<h3>2. 기존 처리 방식</h3>

![image](https://github.com/user-attachments/assets/e01c9def-7943-4684-b7a2-4242f0e065b7)
> 요청이 들어오면 중간 과정 없이 그대로 서버가 받아서 처리하는 구조
- 처리해야 할 데이터가 많아지니 과부하 발생
- 그에 따라 정상적으로 처리되지않고 유실되는 데이터 발생

<br/>

<h3>3. 개선 방안</h3>

![111123](https://github.com/user-attachments/assets/6ecc9f67-4951-4ba6-9bc3-3935d626cec1)

> AMQP 도입, 중간에서 분산 처리하는 구조로 개선
- 처리해야 할 데이터를 분산해줌으로써, 프로세스 안정화
- 결론적으로, 전송된 메세지의 보장성 제공
  
<br/>

<h3>4. 테스트 수행</h3>

<h4>1. Apache JMeter을 사용하여 부하 테스트 진행</h4>
- 초당 5000건, 60초간 총 300,000건의 Request 

<br/>
<br/>

<h4>2. AMQP 도입 전</h4>

![image](https://github.com/user-attachments/assets/ce7301a9-1e7a-47f4-ae16-1fe103d518e6)
> 프로세스 과부하로 인하여 요청 전체를 처리하지 못 하고 Timeout이 발생한 데이터가 생김
  
<br/>

<h4>3. AMQP 도입 후</h4>

![image](https://github.com/user-attachments/assets/cd4f2f4c-2210-4828-ad8c-a11de964364c)
> 리스닝중인 프로세스가 과부하가 발생하지 않게 관리하여 유실된 데이터 없이 전체 처리 완료

<br/>

<h1>멀티 스레드와 스레드 풀</h1>

<h3>1. 도입 목적</h3>

- MQ로 부하 분산 처리를 하였지만 허용량 이상의 요청이 들어오거나,<br/>
  로직 자체가 복잡하여 수행 시간이 오래 걸린다면 결론적으로 과부하가 발생하는 문제 개선

<br/>

<h3>2. 기존 처리 방식</h3>

![image](https://github.com/user-attachments/assets/ea45ae00-d08e-48e4-b9b5-1ae7d14e7ed3)
> 스레드를 하나만 사용하는 싱글 스레드 방식
- 다른 요청에 의하여 기존 스레드의 처리가 지연되면 나머지 요청의 처리도 같이 지연되는 현상 발생 

<br/>

<h3>3-1. 개선 방안 ①</h3>

![image](https://github.com/user-attachments/assets/20f20fb0-3af0-4b4c-95f3-247284a19b9d)
> 스레드의 개수를 늘리는 멀티 스레드 방식
- 장점
  - 하나의 스레드 처리가 지연되어도, 다른 스레드는 정상 동작하여 다중 요청 처리 가능
  - 프로세스간 자원을 공유하기 때문에 효율적인 자원 관리 가능
- 단점
  - 프로세스간 자원을 공유하기 때문에 동기화 문제가 발생할 수 있음(데이터의 일관성을 유지하기 위해 추가 작업 필요)
  - 스레드의 무분별한 생성은 시스템 성능에 영향을 끼침(평균 요청량 및 응답시간 등 철저하게 분석을 하여 적정량의 스레드 생성이 필요)

<br/>

<h3>3-2. 개선 방안 ②</h3>

![image](https://github.com/user-attachments/assets/d22ed1a5-6cce-4983-a1c8-e3bbdce3db4d)
> 미리 스레드를 생성해놓는 스레드 풀 방식<br/>
(생성된 스레드는 대기상태에 놓여있다가 요청이 발생하면 할당하여 해당 요청을 처리한 후 다시 대기상태로 돌아가는 방식)
- 장점
  - 정해진 개수의 스레드를 미리 생성하고 관리하기 때문에 스레드 생성/삭제시 발생하는 자원의 소모가 절약됨
  - 과도한 요청이 발생해도 설정해놓은 개수의 스레드만 생성되기 때문에 시스템의 부하가 줄어듦
- 단점
  - 요청량에 비해 스레드 풀의 개수가 과도하게 많다면 이는 성능저하로 이어짐(시스템에 대한 이해를 바탕으로 적정량의 스레드 풀 설정)
  - 생성가능한 개수가 정해져 있기 때문에 모든 스레드가 사용중이라면 다른 요청들은 지연될 수 있음(사용량 분석)
    
<br/>

<h3>4-1. 테스트 수행(과부하의 경우)</h3>

<h4>1. Apache JMeter을 사용하여 부하 테스트 진행</h4>
- 초당 5000건, 10초간 총 50,000건의 Request<br/>
- 반복문을 사용하여 카운트를 출력하는 간단한 로직 수행

<br/>
<br/>

<h4>2. 싱글 스레드</h4>

![image](https://github.com/user-attachments/assets/353b644f-12b8-4922-be68-f529605267fd)
> 평균 수행 시간 '3041'<br/>
(Request 1건당 0 ~ 1000까지 반복하여 카운트를 출력하는 로직 수행)

<br/>

<h4>3. 멀티 스레드</h4>

![image](https://github.com/user-attachments/assets/7da4d149-126c-40e9-bd55-1dd033e8c354)
> 평균 수행 시간 '865'<br/>
(Request 1건당 0 ~ 1000까지를 5개로 나누어 반복하여 카운트를 출력하는 로직 수행)

<br/>

<h4>4. 스레드 풀</h4>

![image](https://github.com/user-attachments/assets/4f094dbd-0152-4ea4-b17f-543884c397bb)
> 평균 수행 시간 '801'<br/>
(Request 1건당 0 ~ 1000까지를 반복하여 카운트를 출력, 스레드 풀의 개수는 5 ~ 10개로 설정)

<br/>

<h4>5. 테스트 결과</h4>

- 처리해야 할 요청이 급증할수록 싱글 스레드보다는 멀티 스레드, 스레드 풀이 평균 처리 속도 향상에 유의미한 효과가 있었음

<h3>4-2. 테스트 수행(부하가 적은 경우)</h3>

<h4>1. Apache JMeter을 사용하여 부하 테스트 진행</h4>
- 초당 100건, 50초간 총 5,000건의 Request<br/>
- 반복문을 사용하여 카운트를 출력하는 간단한 로직 수행

<br/>
<br/>

<h4>2. 싱글 스레드</h4>

![image](https://github.com/user-attachments/assets/80fb4fb1-af7a-4501-aea1-95c3998a086c)
> 평균 수행 시간 '247'</br>
(Request 1건당 0 ~ 1000까지 반복하여 카운트를 출력하는 로직 수행)

<br/>

<h4>3. 멀티 스레드</h4>

![image](https://github.com/user-attachments/assets/740523ef-fa8f-4e72-9093-8a05427b11cb)
> 평균 수행 시간 '252'<br/>
(Request 1건당 0 ~ 1000까지를 5개로 나누어 반복하여 카운트를 출력하는 로직 수행)

<br/>

<h4>4. 스레드 풀</h4>

![image](https://github.com/user-attachments/assets/f9cb1bfc-5f34-472c-9c6c-fa1580b3160b)
> 평균 수행 시간 '254'<br/>
(Request 1건당 0 ~ 1000까지를 반복하여 카운트를 출력, 스레드 풀의 개수는 5 ~ 10개로 설정)

<br/>

<h4>5. 테스트 결과</h4>

- 처리해야 할 요청이 적다면 셋 다 평균 처리 속도에 별 차이가 없음

<br/>

<h3>5. 결론</h3>

필요한 수준에 비하여 과하게 스레드를 설정한다면 이는 곧 낭비로 이어지며,<br/>
반대의 경우는 처리가 지연되어 시스템에 영향을 끼치는 현상이 발생할 수 있음<br/><br/>
그러므로, 시스템 성능 / 서비스 성능 / 해당 프로세스의 성능 및 요청량 등<br/>
종합적인 요소를 파악하여 가장 적합한 환경을 구축하는 것이 필요함

----

<h3>6. Kafka란?</h3>

![image](https://github.com/user-attachments/assets/57d23d78-f967-4a6f-9d03-7807a33366f4)
> 카프카 공식 문서에 나오는 간단한 아키텍처. 클러스터를 중심으로 프로듀서와 컨슈머가 데이터를 push하고 pull하는 구조이다.

<br/>

- 아파치 소프트웨어 재단이 스칼라 언어로 개발한 오픈 소스 메시지 브로커 프로젝트이다
- 대량의 데이터를 안정적으로 관리하기 위해 높은 처리량 및 낮은 지연시간을 지닌 플랫폼으로 설계되었다
- 여러 시스템 간에 데이터를 신속하게 전송하는 데 사용된다
- pub/sub 메시지 큐로 정의할 수 있으며, 분산 트랜잭션 로그로 구성되어 확장이 가능하다
- 대규머 데이터 처리에 적합하며, 특히 대용량의 로그 데이터를 수집하고 분석하는 데 유용하다

<br/>

<h4>6-1. Cluster</h4>

![image](https://github.com/user-attachments/assets/3723372e-d2c9-41f0-adb3-7e90baa0f2c3)
> 카프카는 클러스터 > 브로커 > 토픽 > 파티션 > 세그먼트로 구성되어 있으며 주키퍼는 클러스터를 관리한다
