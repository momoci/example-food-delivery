![image](https://user-images.githubusercontent.com/487999/79708354-29074a80-82fa-11ea-80df-0db3962fb453.png)

# 예제 - 음식배달

본 예제는 MSA/DDD/Event Storming/EDA 를 포괄하는 분석/설계/구현/운영 전단계를 커버하도록 구성한 예제입니다.
이는 클라우드 네이티브 애플리케이션의 개발에 요구되는 체크포인트들을 통과하기 위한 예시 답안을 포함합니다.
- 체크포인트 : https://workflowy.com/s/assessment-check-po/T5YrzcMewfo4J6LW

# 서비스 시나리오

배달의 민족 커버하기 - https://1sung.tistory.com/106

기능적 요구사항
1. 고객이 메뉴를 선택하여 주문한다.
2. 고객이 선택한 메뉴에 대해 결제한다.
3. 주문이 되면 주문 내역이 입점상점주인에게 주문정보가 전달된다
4. 상점주는 주문을 수락하거나 거절할 수 있다
5. 상점주는 요리시작때와 완료 시점에 시스템에 상태를 입력한다
6. 고객은 아직 요리가 시작되지 않은 주문은 취소할 수 있다
7. 요리가 완료되면 고객의 지역 인근의 라이더들에 의해 배송건 조회가 가능하다
8. 라이더가 해당 요리를 pick 한후, pick했다고 앱을 통해 통보한다.
9. 고객이 주문상태를 중간중간 조회한다
10. 주문상태가 바뀔 때 마다 카톡으로 알림을 보낸다
11. 고객이 요리를 배달 받으면 배송확인 버튼을 탭하여, 모든 거래가 완료된다

추가사항
1. 주문이 접수되었을때 상점주인이 설정해놓은 주문수용량을 확인하고 초과될 시 자동거절된다.
2. 고객은 아직 요리가 시작이 되지 않은 주문의 메뉴 변경이 가능하다.

![image](https://user-images.githubusercontent.com/118874947/205822724-7da28593-a720-486e-8fec-104544727583.png)


# 체크포인트
1. Saga (Pub / Sub)
2. CQRS
3. Compensation / Correlation
4. Request / Response
5. Circuit Breaker
6. Gateway / Ingress




# Saga (Pub / Sub)

사용자에 의해 주문상태가 변경되면 각 이벤트에 맞는 동작이 수행된다.

```
gitpod /workspace/delivery (main) $ http :8081/orderLists foodId="피자" address="111" status="주문접수중"
HTTP/1.1 201 
Connection: keep-alive
Content-Type: application/json
Date: Tue, 06 Dec 2022 04:52:33 GMT
Keep-Alive: timeout=60
Location: http://localhost:8081/orderLists/1
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "_links": {
        "orderList": {
            "href": "http://localhost:8081/orderLists/1"
        },
        "self": {
            "href": "http://localhost:8081/orderLists/1"
        }
    },
    "address": "111",
    "customerId": null,
    "foodId": "피자",
    "status": "주문접수중"
}
```

```
gitpod /workspace/delivery (main) $ http :8082/foodCookings
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/hal+json
Date: Tue, 06 Dec 2022 04:53:05 GMT
Keep-Alive: timeout=60
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "_embedded": {
        "foodCookings": [
            {
                "_links": {
                    "accept": {
                        "href": "http://localhost:8082/foodCookings/1/accept"
                    },
                    "foodCooking": {
                        "href": "http://localhost:8082/foodCookings/1"
                    },
                    "self": {
                        "href": "http://localhost:8082/foodCookings/1"
                    }
                },
                "address": "111",
                "foodId": "피자",
                "orderCapacity": null,
                "orderCount": null,
                "orderId": 1,
                "status": "주문접수중"
            }
        ]
    },
    "_links": {
        "profile": {
            "href": "http://localhost:8082/profile/foodCookings"
        },
        "self": {
            "href": "http://localhost:8082/foodCookings"
        }
    },
    "page": {
        "number": 0,
        "size": 20,
        "totalElements": 1,
        "totalPages": 1
    }
}
```

# CQRS
주문 진행 상태가 변경되면 Mypage의 상태가 변하는지 확인할 수 있다.

![image](https://user-images.githubusercontent.com/118874947/205825675-262521af-aea3-4e52-9962-fe3865ae3aea.png)

```
gitpod /workspace/delivery (main) $ http :8084/myPages
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/hal+json
Date: Tue, 06 Dec 2022 05:34:21 GMT
Keep-Alive: timeout=60
Transfer-Encoding: chunked
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers

{
    "_embedded": {
        "myPages": [
            {
                "_links": {
                    "myPage": {
                        "href": "http://localhost:8084/myPages/1"
                    },
                    "self": {
                        "href": "http://localhost:8084/myPages/1"
                    }
                },
                "status": "주문접수"
            },
            {
                "_links": {
                    "myPage": {
                        "href": "http://localhost:8084/myPages/2"
                    },
                    "self": {
                        "href": "http://localhost:8084/myPages/2"
                    }
                },
                "status": "주문승인"
            }
        ]
    },
    "_links": {
        "profile": {
            "href": "http://localhost:8084/profile/myPages"
        },
        "self": {
            "href": "http://localhost:8084/myPages"
        }
    },
    "page": {
        "number": 0,
        "size": 20,
        "totalElements": 2,
        "totalPages": 1
    }
}
```

# Compensation / Correlation

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd app
mvn spring-boot:run

cd pay
mvn spring-boot:run 

cd store
mvn spring-boot:run  

cd customer
python policy-handler.py 
```

# Request / Response

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd app
mvn spring-boot:run

cd pay
mvn spring-boot:run 

cd store
mvn spring-boot:run  

cd customer
python policy-handler.py 
```

# Circuit Breaker

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd app
mvn spring-boot:run

cd pay
mvn spring-boot:run 

cd store
mvn spring-boot:run  

cd customer
python policy-handler.py 
```

# Gateway / Ingress

분석/설계 단계에서 도출된 헥사고날 아키텍처에 따라, 각 BC별로 대변되는 마이크로 서비스들을 스프링부트와 파이선으로 구현하였다. 구현한 각 서비스를 로컬에서 실행하는 방법은 아래와 같다 (각자의 포트넘버는 8081 ~ 808n 이다)

```
cd app
mvn spring-boot:run

cd pay
mvn spring-boot:run 

cd store
mvn spring-boot:run  

cd customer
python policy-handler.py 
```
