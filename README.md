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
주문을 하면 paymentStatus에 상태가 false로 저장되고 OrderCancel을 하면 상태가 true 변경된다.

```
gitpod /workspace/delivery (main) $ http :8081/orderLists foodId="짜장면" address="222" tatus="주문접수중"
HTTP/1.1 201 
Connection: keep-alive
Content-Type: application/json
Date: Tue, 06 Dec 2022 06:56:00 GMT
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
    "address": "222",
    "customerId": null,
    "foodId": "짜장면",
    "status": null
}
```
```
"paymentStatuses": [
            {
                "_links": {
                    "cancelpayment": {
                        "href": "http://localhost:8081/paymentStatuses/1/cancelpayment"
                    },
                    "paymentStatus": {
                        "href": "http://localhost:8081/paymentStatuses/1"
                    },
                    "self": {
                        "href": "http://localhost:8081/paymentStatuses/1"
                    }
                },
                "cancel": false,
                "orderId": 1
            }
        ]
```
```
gitpod /workspace/delivery (main) $ http DELETE :8081/orderLists/1
HTTP/1.1 204 
Connection: keep-alive
Date: Tue, 06 Dec 2022 06:37:42 GMT
Keep-Alive: timeout=60
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
```
```
"paymentStatuses": [
            {
                "_links": {
                    "cancelpayment": {
                        "href": "http://localhost:8081/paymentStatuses/1/cancelpayment"
                    },
                    "paymentStatus": {
                        "href": "http://localhost:8081/paymentStatuses/1"
                    },
                    "self": {
                        "href": "http://localhost:8081/paymentStatuses/1"
                    }
                },
                "cancel": true,
                "orderId": 1
            }
        ]
```

# Request / Response
주문 취소를 하면 상태를 확인 후 취소 절차가 진행된다

```
@PreRemove
    public void onPreRemove(){
        if ("주문접수중".equals(status) || "주문접수".equals(status) || "주문승인".equals(status)) {
            delivery.external.CancelPaymentCommand cancelPaymentCommand = new delivery.external.CancelPaymentCommand();
            cancelPaymentCommand.setCancel(true);
            // // mappings goes here
            OrderApplication.applicationContext.getBean(delivery.external.PaymentStatusService.class)
                .cancelPayment(getId(), cancelPaymentCommand);

            OrderCanceled orderCanceled = new OrderCanceled(this);
            orderCanceled.publishAfterCommit();
        } else {
            throw new RuntimeException("Order Status : " + status);
        }        
    }
```
```
@RequestMapping(value = "paymentStatuses/{id}/cancelpayment",
        method = RequestMethod.PUT,
        produces = "application/json;charset=UTF-8")
    public PaymentStatus cancelPayment(@PathVariable(value = "id") Long id, @RequestBody CancelPaymentCommand cancelPaymentCommand, HttpServletRequest request, HttpServletResponse response) throws Exception {
            System.out.println("##### /paymentStatus/cancelPayment  called #####");
            Optional<PaymentStatus> optionalPaymentStatus = paymentStatusRepository.findById(id);
            
            optionalPaymentStatus.orElseThrow(()-> new Exception("No Entity Found"));
            PaymentStatus paymentStatus = optionalPaymentStatus.get();
            paymentStatus.cancelPayment(cancelPaymentCommand);
            
            paymentStatusRepository.save(paymentStatus);
            return paymentStatus;
            
    }
```

# Circuit Breaker



# Gateway / Ingress

```
server:
  port: 8088

---

spring:
  profiles: default
  cloud:
    gateway:
      routes:
        - id: order
          uri: http://localhost:8081
          predicates:
            - Path=/orderLists/**, /paymentStatuses/**, 
        - id: store
          uri: http://localhost:8082
          predicates:
            - Path=/foodCookings/**, 
        - id: rider
          uri: http://localhost:8083
          predicates:
            - Path=/deliveries/**, 
        - id: customer
          uri: http://localhost:8084
          predicates:
            - Path=, /myPages/**
        - id: frontend
          uri: http://localhost:8080
          predicates:
            - Path=/**
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins:
              - "*"
            allowedMethods:
              - "*"
            allowedHeaders:
              - "*"
            allowCredentials: true
```

# 추가사항 1
주문이 접수되었을때 상점주인이 설정해놓은 주문수용량을 확인하고 초과될 시 자동거절된다.


# 추가사항 2
고객은 아직 요리가 시작이 되지 않은 주문의 메뉴 변경이 가능하다.
```
gitpod /workspace/delivery (main) $ http :8081/orderLists foodId="피자" address="111" status="주문승인"
HTTP/1.1 201 
Connection: keep-alive
Content-Type: application/json
Date: Tue, 06 Dec 2022 07:57:28 GMT
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
    "status": "주문승인"
}
```

```
gitpod /workspace/delivery (main) $ http :8082/foodCookings
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/hal+json
Date: Tue, 06 Dec 2022 07:57:32 GMT
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
                "status": "주문승인"
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

```
gitpod /workspace/delivery (main) $ http PATCH :8081/orderLists/1 foodId="짜장" address="222" status="주문접수중"
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/json
Date: Tue, 06 Dec 2022 07:57:53 GMT
Keep-Alive: timeout=60
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
    "address": "222",
    "customerId": null,
    "foodId": "짜장",
    "status": "주문접수중"
}
```

```
gitpod /workspace/delivery (main) $ http :8082/foodCookings
HTTP/1.1 200 
Connection: keep-alive
Content-Type: application/hal+json
Date: Tue, 06 Dec 2022 07:57:57 GMT
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
                "address": "222",
                "foodId": "짜장",
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

```
public static void updateStatus(OrderChanged orderChanged){
        repository().findById(orderChanged.getId()).ifPresent(foodCooking->{      
            if(foodCooking.getStatus().equals("주문접수중") || 
               foodCooking.getStatus().equals("주문접수") || 
               foodCooking.getStatus().equals("주문승인")){
                foodCooking.setFoodId(orderChanged.getFoodId());
                foodCooking.setOrderId(orderChanged.getId());
                foodCooking.setId(orderChanged.getId());
                foodCooking.setAddress(orderChanged.getAddress());
                foodCooking.setStatus(orderChanged.getStatus());
                repository().save(foodCooking);
            }    
         });        
    }
```
