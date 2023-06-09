# 예제 - 음식배달

![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

본 예제는 마이크로서비스응용및활용 과목의 기말고사 대체용으로 마이크로서비스를 직접 구현하고 쿠버네티스에 배포/운영한 결과 리포트 작성을 위한 가이드입니다.

# Table of contents

# 서비스 시나리오

요구사항

1. 고객이 대여점의 자동차를 선택하여 렌탈한다.
1. 고객이 자동차 렌탈을 취소할 수 있다.
1. 대여점은 고객의 렌탈 여부를 선택한다.
1. 고객은 자동차 렌탈 여부를 조회한다.
1. 렌탈 시작 시간이 되었을 때 부터 자동차를 픽업한다.
1. 렌탈 종료 시간이 되기 전 까지 자동차를 반납한다.
1. 고객이 결제한다.
1. 고객은 결제를 취소할 수 있다.
1. 자동차 렌탈 상태가 변경될 때 마다, 카카오톡 또는 문자 알림을 보낸다.

# 체크포인트

- 분석 설계
    - 이벤트스토밍:
        - 스티커 색상별 객체의 의미를 제대로 이해하여 헥사고날 아키텍처와의 연계 설계에 적절히 반영하고 있는가?
        - 각 도메인 이벤트가 의미있는 수준으로 정의되었는가?
        - 어그리게잇: Command와 Event 들을 ACID 트랜잭션 단위의 Aggregate 로 제대로 묶었는가?
        - 기능적 요구사항과 비기능적 요구사항을 누락 없이 반영하였는가?

    - 서브 도메인, 바운디드 컨텍스트 분리
        - 팀별 KPI 와 관심사, 상이한 배포주기 등에 따른 Sub-domain 이나 Bounded Context 를 적절히 분리하였고 그 분리 기준의 합리성이 충분히 설명되는가?
            - 적어도 3개 이상 서비스 분리
        - 폴리글랏 설계: 각 마이크로 서비스들의 구현 목표와 기능 특성에 따른 각자의 기술 Stack 과 저장소 구조를 다양하게 채택하여 설계하였는가?
        - 서비스 시나리오 중 ACID 트랜잭션이 크리티컬한 Use 케이스에 대하여 무리하게 서비스가 과다하게 조밀히 분리되지 않았는가?
    - 컨텍스트 매핑 / 이벤트 드리븐 아키텍처
        - 업무 중요성과 도메인간 서열을 구분할 수 있는가? (Core, Supporting, General Domain)
        - Request-Response 방식과 이벤트 드리븐 방식을 구분하여 설계할 수 있는가?
        - 장애격리: 서포팅 서비스를 제거 하여도 기존 서비스에 영향이 없도록 설계하였는가?
        - 신규 서비스를 추가 하였을때 기존 서비스의 데이터베이스에 영향이 없도록 설계(열려있는 아키택처)할 수 있는가?
        - 이벤트와 폴리시를 연결하기 위한 Correlation-key 연결을 제대로 설계하였는가?

    - 헥사고날 아키텍처
        - 설계 결과에 따른 헥사고날 아키텍처 다이어그램을 제대로 그렸는가?

- 구현
    - [DDD] 분석단계에서의 스티커별 색상과 헥사고날 아키텍처에 따라 구현체가 매핑되게 개발되었는가?
        - Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 데이터 접근 어댑터를 개발하였는가
        - [헥사고날 아키텍처] REST Inbound adaptor 이외에 gRPC 등의 Inbound Adaptor 를 추가함에 있어서 도메인 모델의 손상을 주지 않고 새로운 프로토콜에 기존 구현체를
          적응시킬 수 있는가?
        - 분석단계에서의 유비쿼터스 랭귀지 (업무현장에서 쓰는 용어) 를 사용하여 소스코드가 서술되었는가?
    - Request-Response 방식의 서비스 중심 아키텍처 구현
        - 마이크로 서비스간 Request-Response 호출에 있어 대상 서비스를 어떠한 방식으로 찾아서 호출 하였는가? (Service Discovery, REST, FeignClient)
        - 서킷브레이커를 통하여 장애를 격리시킬 수 있는가?
    - 이벤트 드리븐 아키텍처의 구현
        - 카프카를 이용하여 PubSub 으로 하나 이상의 서비스가 연동되었는가?
        - Correlation-key:  각 이벤트 건 (메시지)가 어떠한 폴리시를 처리할때 어떤 건에 연결된 처리건인지를 구별하기 위한 Correlation-key 연결을 제대로 구현 하였는가?
        - Message Consumer 마이크로서비스가 장애상황에서 수신받지 못했던 기존 이벤트들을 다시 수신받아 처리하는가?
        - Scaling-out: Message Consumer 마이크로서비스의 Replica 를 추가했을때 중복없이 이벤트를 수신할 수 있는가
        - CQRS: Materialized View 를 구현하여, 타 마이크로서비스의 데이터 원본에 접근없이(Composite 서비스나 조인SQL 등 없이) 도 내 서비스의 화면 구성과 잦은 조회가
          가능한가?

- 운영
    - SLA 준수
        - 셀프힐링: Liveness Probe 를 통하여 어떠한 서비스의 health 상태가 지속적으로 저하됨에 따라 어떠한 임계치에서 pod 가 재생되는 것을 증명할 수 있는가?
        - 서킷브레이커, 레이트리밋 등을 통한 장애격리와 성능효율을 높힐 수 있는가?
        - 오토스케일러 (HPA) 를 설정하여 확장적 운영이 가능한가?
        - 모니터링, 앨럿팅:
    - 무정지 운영 CI/CD (10)
        - Readiness Probe 의 설정과 Rolling update을 통하여 신규 버전이 완전히 서비스를 받을 수 있는 상태일때 신규버전의 서비스로 전환됨을 siege 등으로 증명

# 분석/설계

## AS-IS 조직 (Horizontally-Aligned)

![image](https://user-images.githubusercontent.com/487999/79684144-2a893200-826a-11ea-9a01-79927d3a0107.png)

## TO-BE 조직 (Vertically-Aligned)

![image](https://user-images.githubusercontent.com/487999/79684159-3543c700-826a-11ea-8d5f-a3fc0c4cad87.png)

## Event Storming 결과

![스크린샷 2023-06-09 오후 1 53 39](https://github.com/acmexii/sjcu-microservice-labs/assets/49636918/2e52a63f-8e98-4651-86e4-c3cfa50cced1)

# 구현:

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다.

```
cd Rental
mvn spring-boot:run

cd Payment
mvn spring-boot:run 

cd RentalShop
mvn spring-boot:run  

cd Notification
mvn spring-boot:run

cd Stock
mvn spring-boot:run
```

각 마이크로 서비스들의 포트 번호는 아래와 같다.

- **Rental**: 8082
- **Payment**: 8083
- **RentalShop**: 8085
- **Notification**: 8088
- **Stock**: 8092

## DDD 의 적용

- 각 서비스내에 도출된 핵심 Aggregate Root 객체를 Entity 로 선언하였다: (예시는 Payment 마이크로 서비스).
  이때 가능한 현업에서 사용하는 언어 (유비쿼터스 랭귀지)를 그대로 사용하려고 노력했다.

```
package rentalshop.domain;

import javax.persistence.*;
import lombok.Data;

@Entity
@Table(name = "Payment_table")
@Data
public class Payment {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private Long rentalId;
    private Integer price;


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public Long getRentalId() {
        return rentalId;
    }

    public void setRentalId(Long rentalId) {
        this.rentalId = rentalId;
    }


    public Integer getPrice() {
        return price;
    }

    public void setPrice(Integer price) {
        this.price = price;
    }
    
}

```

- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기
  위하여 Spring Data REST 의 RestRepository 를 적용하였다

```
package rentalshop.domain;

import org.springframework.data.repository.PagingAndSortingRepository;

```

- 적용 후 REST API 의 테스트

```
# Rental 서비스의 대여처리
http POST :8082/rentals customerId=1 rentalShopId=1 stockId=1 rentalStatus="Started" qty=1 startedAt="2023-06-09" endedAt="2023-06-10" 

# RentalShop의 고객 대여 Accept
http PUT :8085/rentalShops/1/accept

# RentalShop의 Stock 등록
http POST :8092/stocks rentalShopId=1 carName="320D" carType="SUB" qty=3 price=230000

# 고객의 결제 시도
http POST :8085/payments rentalId=1 price=230000

```

## 비동기식 호출 / 시간적 디커플링 / 장애격리(Fault Isolation) / 최종 (Eventual) 일관성 테스트

고객의 대여 신청이 이루어지고, 대여 신청 여부를 대여점에 알려주는 행위를 동기식(Synchronized)이 아닌 비 동기식(ASynchronized)으로 처리하여,
대여 신청이 블로킹 되지 않도록 처리한다.

- 이를 위하여 고객이 대여를 신청하였을 때, 도메인 이벤트를 카프카로 송출(Publish)한다.

```
package rentalshop.domain;

@Entity
@Table(name = "Rental_table")
@Data
public class Rental {

    @PrePersist
    public void onPrePersist(){
        CarRented carRented = new CarRented(this);
        carRented.publishAfterCommit();
    }

}
```

- 대여점에서는 고객이 대여를 신청하였다는 도메인 이벤트를 카프카로 수신(Subscribe)한다.

```
package fooddelivery;

...

@Service
@Transactional
public class PolicyHandler{

    @StreamListener(
        value = KafkaProcessor.INPUT,
        condition = "headers['type']=='CarRented'"
    )
    public void wheneverCarRented_CreateRental(@Payload CarRented carRented) {
        CarRented event = carRented;
        System.out.println(
            "\n\n##### listener CreateRental : " + carRented + "\n\n"
        );

        // Sample Logic //
        RentalShop.createRental(event);
    }

}

```

대여점 시스템은 결제(Payment) 마이크로서비스와 분리되어 있고, 이벤트 수신에 따라 처리되기 때문에,
결제 시스템이 오류가 발생한 상태더라도, 고객의 차량 반납, 렌탈 취소와 같은 행위는 문제가 발생하지 않는다.

```

# 1. Rental 서비스의 대여처리
http POST :8082/rentals customerId=1 rentalShopId=1 stockId=1 rentalStatus="Started" qty=1 startedAt="2023-06-09" endedAt="2023-06-10"
http POST :8082/rentals customerId=1 rentalShopId=1 stockId=2 rentalStatus="Started" qty=1 startedAt="2023-06-09" endedAt="2023-06-10"

# 2. 결제 서비스 (Payment)를 종료 (ctrl+c)

# 3. RentalShop의 고객 대여 Accept
http PUT :8085/rentalShops/1/accept

# 4. 고객의 대여 취소
http PUT :8082/rentals/2/cancel

```

# 운영

## 오토스케일 아웃

앞서 CB 는 시스템을 안정되게 운영할 수 있게 해줬지만 사용자의 요청을 100% 받아들여주지 못했기 때문에 이에 대한 보완책으로 자동화된 확장 기능을 적용하고자 한다.

- 결제서비스에 대한 replica 를 동적으로 늘려주도록 HPA 를 설정한다. 설정은 CPU 사용량이 15프로를 넘어서면 replica 를 10개까지 늘려준다:

```
kubectl autoscale deploy pay --min=1 --max=10 --cpu-percent=15
```

- CB 에서 했던 방식대로 워크로드를 2분 동안 걸어준다.

```
siege -c100 -t120S -r10 --content-type "application/json" 'http://localhost:8081/orders POST {"item": "chicken"}'
```

- 오토스케일이 어떻게 되고 있는지 모니터링을 걸어둔다:

```
kubectl get deploy pay -w
```

- 어느정도 시간이 흐른 후 (약 30초) 스케일 아웃이 벌어지는 것을 확인할 수 있다:

```
NAME    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
pay     1         1         1            1           17s
pay     1         2         1            1           45s
pay     1         4         1            1           1m
:
```

- siege 의 로그를 보아도 전체적인 성공률이 높아진 것을 확인 할 수 있다.

```
Transactions:		        5078 hits
Availability:		       92.45 %
Elapsed time:		       120 secs
Data transferred:	        0.34 MB
Response time:		        5.60 secs
Transaction rate:	       17.15 trans/sec
Throughput:		        0.01 MB/sec
Concurrency:		       96.02
```

## 무정지 재배포

* 먼저 무정지 재배포가 100% 되는 것인지 확인하기 위해서 Autoscaler 이나 CB 설정을 제거함

- seige 로 배포작업 직전에 워크로드를 모니터링 함.

```
siege -c1 -t600S -r10 --content-type "application/json" 'http://localhost:8081/orders POST {"item": "chicken"}'

** SIEGE 4.0.5
** Preparing 100 concurrent users for battle.
The server is now under siege...

HTTP/1.1 201     0.68 secs:     207 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 201     0.68 secs:     207 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 201     0.70 secs:     207 bytes ==> POST http://localhost:8081/orders
HTTP/1.1 201     0.70 secs:     207 bytes ==> POST http://localhost:8081/orders
:

```

- 새버전으로의 배포 시작

```
kubectl set image ...
```

- seige 의 화면으로 넘어가서 Availability 가 100% 미만으로 떨어졌는지 확인

```
Transactions:		        3078 hits
Availability:		       70.45 %
Elapsed time:		       120 secs
Data transferred:	        0.34 MB
Response time:		        5.60 secs
Transaction rate:	       17.15 trans/sec
Throughput:		        0.01 MB/sec
Concurrency:		       96.02

```

배포기간중 Availability 가 평소 100%에서 70% 대로 떨어지는 것을 확인. 원인은 쿠버네티스가 성급하게 새로 올려진 서비스를 READY 상태로 인식하여 서비스 유입을 진행한 것이기 때문. 이를 막기위해
Readiness Probe 를 설정함:

```
# deployment.yaml 의 readiness probe 의 설정:


kubectl apply -f kubernetes/deployment.yaml
```

- 동일한 시나리오로 재배포 한 후 Availability 확인:

```
Transactions:		        3078 hits
Availability:		       100 %
Elapsed time:		       120 secs
Data transferred:	        0.34 MB
Response time:		        5.60 secs
Transaction rate:	       17.15 trans/sec
Throughput:		        0.01 MB/sec
Concurrency:		       96.02

```

배포기간 동안 Availability 가 변화없기 때문에 무정지 재배포가 성공한 것으로 확인됨.
