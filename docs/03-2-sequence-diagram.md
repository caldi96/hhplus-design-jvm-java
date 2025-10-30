# 시퀀스 다이어그램

## 1. 상품 조회 및 장바구니 추가

```mermaid
sequenceDiagram
    actor User as 사용자
    participant API as API Server
    participant ProductService as Product Service
    participant CartService as Cart Service
    participant ProductRepo as ProductRepository
    participant CartRepo as CartRepository

    %% 상품 목록 조회 (카테고리별)
    User->>API: GET /api/categories/{categoryId}/products?page=1&sort=latest
    API->>ProductService: getProductsByCategory(categoryId, page, sort)

    ProductService->>ProductRepo: findByCategoryIdAndIsActiveTrue(categoryId, pageable)
    ProductRepo-->>ProductService: Page<Product>

    ProductService-->>API: ProductList
    API-->>User: 200 OK - 상품 목록

    %% 상품 상세 조회
    User->>API: GET /api/products/{productId}
    API->>ProductService: getProductDetail(productId)

    ProductService->>ProductService: @Transactional 시작

    ProductService->>ProductRepo: findById(productId)
    ProductRepo-->>ProductService: Optional<Product>

    alt 상품 없음
        ProductService-->>API: 404 Not Found
        API-->>User: 상품을 찾을 수 없음
    end

    %% 조회수 증가 (Dirty Checking)
    ProductService->>ProductService: product.increaseViewCount()

    %% 카테고리 정보는 연관관계로 조회
    ProductService->>ProductService: product.getCategory()

    ProductService->>ProductService: @Transactional 종료 (자동 flush)

    ProductService-->>API: ProductDetail (상품정보, 카테고리)
    API-->>User: 200 OK - 상품 상세

    %% 장바구니 추가
    User->>API: POST /api/cart/items
    Note over User,API: userId, productId, quantity

    API->>CartService: addToCart(userId, productId, quantity)

    CartService->>CartService: @Transactional 시작

    %% 상품 존재 및 재고 확인
    CartService->>ProductRepo: findById(productId)
    ProductRepo-->>CartService: Optional<Product>

    alt 상품 없음 또는 비활성
        CartService-->>API: 404 Not Found
        API-->>User: 상품을 찾을 수 없음
    end

    CartService->>CartService: product.validateStock(quantity)
    alt 재고 부족
        CartService-->>API: 400 Bad Request
        API-->>User: 재고 부족
    end

    CartService->>CartService: product.validateOrderQuantity(quantity)
    alt 최소/최대 구매 수량 위반
        CartService-->>API: 400 Bad Request
        API-->>User: 구매 수량 제한 초과
    end

    %% 장바구니 중복 확인
    CartService->>CartRepo: findByUserIdAndProductId(userId, productId)
    CartRepo-->>CartService: Optional<Cart>

    alt 이미 장바구니에 있음
        CartService->>CartService: cart.addQuantity(quantity)
        CartService->>CartRepo: save(cart)
    else 새로운 상품
        CartService->>CartService: Cart.create(user, product, quantity)
        CartService->>CartRepo: save(cart)
    end

    CartRepo-->>CartService: Cart

    CartService->>CartService: @Transactional 종료

    CartService-->>API: CartItem
    API-->>User: 201 Created - 장바구니 추가 완료
```

---

## 2. 주문 생성 및 결제 프로세스

```mermaid
sequenceDiagram
    actor User as 사용자
    participant API as API Server
    participant OrderService as Order Service
    participant PaymentService as Payment Service
    participant ProductRepo as ProductRepository
    participant UserRepo as UserRepository
    participant UserCouponRepo as UserCouponRepository
    participant OrderRepo as OrderRepository
    participant PaymentRepo as PaymentRepository
    participant PointRepo as PointRepository

    User->>API: POST /api/orders
    Note over User,API: userId, items[{productId, quantity}], couponId, pointAmount

    API->>OrderService: createOrder(orderRequest)

    OrderService->>OrderService: @Transactional 시작

    %% 상품 재고 및 가격 확인 (비관적 락)
    loop 각 주문 상품
        OrderService->>ProductRepo: findByIdWithPessimisticLock(productId)
        Note over OrderService,ProductRepo: @Lock(PESSIMISTIC_WRITE)
        ProductRepo-->>OrderService: Product

        alt 상품 없음 또는 비활성
            OrderService-->>API: 404 Not Found
            API-->>User: 상품을 찾을 수 없음
        end

        OrderService->>OrderService: product.validateStock(quantity)
        alt 재고 부족
            OrderService-->>API: 400 Bad Request
            API-->>User: 재고 부족
        end

        OrderService->>OrderService: product.validateOrderQuantity(quantity)
        alt 수량 제한 위반
            OrderService-->>API: 400 Bad Request
            API-->>User: 구매 수량 제한 초과
        end
    end

    OrderService->>OrderService: 총 금액 계산

    %% 쿠폰 검증 및 할인 계산
    opt 쿠폰 사용
        OrderService->>UserCouponRepo: findByUserIdAndCouponIdAndStatus(userId, couponId, AVAILABLE)
        UserCouponRepo-->>OrderService: Optional<UserCoupon>

        alt 쿠폰 없음 또는 유효하지 않음
            OrderService-->>API: 400 Bad Request
            API-->>User: 사용 불가능한 쿠폰
        end

        OrderService->>OrderService: userCoupon.validate()
        alt 쿠폰 만료
            OrderService-->>API: 400 Bad Request
            API-->>User: 만료된 쿠폰
        end

        OrderService->>OrderService: coupon.validateMinOrderAmount(totalAmount)
        alt 최소 주문 금액 미달
            OrderService-->>API: 400 Bad Request
            API-->>User: 최소 주문 금액 미달
        end

        OrderService->>OrderService: coupon.calculateDiscount(totalAmount)
    end

    %% 포인트 검증
    opt 포인트 사용
        OrderService->>UserRepo: findById(userId)
        UserRepo-->>OrderService: User

        OrderService->>OrderService: user.validatePointBalance(pointAmount)
        alt 포인트 부족
            OrderService-->>API: 400 Bad Request
            API-->>User: 포인트 부족
        end
    end

    %% 최종 금액 계산
    OrderService->>OrderService: 배송비 및 무료배송 확인
    OrderService->>OrderService: 최종 금액 계산 (total - discount + shipping - point)

    %% 주문 생성
    OrderService->>OrderService: Order.create(user, totalAmount, discountAmount, ...)
    OrderService->>OrderRepo: save(order)
    OrderRepo-->>OrderService: Order

    %% 주문 상품 생성 (Cascade)
    loop 각 주문 상품
        OrderService->>OrderService: OrderItem.create(product, quantity, price)
        OrderService->>OrderService: order.addOrderItem(orderItem)
    end

    %% 재고 차감
    loop 각 주문 상품
        OrderService->>OrderService: product.decreaseStock(quantity)
    end

    %% 쿠폰 사용 처리
    opt 쿠폰 사용
        OrderService->>OrderService: userCoupon.use()
        OrderService->>OrderService: coupon.increaseUsageCount()
    end

    %% 포인트 차감
    opt 포인트 사용
        OrderService->>OrderService: user.usePoints(pointAmount)
        OrderService->>OrderService: Point.createUsed(user, pointAmount, order)
        OrderService->>PointRepo: save(point)
    end

    OrderService->>OrderService: @Transactional 종료 (자동 flush & commit)

    OrderService-->>API: Order (PENDING 상태)
    API-->>User: 201 Created - 주문 생성 완료

    %% 결제 처리
    User->>API: POST /api/orders/{orderId}/payment
    Note over User,API: paymentMethod

    API->>PaymentService: processPayment(orderId, paymentMethod)

    PaymentService->>PaymentService: @Transactional 시작

    PaymentService->>OrderRepo: findById(orderId)
    OrderRepo-->>PaymentService: Optional<Order>

    alt 주문 없음
        PaymentService-->>API: 404 Not Found
        API-->>User: 주문을 찾을 수 없음
    end

    %% PG사 결제 요청
    PaymentService->>PaymentService: PG사 결제 요청 (Mock)

    alt 결제 성공
        PaymentService->>PaymentService: Payment.createCompleted(order, amount, method, txId)
        PaymentService->>PaymentRepo: save(payment)

        PaymentService->>PaymentService: order.completePay(paidAt)

        %% 상품 판매량 증가 (Dirty Checking)
        loop 각 주문 상품
            PaymentService->>PaymentService: orderItem.getProduct().increaseSoldCount(quantity)
        end

        PaymentService->>PaymentService: @Transactional 종료

        PaymentService-->>API: Payment 성공
        API-->>User: 200 OK - 결제 완료

    else 결제 실패
        PaymentService->>PaymentService: Payment.createFailed(order, amount, method, reason)
        PaymentService->>PaymentRepo: save(payment)

        %% 롤백 처리
        loop 각 주문 상품
            PaymentService->>PaymentService: orderItem.getProduct().increaseStock(quantity)
        end

        opt 포인트 사용했을 경우
            PaymentService->>PaymentService: order.getUser().refundPoints(pointAmount)
            PaymentService->>PointRepo: deleteByOrderId(orderId)
        end

        opt 쿠폰 사용했을 경우
            PaymentService->>UserCouponRepo: findByUserIdAndCouponId(userId, couponId)
            PaymentService->>PaymentService: userCoupon.restore()
            PaymentService->>PaymentService: coupon.decreaseUsageCount()
        end

        PaymentService->>PaymentService: order.fail()

        PaymentService->>PaymentService: @Transactional 종료

        PaymentService-->>API: Payment 실패
        API-->>User: 400 Bad Request - 결제 실패
    end
```

---

## 3. 선착순 쿠폰 발급 (대기열 처리)

```mermaid
sequenceDiagram
    actor User as 사용자
    participant WS as WebSocket Server
    participant API as API Server
    participant QueueService as Queue Service
    participant CouponService as Coupon Service
    participant Redis as Redis
    participant CouponRepo as CouponRepository
    participant QueueRepo as CouponQueueRepository
    participant UserCouponRepo as UserCouponRepository
    participant EventRepo as QueueEventRepository

    %% 대기열 진입
    User->>WS: WebSocket 연결
    WS-->>User: 연결 완료 (sessionId 생성)

    User->>API: POST /api/coupons/events/{eventId}/queue
    Note over User,API: userId

    API->>QueueService: joinQueue(eventId, userId, sessionId)

    %% 쿠폰 이벤트 확인
    QueueService->>CouponRepo: findByIdAndIsActiveTrue(eventId)
    CouponRepo-->>QueueService: Optional<Coupon>

    alt 쿠폰 이벤트 없음 또는 비활성
        QueueService-->>API: 404 Not Found
        API-->>User: 쿠폰 이벤트를 찾을 수 없음
    end

    QueueService->>QueueService: coupon.validateEventPeriod()
    alt 이벤트 기간 아님
        QueueService-->>API: 400 Bad Request
        API-->>User: 이벤트 기간이 아닙니다
    end

    %% 중복 진입 확인
    QueueService->>QueueRepo: findByCouponIdAndUserIdAndStatusIn(eventId, userId, [WAITING, PROCESSING, ISSUED])
    QueueRepo-->>QueueService: Optional<CouponQueue>

    alt 이미 대기열에 있음
        QueueService-->>API: 400 Bad Request
        API-->>User: 이미 대기 중이거나 발급 완료
    end

    %% 이미 쿠폰 발급받았는지 확인
    QueueService->>UserCouponRepo: countByCouponIdAndUserId(eventId, userId)
    UserCouponRepo-->>QueueService: count

    QueueService->>QueueService: coupon.validatePerUserLimit(count)
    alt 발급 제한 초과
        QueueService-->>API: 400 Bad Request
        API-->>User: 이미 발급받은 쿠폰입니다
    end

    %% Redis에서 대기 순번 생성
    QueueService->>Redis: INCR queue:{eventId}:position_counter
    Redis-->>QueueService: position

    %% Redis 대기열에 추가
    QueueService->>Redis: ZADD queue:{eventId}:waiting {score: position, member: userId}
    QueueService->>Redis: HSET queue:{eventId}:user:{userId} position:{position} sessionId:{sessionId}
    QueueService->>Redis: EXPIRE queue:{eventId}:user:{userId} 3600

    %% DB에 대기열 정보 저장
    QueueService->>QueueService: CouponQueue.create(coupon, user, position, sessionId)
    QueueService->>QueueRepo: save(couponQueue)
    QueueRepo-->>QueueService: CouponQueue

    %% 대기열 이벤트 기록
    QueueService->>QueueService: QueueEvent.createUserJoined(coupon, user)
    QueueService->>EventRepo: save(queueEvent)

    %% 예상 대기 시간 계산
    QueueService->>QueueService: calculateEstimatedWaitTime(position)

    QueueService-->>API: QueuePosition (position, estimatedWaitTime)
    API-->>User: 200 OK - 대기 순번 정보

    %% 실시간 순번 업데이트
    loop Heartbeat & 순번 확인 (주기적)
        User->>API: GET /api/coupons/events/{eventId}/queue/position?userId={userId}
        API->>QueueService: getPosition(eventId, userId)

        QueueService->>Redis: HGET queue:{eventId}:user:{userId} position
        Redis-->>QueueService: currentPosition

        alt 대기열에 없음 (만료됨)
            QueueService-->>API: 404 Not Found
            API-->>User: 대기열에서 제외되었습니다
        end

        %% Heartbeat 업데이트
        QueueService->>QueueRepo: findByCouponIdAndUserId(eventId, userId)
        QueueService->>QueueService: couponQueue.updateHeartbeat()

        QueueService-->>API: QueuePosition (currentPosition, estimatedWaitTime)
        API-->>User: 200 OK - 현재 순번

        WS->>User: 순번 업데이트 푸시
    end

    %% 쿠폰 발급 처리 (백그라운드 워커)
    loop 발급 처리 Worker
        QueueService->>Redis: ZPOPMIN queue:{eventId}:waiting 1
        Redis-->>QueueService: userId, score

        alt 대기자 없음
            QueueService->>QueueService: 대기 (sleep)
        end

        %% 유저 세션 확인
        QueueService->>Redis: HGET queue:{eventId}:user:{userId} sessionId
        Redis-->>QueueService: sessionId

        alt 세션 없음 (타임아웃 또는 연결 끊김)
            QueueService->>QueueRepo: findByCouponIdAndUserId(eventId, userId)
            QueueService->>QueueService: couponQueue.expire()
            QueueService->>QueueService: 다음 대기자 처리
        end

        %% 처리 시작 상태로 변경
        QueueService->>QueueRepo: findByCouponIdAndUserId(eventId, userId)
        QueueService->>QueueService: couponQueue.startProcessing()

        %% 쿠폰 발급
        QueueService->>CouponService: issueCoupon(eventId, userId)
        CouponService->>CouponService: @Transactional 시작

        %% 쿠폰 수량 확인 (비관적 락)
        CouponService->>CouponRepo: findByIdWithPessimisticLock(eventId)
        Note over CouponService,CouponRepo: @Lock(PESSIMISTIC_WRITE)
        CouponRepo-->>CouponService: Coupon

        CouponService->>CouponService: coupon.canIssue()

        alt 쿠폰 소진
            CouponService->>CouponService: @Transactional 롤백
            CouponService-->>QueueService: CouponSoldOutException

            QueueService->>QueueRepo: findByCouponIdAndUserId(eventId, userId)
            QueueService->>QueueService: couponQueue.fail()
            QueueService->>QueueService: QueueEvent.createCouponIssued(coupon, user)
            QueueService->>EventRepo: save(queueEvent)

            QueueService->>Redis: DEL queue:{eventId}:user:{userId}

            QueueService->>WS: 발급 실패 알림
            WS->>User: 쿠폰 소진 알림 푸시

        else 발급 가능
            %% 쿠폰 발급 수량 증가
            CouponService->>CouponService: coupon.issue()

            %% 유저 쿠폰 발급
            CouponService->>CouponService: UserCoupon.create(coupon, user, expiresAt)
            CouponService->>UserCouponRepo: save(userCoupon)
            UserCouponRepo-->>CouponService: UserCoupon

            CouponService->>CouponService: @Transactional 종료
            CouponService-->>QueueService: UserCoupon

            %% 대기열 상태 업데이트
            QueueService->>QueueRepo: findByCouponIdAndUserId(eventId, userId)
            QueueService->>QueueService: couponQueue.complete()

            %% 대기열 이벤트 기록
            QueueService->>QueueService: QueueEvent.createCouponIssued(coupon, user)
            QueueService->>EventRepo: save(queueEvent)

            QueueService->>Redis: DEL queue:{eventId}:user:{userId}

            %% 실시간 알림
            QueueService->>WS: 발급 완료 알림
            WS->>User: 쿠폰 발급 완료 푸시
        end

        %% 다른 대기자들에게 순번 업데이트 알림
        QueueService->>QueueService: QueueEvent.createPositionUpdated(coupon, -1)
        QueueService->>EventRepo: save(queueEvent)
        QueueService->>WS: 순번 업데이트 브로드캐스트
    end

    %% 연결 끊김 처리
    User->>WS: WebSocket 연결 종료
    WS->>QueueService: handleDisconnect(sessionId)

    QueueService->>QueueRepo: findBySessionIdAndStatus(sessionId, WAITING)
    QueueRepo-->>QueueService: Optional<CouponQueue>

    opt 아직 발급 전
        QueueService->>QueueService: couponQueue.expire()
        QueueService->>QueueService: QueueEvent.createUserLeft(coupon, user)
        QueueService->>EventRepo: save(queueEvent)
        QueueService->>Redis: ZREM queue:{eventId}:waiting {userId}
        QueueService->>Redis: DEL queue:{eventId}:user:{userId}
    end
```

---

## 4. 주문 취소 및 환불 프로세스

```mermaid
sequenceDiagram
    actor User as 사용자
    participant API as API Server
    participant OrderService as Order Service
    participant PaymentService as Payment Service
    participant OrderRepo as OrderRepository
    participant PaymentRepo as PaymentRepository
    participant UserCouponRepo as UserCouponRepository
    participant PointRepo as PointRepository
    participant DeliveryRepo as DeliveryRepository

    User->>API: POST /api/orders/{orderId}/cancel
    Note over User,API: userId, reason

    API->>OrderService: cancelOrder(orderId, userId, reason)

    OrderService->>OrderService: @Transactional 시작

    %% 주문 정보 조회 (비관적 락)
    OrderService->>OrderRepo: findByIdWithPessimisticLock(orderId)
    Note over OrderService,OrderRepo: @Lock(PESSIMISTIC_WRITE)
    OrderRepo-->>OrderService: Optional<Order>

    alt 주문 없음
        OrderService-->>API: 404 Not Found
        API-->>User: 주문을 찾을 수 없음
    end

    %% 권한 확인
    OrderService->>OrderService: order.validateOwner(userId)
    alt 본인 주문 아님
        OrderService-->>API: 403 Forbidden
        API-->>User: 권한이 없습니다
    end

    %% 취소 가능 상태 확인
    OrderService->>OrderService: order.canCancel()
    alt 취소 불가 상태
        OrderService-->>API: 400 Bad Request
        API-->>User: 취소 불가능한 상태입니다
    end

    %% 배송 시작 여부 확인
    OrderService->>DeliveryRepo: countByOrderItemsInAndDeliveryStatusNot(orderItems, PREPARING)
    DeliveryRepo-->>OrderService: shippedCount

    alt 배송 시작됨
        OrderService-->>API: 400 Bad Request
        API-->>User: 배송 시작 후 취소 불가
    end

    %% 재고 복구
    loop 각 주문 상품
        OrderService->>OrderService: orderItem.getProduct().increaseStock(quantity)
    end

    %% 쿠폰 복구
    opt 쿠폰 사용했을 경우
        OrderService->>UserCouponRepo: findByUserIdAndCouponId(userId, couponId)
        UserCouponRepo-->>OrderService: Optional<UserCoupon>
        OrderService->>OrderService: userCoupon.restore()
        OrderService->>OrderService: coupon.decreaseUsageCount()
    end

    %% 포인트 환불
    opt 포인트 사용했을 경우
        OrderService->>OrderService: order.getUser().refundPoints(pointAmount)
        OrderService->>OrderService: Point.createRefunded(user, pointAmount, order)
        OrderService->>PointRepo: save(point)
    end

    %% 주문 상태 변경
    OrderService->>OrderService: order.cancel(reason)

    %% 판매량 감소 (Dirty Checking)
    loop 각 주문 상품
        OrderService->>OrderService: orderItem.getProduct().decreaseSoldCount(quantity)
    end

    %% 결제 환불 처리
    alt 결제 완료된 주문
        OrderService->>PaymentService: processRefund(orderId)

        PaymentService->>PaymentRepo: findByOrderIdAndPaymentTypeAndPaymentStatus(orderId, PAYMENT, COMPLETED)
        PaymentRepo-->>PaymentService: Optional<Payment>

        alt 결제 정보 없음
            PaymentService-->>OrderService: PaymentNotFoundException
            OrderService-->>API: 500 Internal Server Error
            API-->>User: 결제 정보를 찾을 수 없음
        end

        %% PG사 환불 요청
        PaymentService->>PaymentService: PG사 환불 요청 (Mock - transactionId 사용)

        alt 환불 성공
            PaymentService->>PaymentService: Payment.createRefundCompleted(order, amount, method, txId)
            PaymentService->>PaymentRepo: save(refundPayment)
            PaymentService-->>OrderService: 환불 완료

        else 환불 실패
            PaymentService->>PaymentService: Payment.createRefundFailed(order, amount, method, reason)
            PaymentService->>PaymentRepo: save(refundPayment)
            PaymentService-->>OrderService: 환불 실패

            OrderService-->>API: 500 Internal Server Error
            API-->>User: 환불 처리 실패
        end
    end

    OrderService->>OrderService: @Transactional 종료

    OrderService-->>API: CancelledOrder (환불 금액, 복구 포인트)
    API-->>User: 200 OK - 주문 취소 완료
    Note over User,API: 환불 금액, 복구된 포인트, 쿠폰 정보
```

---

## 5. 배송 관리 프로세스

```mermaid
sequenceDiagram
    actor Admin as 관리자
    actor User as 사용자
    participant API as API Server
    participant DeliveryService as Delivery Service
    participant OrderService as Order Service
    participant DeliveryRepo as DeliveryRepository
    participant OrderItemRepo as OrderItemRepository
    participant OrderRepo as OrderRepository

    %% 배송 등록 (결제 완료 후)
    Admin->>API: POST /api/deliveries
    Note over Admin,API: orderItemId, 배송지 정보, 택배사

    API->>DeliveryService: createDelivery(deliveryRequest)

    DeliveryService->>DeliveryService: @Transactional 시작

    %% 주문 상품 확인
    DeliveryService->>OrderItemRepo: findById(orderItemId)
    OrderItemRepo-->>DeliveryService: Optional<OrderItem>

    alt 주문 상품 없음
        DeliveryService-->>API: 404 Not Found
        API-->>Admin: 주문 상품을 찾을 수 없음
    end

    DeliveryService->>DeliveryService: orderItem.getOrder().isPaid()
    alt 결제 미완료
        DeliveryService-->>API: 400 Bad Request
        API-->>Admin: 결제 완료 후 배송 등록 가능
    end

    %% 중복 배송 등록 확인
    DeliveryService->>DeliveryRepo: findByOrderItemId(orderItemId)
    DeliveryRepo-->>DeliveryService: Optional<Delivery>

    alt 이미 배송 등록됨
        DeliveryService-->>API: 400 Bad Request
        API-->>Admin: 이미 배송이 등록된 상품입니다
    end

    %% 배송 정보 저장
    DeliveryService->>DeliveryService: Delivery.create(orderItem, address, parcelCorp)
    DeliveryService->>DeliveryRepo: save(delivery)
    DeliveryRepo-->>DeliveryService: Delivery

    DeliveryService->>DeliveryService: @Transactional 종료

    DeliveryService-->>API: Delivery
    API-->>Admin: 201 Created - 배송 등록 완료

    %% 배송 시작 (출고)
    Admin->>API: PATCH /api/deliveries/{deliveryId}/ship
    Note over Admin,API: parcelNumber (송장번호)

    API->>DeliveryService: shipDelivery(deliveryId, parcelNumber)

    DeliveryService->>DeliveryService: @Transactional 시작

    DeliveryService->>DeliveryRepo: findById(deliveryId)
    DeliveryRepo-->>DeliveryService: Optional<Delivery>

    alt 배송 정보 없음
        DeliveryService-->>API: 404 Not Found
        API-->>Admin: 배송 정보를 찾을 수 없음
    end

    %% 배송 상태 업데이트
    DeliveryService->>DeliveryService: delivery.ship(parcelNumber)

    %% 주문 상태 변경
    DeliveryService->>DeliveryService: delivery.getOrderItem().getOrder().ship()

    DeliveryService->>DeliveryService: @Transactional 종료

    DeliveryService-->>API: Delivery
    API-->>Admin: 200 OK - 배송 시작 처리 완료

    %% 배송 완료 처리
    Admin->>API: PATCH /api/deliveries/{deliveryId}/complete

    API->>DeliveryService: completeDelivery(deliveryId)

    DeliveryService->>DeliveryService: @Transactional 시작

    DeliveryService->>DeliveryRepo: findById(deliveryId)
    DeliveryRepo-->>DeliveryService: Optional<Delivery>

    alt 배송 정보 없음
        DeliveryService-->>API: 404 Not Found
        API-->>Admin: 배송 정보를 찾을 수 없음
    end

    %% 배송 완료 처리
    DeliveryService->>DeliveryService: delivery.complete()

    %% 주문의 모든 상품 배송 완료 확인
    DeliveryService->>DeliveryService: order = delivery.getOrderItem().getOrder()
    DeliveryService->>DeliveryService: totalItems = order.getOrderItems().size()

    DeliveryService->>DeliveryRepo: countByOrderItemsInAndDeliveryStatus(order.getOrderItems(), DELIVERED)
    DeliveryRepo-->>DeliveryService: deliveredItems

    alt 모든 상품 배송 완료
        DeliveryService->>DeliveryService: order.completeDelivery()
    end

    DeliveryService->>DeliveryService: @Transactional 종료

    DeliveryService-->>API: Delivery
    API-->>Admin: 200 OK - 배송 완료 처리

    %% 사용자의 배송 조회
    User->>API: GET /api/orders/{orderId}/delivery
    API->>DeliveryService: getDeliveryInfo(orderId)

    DeliveryService->>OrderRepo: findById(orderId)
    OrderRepo-->>DeliveryService: Optional<Order>

    DeliveryService->>DeliveryRepo: findByOrderItemsIn(order.getOrderItems())
    DeliveryRepo-->>DeliveryService: List<Delivery>

    DeliveryService-->>API: DeliveryList
    API-->>User: 200 OK - 배송 정보

    %% 송장번호로 배송 추적
    User->>API: GET /api/deliveries/{parcelNumber}/track
    API->>DeliveryService: trackDelivery(parcelNumber)

    DeliveryService->>DeliveryRepo: findByParcelNumber(parcelNumber)
    DeliveryRepo-->>DeliveryService: Optional<Delivery>

    alt 배송 정보 없음
        DeliveryService-->>API: 404 Not Found
        API-->>User: 배송 정보를 찾을 수 없음
    end

    DeliveryService-->>API: DeliveryTrackingInfo
    API-->>User: 200 OK - 배송 추적 정보
```

---

## 6. 포인트 충전 프로세스

```mermaid
sequenceDiagram
    actor User as 사용자
    participant API as API Server
    participant PointService as Point Service
    participant PaymentService as Payment Service
    participant UserRepo as UserRepository
    participant PointRepo as PointRepository

    User->>API: POST /api/points/charge
    Note over User,API: userId, amount, paymentMethod

    API->>PointService: chargePoints(userId, amount, paymentMethod)

    PointService->>UserRepo: findById(userId)
    UserRepo-->>PointService: Optional<User>

    alt 사용자 없음
        PointService-->>API: 404 Not Found
        API-->>User: 사용자를 찾을 수 없음
    end

    PointService->>PointService: validateChargeAmount(amount)
    alt 충전 금액 오류
        PointService-->>API: 400 Bad Request
        API-->>User: 충전 금액이 유효하지 않습니다
    end

    %% 결제 처리
    PointService->>PaymentService: processPointChargePayment(userId, amount, paymentMethod)

    PaymentService->>PaymentService: PG사 결제 요청 (Mock)

    alt 결제 실패
        PaymentService-->>PointService: PaymentFailedException
        PointService-->>API: 400 Bad Request
        API-->>User: 결제 실패
    end

    PaymentService-->>PointService: transactionId, pgProvider

    %% 포인트 충전
    PointService->>PointService: @Transactional 시작

    %% 포인트 잔액 업데이트 (Dirty Checking)
    PointService->>PointService: user.chargePoints(amount)

    %% 포인트 내역 저장 (1년 후 만료)
    PointService->>PointService: Point.createEarned(user, amount, "포인트 충전", expiresAt)
    PointService->>PointRepo: save(point)
    PointRepo-->>PointService: Point

    PointService->>PointService: @Transactional 종료

    %% 최종 포인트 잔액 조회
    PointService->>PointService: totalBalance = user.getPointBalance()

    PointService-->>API: ChargeResult (chargedAmount, totalBalance)
    API-->>User: 200 OK - 포인트 충전 완료
    Note over User,API: 충전 금액, 총 포인트 잔액

    %% 포인트 내역 조회
    User->>API: GET /api/points/history?userId={userId}&page=1&size=20
    API->>PointService: getPointHistory(userId, page, size)

    PointService->>PointRepo: findByUserIdOrderByCreatedAtDesc(userId, pageable)
    PointRepo-->>PointService: Page<Point>

    PointService-->>API: PointHistoryList
    API-->>User: 200 OK - 포인트 내역

    %% 포인트 만료 처리 (배치 작업)
    Note over PointService: 스케줄러에 의한 자동 실행 (매일 자정)

    PointService->>PointService: @Transactional 시작

    %% 만료된 포인트 조회
    PointService->>PointRepo: findExpiredPoints(LocalDateTime.now())
    Note over PointService,PointRepo: @Query로 custom query
    PointRepo-->>PointService: Map<User, BigDecimal>

    loop 각 사용자
        %% 포인트 차감 (Dirty Checking)
        PointService->>PointService: user.expirePoints(expiredAmount)

        %% 만료 내역 저장
        PointService->>PointService: Point.createExpired(user, expiredAmount)
        PointService->>PointRepo: save(point)
    end

    PointService->>PointService: @Transactional 종료
```

---

## 7. 장바구니에서 주문 생성

```mermaid
sequenceDiagram
    actor User as 사용자
    participant API as API Server
    participant CartService as Cart Service
    participant OrderService as Order Service
    participant CartRepo as CartRepository
    participant OrderRepo as OrderRepository

    %% 장바구니 조회
    User->>API: GET /api/users/{userId}/cart
    API->>CartService: getCart(userId)

    CartService->>CartRepo: findByUserIdWithProduct(userId)
    Note over CartService,CartRepo: @EntityGraph로 Product fetch
    CartRepo-->>CartService: List<Cart>

    loop 각 장바구니 상품
        CartService->>CartService: cart.calculateSubtotal()
        CartService->>CartService: cart.checkAvailability()
    end

    CartService->>CartService: totalPrice 계산
    CartService->>CartService: unavailableItems 필터링

    CartService-->>API: Cart (cartItems, totalPrice, unavailableItems)
    API-->>User: 200 OK - 장바구니 조회

    %% 장바구니에서 주문 생성
    User->>API: POST /api/orders/from-cart
    Note over User,API: userId, cartItemIds[], couponId, pointAmount

    API->>OrderService: createOrderFromCart(userId, cartItemIds, couponId, pointAmount)

    OrderService->>OrderService: @Transactional 시작

    %% 장바구니 상품 조회
    OrderService->>CartRepo: findByIdInAndUserIdWithProduct(cartItemIds, userId)
    Note over OrderService,CartRepo: @EntityGraph로 Product fetch
    CartRepo-->>OrderService: List<Cart>

    alt 장바구니 아이템 없음
        OrderService-->>API: 400 Bad Request
        API-->>User: 장바구니가 비어있습니다
    end

    %% 각 상품 검증
    loop 각 장바구니 상품
        OrderService->>OrderService: product = cart.getProduct()
        OrderService->>OrderService: product.validateActive()
        alt 판매 중지된 상품
            OrderService-->>API: 400 Bad Request
            API-->>User: 판매 중지된 상품이 포함되어 있습니다
        end

        OrderService->>OrderService: product.validateStock(cart.getQuantity())
        alt 재고 부족
            OrderService-->>API: 400 Bad Request
            API-->>User: 재고가 부족한 상품이 있습니다
        end

        OrderService->>OrderService: product.validateOrderQuantity(cart.getQuantity())
        alt 수량 제한 위반
            OrderService-->>API: 400 Bad Request
            API-->>User: 구매 수량 제한을 초과했습니다
        end
    end

    %% 주문 생성 (기존 주문 생성 로직 재사용)
    OrderService->>OrderService: totalAmount 계산

    opt 쿠폰 사용
        OrderService->>OrderService: 쿠폰 검증 및 할인 계산
    end

    opt 포인트 사용
        OrderService->>OrderService: 포인트 검증
    end

    OrderService->>OrderService: 최종 금액 계산

    %% 주문 생성
    OrderService->>OrderService: Order.create(user, totalAmount, discountAmount, ...)
    OrderService->>OrderRepo: save(order)
    OrderRepo-->>OrderService: Order

    %% 주문 상품 생성 (Cascade)
    loop 각 장바구니 상품
        OrderService->>OrderService: OrderItem.create(cart.getProduct(), cart.getQuantity(), price)
        OrderService->>OrderService: order.addOrderItem(orderItem)
    end

    %% 재고 차감 (Dirty Checking)
    loop 각 장바구니 상품
        OrderService->>OrderService: cart.getProduct().decreaseStock(cart.getQuantity())
    end

    %% 쿠폰/포인트 처리
    opt 쿠폰 사용
        OrderService->>OrderService: userCoupon.use()
        OrderService->>OrderService: coupon.increaseUsageCount()
    end

    opt 포인트 사용
        OrderService->>OrderService: user.usePoints(pointAmount)
        OrderService->>OrderService: Point.createUsed(user, pointAmount, order)
    end

    %% 장바구니에서 주문한 상품 삭제
    OrderService->>CartRepo: deleteAllById(cartItemIds)

    OrderService->>OrderService: @Transactional 종료

    OrderService-->>API: Order (PENDING 상태)
    API-->>User: 201 Created - 주문 생성 완료
    Note over User,API: 주문 ID, 최종 금액
```