---
config:
  look: neo
---

---------- 상품 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant P1 as ProductController
participant P2 as ProductService
participant P3 as ProductRepository

    A1->>P1: 전체상품조회 : GET /products/${kind}
    P1->>P2: getProducts()
    P2->>P3: findAll()
    P3-->>P2: 상품 데이터
    P2-->>P1: 상품 목록
    P1-->>A1: 200 OK (전체 상품 목록 json)

    A1->>P1: GET /products/${kind}?page=1&size=20&sort=latest|orderCount|rating
    Note right of P1: sort: latest(최신순)<br/>orderCount(주문많은순)<br/>rating(평점높은순)
    P1->>P2: getProducts(kind, PageRequest)
    P2->>P3: findAll(kind, Pageable)
    P3-->>P2: Page<Product> (20개)
    P2-->>P1: 페이징된 상품 목록
    P1-->>A1: 200 OK<br/>{products[], page, totalPages}

    A1->>P1: 상품상세조회 : GET /products/{id}
    P1->>P2: getProductById(id)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 상품 데이터
        P2-->>P1: 상품 정보
        P1-->>A1: 200 OK (상품 상세)
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end

    A1->>P1: 상품 등록 : POST /products
    P1->>P2: createProduct(productDto)
    P2->>P3: save(product)
    P3-->>P2: 저장된 상품
    P2-->>P1: 생성된 상품 정보
    P1-->>A1: 201 Created

    A1->>P1: 상품 수정 : PUT /products/{id}
    P1->>P2: updateProduct(id, productDto)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 기존 상품 데이터
        P2->>P3: save(updatedProduct)
        P3-->>P2: 수정된 상품
        P2-->>P1: 수정된 상품 정보
        P1-->>A1: 200 OK
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end

    A1->>P1: 상품 삭제 : DELETE /products/{id}
    P1->>P2: deleteProduct(id)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 상품 데이터
        P2->>P3: delete(id)
        P3-->>P2: 삭제 완료
        P2-->>P1: 삭제 성공
        P1-->>A1: 204 No Content
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end

    Note over A1,P3: 재고 수정
    A1->>P1: PATCH /products/{id}/stock
    P1->>P2: updateStock(id, quantity)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 기존 상품 데이터
        P2->>P3: save(updatedProduct)
        P3-->>P2: 재고 수정된 상품
        P2-->>P1: 수정된 상품 정보
        P1-->>A1: 200 OK
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end
    
    Note over A1,P3: 가격 수정
    A1->>P1: PATCH /products/{id}/price
    P1->>P2: updatePrice(id, price)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 기존 상품 데이터
        P2->>P3: save(updatedProduct)
        P3-->>P2: 가격 수정된 상품
        P2-->>P1: 수정된 상품 정보
        P1-->>A1: 200 OK
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end

    Note over A1,P3: 품절 상태 변경
    A1->>P1: PATCH /products/{id}/stock-status
    P1->>P2: updateStockStatus(id, status)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 기존 상품 데이터
        P2->>P3: save(updatedProduct)
        P3-->>P2: 품절 상태 변경된 상품
        P2-->>P1: 수정된 상품 정보
        P1-->>A1: 200 OK
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end
    
    Note over A1,P3: 판매 중지/재개
    A1->>P1: PATCH /products/{id}/sale-status
    P1->>P2: updateSaleStatus(id, status)
    P2->>P3: findById(id)
    alt 상품이 존재하는 경우
        P3-->>P2: 기존 상품 데이터
        P2->>P3: save(updatedProduct)
        P3-->>P2: 판매 상태 변경된 상품
        P2-->>P1: 수정된 상품 정보
        P1-->>A1: 200 OK
    else 상품이 없는 경우
        P3-->>P2: Empty
        P2-->>P1: ProductNotFoundException
        P1-->>A1: 404 Not Found
    end

---------- 카테고리 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant C1 as CategoryController
participant C2 as CategoryService
participant C3 as CategoryRepository

    A1->>C1: 카테고리 전체 조회 : GET /categories
    C1->>C2: getAllCategories()
    C2->>C3: findAll()
    C3-->>C2: 카테고리 목록
    C2-->>C1: 카테고리 리스트
    C1-->>A1: 200 OK (전체 카테고리)

    A1->>C1: 카테고리 단건 조회 : GET /categories/{id}
    C1->>C2: getCategoryById(id)
    C2->>C3: findById(id)
    C3-->>C2: 카테고리 데이터
    C2-->>C1: 카테고리 정보
    C1-->>A1: 200 OK (카테고리 상세)

    A1->>C1: 카테고리 생성 : POST /categories
    C1->>C2: createCategory(categoryDto)
    C2->>C3: save(category)
    C3-->>C2: 저장된 카테고리
    C2-->>C1: 생성된 카테고리 정보
    C1-->>A1: 201 Created

    Note over A1,C3: 카테고리 수정
    A1->>C1: PUT /categories/{id}
    C1->>C2: updateCategory(id, categoryDto)
    C2->>C3: findById(id)
    alt 카테고리가 존재하는 경우
        C3-->>C2: 기존 카테고리
        C2->>C3: save(updatedCategory)
        C3-->>C2: 수정된 카테고리
        C2-->>C1: 수정된 카테고리 정보
        C1-->>A1: 200 OK
    else 카테고리가 없는 경우
        C3-->>C2: Empty
        C2-->>C1: CategoryNotFoundException
        C1-->>A1: 404 Not Found
    end
    
    Note over A1,C3: 카테고리 삭제
    A1->>C1: DELETE /categories/{id}
    C1->>C2: deleteCategory(id)
    C2->>C3: findById(id)
    alt 카테고리가 존재하는 경우
        C3-->>C2: 카테고리 데이터
        C2->>C3: delete(id)
        C3-->>C2: 삭제 완료
        C2-->>C1: 삭제 성공
        C1-->>A1: 204 No Content
    else 카테고리가 없는 경우
        C3-->>C2: Empty
        C2-->>C1: CategoryNotFoundException
        C1-->>A1: 404 Not Found
    end

---------- 카테고리별 상품 조회 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant P1 as ProductController
participant P2 as ProductService
participant P3 as ProductRepository
participant C3 as CategoryRepository

    A1->>P1: 카테고리별 상품 목록 조회 : GET /categories/{categoryId}/products?page=1&size=20&sort=latest
    Note right of P1: sort: latest(최신순)<br/>orderCount(주문많은순)<br/>rating(평점높은순)
    P1->>P2: getProductsByCategory(categoryId, PageRequest)
    P2->>C3: findById(categoryId)
    alt 카테고리가 존재하는 경우
        C3-->>P2: 카테고리 데이터
        P2->>P3: findByCategoryId(categoryId, Pageable)
        P3-->>P2: Page<Product> (20개)
        P2-->>P1: 카테고리별 상품 목록 + 페이징 정보
        P1-->>A1: 200 OK<br/>{category, products[], page, totalPages, totalElements}
    else 카테고리가 없는 경우
        C3-->>P2: Empty
        P2-->>P1: CategoryNotFoundException
        P1-->>A1: 404 Not Found
    end

---------- 주문 / 결제 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant O1 as OrderController
participant O2 as OrderService
participant O3 as OrderRepository
participant OI3 as OrderItemRepository
participant U3 as UserRepository
participant P3 as ProductRepository
participant Po3 as PointRepository
participant C3 as CouponRepository
participant Pay as PaymentService

    A1->>O1: POST /orders
    O1->>O2: createOrder(orderDto)
    
    Note over O2: 1. 유저 확인
    O2->>U3: findById(userId)
    alt 유저가 존재하는 경우
        U3-->>O2: 유저 정보
    else 유저가 없는 경우
        U3-->>O2: Empty
        O2-->>O1: UserNotFoundException
        O1-->>A1: 404 Not Found
    end
    
    Note over O2: 2. 상품 및 재고 확인
    loop 각 주문 상품마다
        O2->>P3: findById(productId)
        alt 상품이 존재하는 경우
            P3-->>O2: 상품 정보
            alt 재고 >= 요청수량
                Note right of O2: 재고 확인 통과
            else 재고 < 요청수량
                O2-->>O1: InsufficientStockException
                O1-->>A1: 400 Bad Request (재고 부족)
            end
        else 상품이 없는 경우
            P3-->>O2: Empty
            O2-->>O1: ProductNotFoundException
            O1-->>A1: 404 Not Found
        end
    end
    
    Note over O2: 3. 총 상품 가격 계산
    Note right of O2: totalPrice = Σ(상품가격 × 수량)
    
    Note over O2: 4. 포인트 차감 (1순위)
    alt 포인트 사용 시
        O2->>Po3: findByUserId(userId)
        Po3-->>O2: 보유 포인트
        alt 보유 포인트 충분
            Note right of O2: price = totalPrice - point
            O2->>Po3: save(차감된 포인트)
            Po3-->>O2: 포인트 차감 완료
        else 포인트 부족
            O2-->>O1: InsufficientPointException
            O1-->>A1: 400 Bad Request
        end
    else 포인트 미사용
        Note right of O2: price = totalPrice
    end
    
    Note over O2: 5. 쿠폰 적용 (2순위)
    alt 쿠폰 사용 시
        O2->>C3: findById(couponId)
        C3-->>O2: 쿠폰 정보
        alt 정률 쿠폰
            Note right of O2: discount = price × (할인율/100)<br/>finalPrice = price - discount
        else 정량 쿠폰
            alt 할인가격 >= price
                Note right of O2: finalPrice = 0원
            else 할인가격 < price
                Note right of O2: finalPrice = price - 할인가격
            end
        end
    else 쿠폰 미사용
        Note right of O2: finalPrice = price
    end
    
    Note over O2: 6. 재고 차감
    loop 각 주문 상품마다
        O2->>P3: findById(productId)
        P3-->>O2: 상품 정보
        Note right of O2: stock -= 주문수량
        O2->>P3: save(updatedProduct)
        P3-->>O2: 재고 차감 완료
    end
    
    Note over O2: 7. 주문 생성 (상태: PENDING)
    O2->>O3: save(order)
    O3-->>O2: 생성된 주문 (orderId)
    
    Note over O2: 8. 주문 아이템 생성
    loop 각 주문 상품마다
        Note right of O2: OrderItem 생성<br/>(orderId, productId, quantity, price)
        O2->>OI3: save(orderItem)
        OI3-->>O2: 저장된 주문 아이템
    end
    
    O2-->>O1: 주문 정보 (PENDING 상태)
    O1-->>A1: 201 Created<br/>{orderId, finalPrice, status: PENDING}
    
    Note over A1,Pay: 결제 처리
    A1->>O1: POST /orders/{orderId}/payment
    O1->>O2: processPayment(orderId, paymentDto)
    
    O2->>O3: findById(orderId)
    alt 주문이 존재하는 경우
        O3-->>O2: 주문 정보
        O2->>Pay: requestPayment(finalPrice, paymentMethod)
        alt 결제 성공
            Pay-->>O2: 결제 성공 (paymentId)
            Note right of O2: 주문 상태: PENDING → PAID
            O2->>O3: save(paidOrder)
            O3-->>O2: 결제 완료된 주문
            O2-->>O1: 결제 성공 정보
            O1-->>A1: 200 OK<br/>{paymentId, status: PAID}
        else 결제 실패
            Pay-->>O2: 결제 실패
            Note right of O2: 재고 복구 필요
            loop 각 주문 아이템마다
                O2->>OI3: findByOrderId(orderId)
                OI3-->>O2: 주문 아이템 목록
                O2->>P3: findById(productId)
                P3-->>O2: 상품 정보
                Note right of O2: stock += 주문수량
                O2->>P3: save(restoredProduct)
                P3-->>O2: 재고 복구 완료
            end
            Note right of O2: 포인트 복구
            alt 포인트 사용했었다면
                O2->>Po3: save(restoredPoint)
                Po3-->>O2: 포인트 복구 완료
            end
            Note right of O2: 주문 상태: FAILED
            O2->>O3: save(failedOrder)
            O3-->>O2: 실패 처리된 주문
            O2-->>O1: 결제 실패 정보
            O1-->>A1: 400 Bad Request<br/>{status: FAILED}
        end
    else 주문이 없는 경우
        O3-->>O2: Empty
        O2-->>O1: OrderNotFoundException
        O1-->>A1: 404 Not Found
    end

---------- 장바구니 아이템 추가 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant C1 as CartController
participant C2 as CartService
participant C3 as CartRepository
participant CI3 as CartItemRepository
participant U3 as UserRepository
participant P3 as ProductRepository

    A1->>C1: POST /cart/items
    C1->>C2: addCartItem(userId, productId, quantity)
    
    Note over C2: 1. 유저 확인
    C2->>U3: findById(userId)
    alt 유저가 존재하는 경우
        U3-->>C2: 유저 정보
    else 유저가 없는 경우
        U3-->>C2: Empty
        C2-->>C1: UserNotFoundException
        C1-->>A1: 404 Not Found
    end
    
    Note over C2: 2. 상품 확인
    C2->>P3: findById(productId)
    alt 상품이 존재하는 경우
        P3-->>C2: 상품 정보
    else 상품이 없는 경우
        P3-->>C2: Empty
        C2-->>C1: ProductNotFoundException
        C1-->>A1: 404 Not Found
    end
    
    Note over C2: 3. 장바구니 조회 또는 생성
    C2->>C3: findByUserId(userId)
    alt 장바구니가 존재하는 경우
        C3-->>C2: 기존 장바구니
    else 장바구니가 없는 경우
        C2->>C3: save(newCart)
        C3-->>C2: 새 장바구니 생성
    end
    
    Note over C2: 4. 장바구니 아이템 추가/수정
    C2->>CI3: findByCartIdAndProductId(cartId, productId)
    alt 이미 담긴 상품인 경우
        CI3-->>C2: 기존 장바구니 아이템
        Note right of C2: 수량 += quantity
        C2->>CI3: save(updatedCartItem)
        CI3-->>C2: 수량 업데이트 완료
    else 새로운 상품인 경우
        Note right of C2: CartItem 생성
        C2->>CI3: save(newCartItem)
        CI3-->>C2: 장바구니 아이템 추가 완료
    end
    
    C2-->>C1: 장바구니 아이템 정보
    C1-->>A1: 201 Created

---------- 장바구니 조회 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant C1 as CartController
participant C2 as CartService
participant C3 as CartRepository
participant CI3 as CartItemRepository
participant P3 as ProductRepository

    A1->>C1: GET /cart
    C1->>C2: getCart(userId)
    C2->>C3: findByUserId(userId)
    alt 장바구니가 존재하는 경우
        C3-->>C2: 장바구니 정보
        C2->>CI3: findByCartId(cartId)
        CI3-->>C2: 장바구니 아이템 목록
        loop 각 장바구니 아이템마다
            C2->>P3: findById(productId)
            P3-->>C2: 상품 정보
        end
        Note right of C2: 총 금액 계산
        C2-->>C1: 장바구니 전체 정보
        C1-->>A1: 200 OK<br/>{cartItems[], totalPrice}
    else 장바구니가 없는 경우
        C3-->>C2: Empty
        C2-->>C1: 빈 장바구니
        C1-->>A1: 200 OK<br/>{cartItems: []}
    end

---------- 장바구니 아이템 수량 변경 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant C1 as CartController
participant C2 as CartService
participant CI3 as CartItemRepository

    A1->>C1: PATCH /cart/items/{itemId}
    C1->>C2: updateCartItemQuantity(itemId, quantity)
    C2->>CI3: findById(itemId)
    alt 장바구니 아이템이 존재하는 경우
        CI3-->>C2: 장바구니 아이템
        Note right of C2: quantity 업데이트
        C2->>CI3: save(updatedCartItem)
        CI3-->>C2: 수량 변경 완료
        C2-->>C1: 변경된 아이템 정보
        C1-->>A1: 200 OK
    else 아이템이 없는 경우
        CI3-->>C2: Empty
        C2-->>C1: CartItemNotFoundException
        C1-->>A1: 404 Not Found
    end

---------- 장바구니 아이템 삭제 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant C1 as CartController
participant C2 as CartService
participant CI3 as CartItemRepository

    A1->>C1: DELETE /cart/items/{itemId}
    C1->>C2: deleteCartItem(itemId)
    C2->>CI3: findById(itemId)
    alt 장바구니 아이템이 존재하는 경우
        CI3-->>C2: 장바구니 아이템
        C2->>CI3: delete(itemId)
        CI3-->>C2: 삭제 완료
        C2-->>C1: 삭제 성공
        C1-->>A1: 204 No Content
    else 아이템이 없는 경우
        CI3-->>C2: Empty
        C2-->>C1: CartItemNotFoundException
        C1-->>A1: 404 Not Found
    end

---------- 장바구니에서 주문 생성 (전체 프로세스) ----------\

sequenceDiagram
actor A1 as 클라이언트
participant C1 as CartController
participant C2 as CartService
participant C3 as CartRepository
participant CI3 as CartItemRepository
participant O1 as OrderController
participant O2 as OrderService
participant O3 as OrderRepository
participant OI3 as OrderItemRepository
participant P3 as ProductRepository
participant Po3 as PointRepository
participant Cp3 as CouponRepository
participant Pay as PaymentService

    Note over A1,Pay: 1. 장바구니 조회
    A1->>C1: GET /cart
    C1->>C2: getCart(userId)
    C2->>C3: findByUserId(userId)
    C3-->>C2: 장바구니 정보
    C2->>CI3: findByCartId(cartId)
    CI3-->>C2: 장바구니 아이템 목록
    C2-->>C1: 장바구니 전체 정보
    C1-->>A1: 200 OK (장바구니 아이템들)
    
    Note over A1,Pay: 2. 주문 생성 요청
    A1->>O1: POST /orders/from-cart
    O1->>O2: createOrderFromCart(userId, cartId, orderDto)
    
    Note over O2: 장바구니 아이템 조회
    O2->>CI3: findByCartId(cartId)
    CI3-->>O2: 장바구니 아이템 목록
    
    Note over O2: 상품 및 재고 확인
    loop 각 장바구니 아이템마다
        O2->>P3: findById(productId)
        alt 상품 존재 및 재고 충분
            P3-->>O2: 상품 정보
        else 상품 없음 또는 재고 부족
            P3-->>O2: Empty / InsufficientStock
            O2-->>O1: Exception
            O1-->>A1: 400/404 Error
        end
    end
    
    Note over O2: 가격 계산
    Note right of O2: totalPrice = Σ(상품가격 × 수량)
    
    Note over O2: 포인트 차감 (1순위)
    alt 포인트 사용 시
        O2->>Po3: findByUserId(userId)
        Po3-->>O2: 포인트 정보
        O2->>Po3: save(차감된 포인트)
        Po3-->>O2: 포인트 차감 완료
    end
    
    Note over O2: 쿠폰 적용 (2순위)
    alt 쿠폰 사용 시
        O2->>Cp3: findById(couponId)
        Cp3-->>O2: 쿠폰 정보
        Note right of O2: 정률/정량 할인 적용<br/>finalPrice 계산
    end
    
    Note over O2: 재고 차감
    loop 각 장바구니 아이템마다
        O2->>P3: findById(productId)
        P3-->>O2: 상품 정보
        Note right of O2: stock -= quantity
        O2->>P3: save(updatedProduct)
        P3-->>O2: 재고 차감 완료
    end
    
    Note over O2: 주문 생성
    O2->>O3: save(order - PENDING)
    O3-->>O2: 생성된 주문
    
    Note over O2: 주문 아이템 생성
    loop 각 장바구니 아이템마다
        Note right of O2: CartItem → OrderItem 변환
        O2->>OI3: save(orderItem)
        OI3-->>O2: 주문 아이템 생성 완료
    end
    
    Note over O2: 장바구니 비우기
    O2->>CI3: deleteByCartId(cartId)
    CI3-->>O2: 장바구니 아이템 삭제 완료
    
    O2-->>O1: 주문 정보
    O1-->>A1: 201 Created<br/>{orderId, finalPrice, status: PENDING}
    
    Note over A1,Pay: 3. 결제 처리
    A1->>O1: POST /orders/{orderId}/payment
    O1->>O2: processPayment(orderId, paymentDto)
    O2->>O3: findById(orderId)
    O3-->>O2: 주문 정보
    O2->>Pay: requestPayment(finalPrice)
    alt 결제 성공
        Pay-->>O2: 결제 성공
        Note right of O2: 상태: PENDING → PAID
        O2->>O3: save(paidOrder)
        O3-->>O2: 주문 확정
        O2-->>O1: 결제 성공
        O1-->>A1: 200 OK<br/>{paymentId, status: PAID}
    else 결제 실패
        Pay-->>O2: 결제 실패
        Note over O2: 재고/포인트 복구
        loop 각 주문 아이템마다
            O2->>OI3: findByOrderId(orderId)
            OI3-->>O2: 주문 아이템 목록
            O2->>P3: 재고 복구
            P3-->>O2: 재고 복구 완료
        end
        O2->>Po3: 포인트 복구
        Po3-->>O2: 포인트 복구 완료
        Note right of O2: 상태: FAILED
        O2->>O3: save(failedOrder)
        O3-->>O2: 실패 처리
        O2-->>O1: 결제 실패
        O1-->>A1: 400 Bad Request<br/>{status: FAILED}
    end

---------- 기본 포인트 충전 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant Po1 as PointController
participant Po2 as PointService
participant Po3 as PointRepository
participant U3 as UserRepository
participant Pay as PaymentService

    A1->>Po1: POST /points/charge
    Po1->>Po2: chargePoint(userId, amount, paymentDto)
    
    Note over Po2: 1. 유저 확인
    Po2->>U3: findById(userId)
    alt 유저가 존재하는 경우
        U3-->>Po2: 유저 정보
    else 유저가 없는 경우
        U3-->>Po2: Empty
        Po2-->>Po1: UserNotFoundException
        Po1-->>A1: 404 Not Found
    end
    
    Note over Po2: 2. 결제 처리
    Po2->>Pay: requestPayment(amount, paymentMethod)
    alt 결제 성공
        Pay-->>Po2: 결제 성공 (paymentId)
        
        Note over Po2: 3. 포인트 충전
        Po2->>Po3: findByUserId(userId)
        alt 포인트 계정이 존재하는 경우
            Po3-->>Po2: 기존 포인트 정보
            Note right of Po2: currentPoint += amount
            Po2->>Po3: save(updatedPoint)
            Po3-->>Po2: 포인트 충전 완료
        else 포인트 계정이 없는 경우
            Note right of Po2: 새 포인트 계정 생성<br/>point = amount
            Po2->>Po3: save(newPoint)
            Po3-->>Po2: 포인트 계정 생성 완료
        end
        
        Po2-->>Po1: 충전 성공 정보
        Po1-->>A1: 200 OK<br/>{paymentId, chargedAmount, totalPoint}
    else 결제 실패
        Pay-->>Po2: 결제 실패
        Po2-->>Po1: PaymentFailedException
        Po1-->>A1: 400 Bad Request<br/>{message: 결제 실패}
    end

---------- 주문 완료 시 적립 포인트 지급 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant O1 as OrderController
participant O2 as OrderService
participant O3 as OrderRepository
participant Po2 as PointService
participant Po3 as PointRepository
participant Pay as PaymentService

    Note over A1,Pay: 주문 결제 완료
    A1->>O1: POST /orders/{orderId}/payment
    O1->>O2: processPayment(orderId, paymentDto)
    O2->>O3: findById(orderId)
    O3-->>O2: 주문 정보
    O2->>Pay: requestPayment(finalPrice)
    Pay-->>O2: 결제 성공
    
    Note over O2: 주문 상태: PAID
    O2->>O3: save(paidOrder)
    O3-->>O2: 결제 완료된 주문
    
    Note over O2: 적립 포인트 계산
    Note right of O2: earnedPoint = finalPrice × 적립률<br/>(예: 1% 적립)
    
    O2->>Po2: addRewardPoint(userId, earnedPoint, orderId)
    Po2->>Po3: findByUserId(userId)
    Po3-->>Po2: 포인트 정보
    Note right of Po2: point += earnedPoint
    Po2->>Po3: save(updatedPoint)
    Po3-->>Po2: 포인트 적립 완료
    Po2-->>O2: 적립 성공
    
    O2-->>O1: 결제 및 적립 완료
    O1-->>A1: 200 OK<br/>{paymentId, earnedPoint, totalPoint}

---------- 포인트 잔액 조회 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant Po1 as PointController
participant Po2 as PointService
participant Po3 as PointRepository

    A1->>Po1: GET /points
    Po1->>Po2: getPoint(userId)
    Po2->>Po3: findByUserId(userId)
    alt 포인트 계정이 존재하는 경우
        Po3-->>Po2: 포인트 정보
        Po2-->>Po1: 포인트 잔액
        Po1-->>A1: 200 OK<br/>{point, userId}
    else 포인트 계정이 없는 경우
        Po3-->>Po2: Empty
        Po2-->>Po1: 포인트 0
        Po1-->>A1: 200 OK<br/>{point: 0}
    end

---------- 포인트 충전/사용 내역 조회 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant Po1 as PointController
participant Po2 as PointService
participant PH3 as PointHistoryRepository
participant Po3 as PointRepository
participant U3 as UserRepository

    Note over A1,U3: 1. 전체 내역 조회
    A1->>Po1: GET /points/history?page=1&size=20
    Po1->>Po2: getPointHistory(userId, PageRequest)
    Po2->>U3: findById(userId)
    U3-->>Po2: 유저 정보
    Po2->>PH3: findByUserId(userId, Pageable)
    PH3-->>Po2: 전체 내역 (20개)
    Po2-->>Po1: 포인트 내역
    Po1-->>A1: 200 OK<br/>{history[], page, totalPages}
    
    Note over A1,U3: 2. 타입별 내역 조회
    A1->>Po1: GET /points/history?type=CHARGE&page=1&size=20
    Po1->>Po2: getPointHistoryByType(userId, CHARGE)
    Po2->>PH3: findByUserIdAndType(userId, CHARGE, Pageable)
    PH3-->>Po2: 충전 내역 (20개)
    Po2-->>Po1: 충전 내역만
    Po1-->>A1: 200 OK<br/>{history[], page}
    
    Note over A1,U3: 3. 기간별 내역 조회
    A1->>Po1: GET /points/history?startDate=2025-01-01&endDate=2025-01-31
    Po1->>Po2: getPointHistoryByDateRange(userId, dates)
    Po2->>PH3: findByUserIdAndCreatedAtBetween(userId, dates, Pageable)
    PH3-->>Po2: 기간별 내역
    Po2-->>Po1: 기간별 내역
    Po1-->>A1: 200 OK<br/>{history[], page}
    
    Note over A1,U3: 4. 포인트 통계 조회
    A1->>Po1: GET /points/summary
    Po1->>Po2: getPointSummary(userId)
    Po2->>Po3: findByUserId(userId)
    Po3-->>Po2: 현재 포인트
    Po2->>PH3: sumByUserIdAndType(userId, CHARGE)
    PH3-->>Po2: 총 충전액
    Po2->>PH3: sumByUserIdAndType(userId, USE)
    PH3-->>Po2: 총 사용액
    Po2->>PH3: sumByUserIdAndType(userId, EARN)
    PH3-->>Po2: 총 적립액
    Po2->>PH3: sumByUserIdAndType(userId, REFUND)
    PH3-->>Po2: 총 환불액
    Po2-->>Po1: 통계 정보
    Po1-->>A1: 200 OK<br/>{currentPoint, totalCharged, totalUsed, totalEarned, totalRefunded}

---------- 쿠폰 선착순 대기열 ----------\

sequenceDiagram
actor Admin as 관리자
actor A1 as 클라이언트
participant C1 as CouponController
participant C2 as CouponService
participant CE3 as CouponEventRepository
participant Q as QueueService(Redis)
participant Worker as CouponWorker
participant C3 as CouponRepository
participant UC3 as UserCouponRepository

    Note over Admin,UC3: 1. 쿠폰 이벤트 생성
    Admin->>C1: POST /coupons/events
    C1->>C2: createCouponEvent(totalCount: 100)
    C2->>CE3: save(couponEvent)
    CE3-->>C2: 생성 완료
    C2-->>C1: 이벤트 정보
    C1-->>Admin: 201 Created
    
    Note over Admin,UC3: 2. 사용자 대기열 진입
    A1->>C1: POST /coupons/events/{eventId}/queue
    C1->>C2: joinQueue(userId, eventId)
    C2->>CE3: 이벤트 확인
    CE3-->>C2: 이벤트 정보
    C2->>Q: addToQueue(userId, eventId)
    Q-->>C2: 대기 순번
    C2-->>C1: 대기열 정보
    C1-->>A1: 200 OK (대기 순번)
    
    Note over Admin,UC3: 3. 대기 순번 조회
    A1->>C1: GET /coupons/events/{eventId}/queue/position
    C1->>C2: getQueuePosition(userId, eventId)
    C2->>Q: getPosition(userId, eventId)
    Q-->>C2: 현재 순번
    C2-->>C1: 순번 정보
    C1-->>A1: 200 OK (순번)
    
    Note over Admin,UC3: 4. Worker가 대기열 처리
    loop 대기열 처리
        Worker->>Q: popFromQueue(eventId, 100)
        Q-->>Worker: 100명
        Worker->>CE3: 쿠폰 재고 확인
        CE3-->>Worker: remainingCount
        alt remainingCount > 0
            Worker->>C3: save(coupon)
            C3-->>Worker: 쿠폰 생성
            Worker->>UC3: save(userCoupon)
            UC3-->>Worker: 발급 완료
            Worker->>CE3: remainingCount -= 1
            CE3-->>Worker: 업데이트
        else remainingCount = 0
            Worker->>CE3: status = SOLD_OUT
            CE3-->>Worker: 소진 처리
            Worker->>Q: clearQueue(eventId)
            Q-->>Worker: 대기열 정리
        end
    end
    
    Note over Admin,UC3: 5. 발급 결과 조회
    A1->>C1: GET /coupons/events/{eventId}/result
    C1->>C2: getCouponIssueResult(userId, eventId)
    C2->>UC3: findByUserIdAndEventId(userId, eventId)
    UC3-->>C2: 쿠폰 정보
    C2-->>C1: 발급 결과
    C1-->>A1: 200 OK (쿠폰 정보)

---------- 배송 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant O1 as OrderController
participant O2 as OrderService
participant O3 as OrderRepository
participant D2 as DeliveryService
participant D3 as DeliveryRepository
participant MockAPI as MockDeliveryAPI

    Note over A1,MockAPI: 1. 주문 결제 완료 → 배송 생성
    A1->>O1: POST /orders/{orderId}/payment
    O1->>O2: processPayment(orderId)
    Note right of O2: 결제 성공
    O2->>D2: createDelivery(orderId, address)
    D2->>D3: save(delivery - READY)
    D3-->>D2: 배송 생성
    D2->>MockAPI: registerDelivery()
    MockAPI-->>D2: trackingNumber
    D2->>D3: save(trackingNumber)
    D3-->>D2: 운송장 저장
    D2-->>O2: 배송 생성 완료
    O2-->>O1: 결제 및 배송 완료
    O1-->>A1: 200 OK (trackingNumber)
    
    Note over A1,MockAPI: 2. 배송 시작
    Note right of D2: 관리자 또는 자동
    D2->>D3: findById(deliveryId)
    D3-->>D2: 배송 정보
    D2->>MockAPI: startShipping(trackingNumber)
    MockAPI-->>D2: 배송 시작 확인
    D2->>D3: save(status: SHIPPING)
    D3-->>D2: 상태 업데이트
    D2->>O3: save(orderStatus: SHIPPING)
    O3-->>D2: 주문 상태 업데이트
    
    Note over A1,MockAPI: 3. 배송 추적
    A1->>O1: GET /orders/{orderId}/delivery
    O1->>O2: getOrderDelivery(orderId)
    O2->>D3: findByOrderId(orderId)
    D3-->>O2: 배송 정보
    O2->>MockAPI: getDeliveryStatus(trackingNumber)
    MockAPI-->>O2: 배송 추적 정보
    O2-->>O1: 배송 상태
    O1-->>A1: 200 OK<br/>{status: SHIPPING, location, eta}
    
    Note over A1,MockAPI: 4. 배송 완료
    MockAPI->>D2: deliveryCompleted(trackingNumber)
    D2->>D3: findByTrackingNumber(trackingNumber)
    D3-->>D2: 배송 정보
    D2->>D3: save(status: DELIVERED)
    D3-->>D2: 배송 완료 처리
    D2->>O3: save(orderStatus: DELIVERED)
    O3-->>D2: 주문 완료 처리
    D2-->>MockAPI: 완료 확인
    
    Note over A1,MockAPI: 5. 배송 완료 확인
    A1->>O1: GET /orders/{orderId}
    O1->>O2: getOrder(orderId)
    O2->>O3: findById(orderId)
    O3-->>O2: 주문 정보
    O2-->>O1: 주문 상세
    O1-->>A1: 200 OK<br/>{orderStatus: DELIVERED}

---------- 구매 확정 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant O1 as OrderController
participant O2 as OrderService
participant O3 as OrderRepository
participant D3 as DeliveryRepository

    A1->>O1: POST /orders/{orderId}/confirm
    O1->>O2: confirmOrder(orderId, userId)
    
    Note over O2: 1. 주문 확인
    O2->>O3: findById(orderId)
    alt 주문이 존재하는 경우
        O3-->>O2: 주문 정보
        
        Note over O2: 2. 본인 주문인지 확인
        alt 본인 주문인 경우
            Note over O2: 3. 배송 상태 확인
            O2->>D3: findByOrderId(orderId)
            D3-->>O2: 배송 정보
            alt 배송 완료 상태인 경우 (DELIVERED)
                Note right of O2: 주문 상태: DELIVERED → CONFIRMED<br/>confirmedAt: timestamp
                O2->>O3: save(confirmedOrder)
                O3-->>O2: 구매 확정 완료
                O2-->>O1: 확정 성공
                O1-->>A1: 200 OK<br/>{status: CONFIRMED, canReview: true}
            else 배송 미완료인 경우
                O2-->>O1: DeliveryNotCompletedException
                O1-->>A1: 400 Bad Request<br/>(배송 완료 후 구매 확정 가능)
            end
        else 본인 주문이 아닌 경우
            O2-->>O1: UnauthorizedException
            O1-->>A1: 403 Forbidden
        end
    else 주문이 없는 경우
        O3-->>O2: Empty
        O2-->>O1: OrderNotFoundException
        O1-->>A1: 404 Not Found
    end

---------- 리뷰 작성 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant R1 as ReviewController
participant R2 as ReviewService
participant R3 as ReviewRepository
participant O3 as OrderRepository
participant P3 as ProductRepository
participant U3 as UserRepository

    A1->>R1: POST /reviews
    R1->>R2: createReview(userId, reviewDto)
    
    Note over R2: 1. 유저 확인
    R2->>U3: findById(userId)
    alt 유저가 존재하는 경우
        U3-->>R2: 유저 정보
    else 유저가 없는 경우
        U3-->>R2: Empty
        R2-->>R1: UserNotFoundException
        R1-->>A1: 404 Not Found
    end
    
    Note over R2: 2. 상품 확인
    R2->>P3: findById(productId)
    alt 상품이 존재하는 경우
        P3-->>R2: 상품 정보
    else 상품이 없는 경우
        P3-->>R2: Empty
        R2-->>R1: ProductNotFoundException
        R1-->>A1: 404 Not Found
    end
    
    Note over R2: 3. 구매 확정 여부 확인
    R2->>O3: findByUserIdAndProductIdAndStatus(userId, productId, CONFIRMED)
    alt 구매 확정된 주문이 있는 경우
        O3-->>R2: 주문 정보
        
        Note over R2: 4. 중복 리뷰 확인
        R2->>R3: findByUserIdAndProductIdAndOrderId(userId, productId, orderId)
        alt 이미 리뷰를 작성한 경우
            R3-->>R2: 기존 리뷰
            R2-->>R1: ReviewAlreadyExistsException
            R1-->>A1: 400 Bad Request<br/>(이미 리뷰 작성 완료)
        else 리뷰 미작성인 경우
            Note right of R2: 리뷰 생성<br/>rating(별점 1-5)<br/>content(리뷰 내용)
            R2->>R3: save(review)
            R3-->>R2: 생성된 리뷰
            
            Note over R2: 5. 상품 평점 업데이트
            R2->>R3: getAverageRating(productId)
            R3-->>R2: 평균 별점
            R2->>P3: updateRating(productId, avgRating)
            P3-->>R2: 평점 업데이트 완료
            
            R2-->>R1: 리뷰 정보
            R1-->>A1: 201 Created<br/>{reviewId, rating, content}
        end
    else 구매 확정된 주문이 없는 경우
        O3-->>R2: Empty
        R2-->>R1: OrderNotConfirmedException
        R1-->>A1: 400 Bad Request<br/>(구매 확정 후 리뷰 작성 가능)
    end

---------- 리뷰 조회 (상품별) ----------\

sequenceDiagram
actor A1 as 클라이언트
participant R1 as ReviewController
participant R2 as ReviewService
participant R3 as ReviewRepository
participant U3 as UserRepository

    A1->>R1: GET /products/{productId}/reviews?page=1&size=20&sort=latest
    Note right of R1: sort: latest(최신순)<br/>rating_high(별점높은순)<br/>rating_low(별점낮은순)
    R1->>R2: getProductReviews(productId, PageRequest)
    R2->>R3: findByProductId(productId, Pageable)
    R3-->>R2: Page<Review> (20개)
    
    loop 각 리뷰마다
        R2->>U3: findById(userId)
        U3-->>R2: 유저 정보 (닉네임)
    end
    
    R2-->>R1: 리뷰 목록 + 페이징 정보
    R1-->>A1: 200 OK<br/>{reviews[], avgRating, totalReviews, page}

---------- 리뷰 수정 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant R1 as ReviewController
participant R2 as ReviewService
participant R3 as ReviewRepository
participant P3 as ProductRepository

    A1->>R1: PUT /reviews/{reviewId}
    R1->>R2: updateReview(reviewId, userId, reviewDto)
    
    R2->>R3: findById(reviewId)
    alt 리뷰가 존재하는 경우
        R3-->>R2: 리뷰 정보
        alt 본인이 작성한 리뷰인 경우
            Note right of R2: rating, content 수정
            R2->>R3: save(updatedReview)
            R3-->>R2: 수정된 리뷰
            
            Note over R2: 상품 평점 재계산
            R2->>R3: getAverageRating(productId)
            R3-->>R2: 평균 별점
            R2->>P3: updateRating(productId, avgRating)
            P3-->>R2: 평점 업데이트 완료
            
            R2-->>R1: 수정 완료
            R1-->>A1: 200 OK<br/>{reviewId, rating, content}
        else 본인이 작성한 리뷰가 아닌 경우
            R2-->>R1: UnauthorizedException
            R1-->>A1: 403 Forbidden<br/>(본인 리뷰만 수정 가능)
        end
    else 리뷰가 없는 경우
        R3-->>R2: Empty
        R2-->>R1: ReviewNotFoundException
        R1-->>A1: 404 Not Found
    end

---------- 리뷰 삭제 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant R1 as ReviewController
participant R2 as ReviewService
participant R3 as ReviewRepository
participant RC3 as ReviewCommentRepository
participant P3 as ProductRepository

    A1->>R1: DELETE /reviews/{reviewId}
    R1->>R2: deleteReview(reviewId, userId)
    
    R2->>R3: findById(reviewId)
    alt 리뷰가 존재하는 경우
        R3-->>R2: 리뷰 정보
        alt 본인이 작성한 리뷰인 경우
            Note over R2: 리뷰에 달린 댓글도 삭제
            R2->>RC3: deleteByReviewId(reviewId)
            RC3-->>R2: 댓글 삭제 완료
            
            R2->>R3: delete(reviewId)
            R3-->>R2: 리뷰 삭제 완료
            
            Note over R2: 상품 평점 재계산
            R2->>R3: getAverageRating(productId)
            R3-->>R2: 평균 별점
            R2->>P3: updateRating(productId, avgRating)
            P3-->>R2: 평점 업데이트 완료
            
            R2-->>R1: 삭제 완료
            R1-->>A1: 204 No Content
        else 본인이 작성한 리뷰가 아닌 경우
            R2-->>R1: UnauthorizedException
            R1-->>A1: 403 Forbidden<br/>(본인 리뷰만 삭제 가능)
        end
    else 리뷰가 없는 경우
        R3-->>R2: Empty
        R2-->>R1: ReviewNotFoundException
        R1-->>A1: 404 Not Found
    end

---------- 리뷰 댓글 작성 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant RC1 as ReviewCommentController
participant RC2 as ReviewCommentService
participant RC3 as ReviewCommentRepository
participant R3 as ReviewRepository
participant U3 as UserRepository

    A1->>RC1: POST /reviews/{reviewId}/comments
    RC1->>RC2: createComment(reviewId, userId, commentDto)
    
    Note over RC2: 1. 유저 확인
    RC2->>U3: findById(userId)
    alt 유저가 존재하는 경우
        U3-->>RC2: 유저 정보
    else 유저가 없는 경우
        U3-->>RC2: Empty
        RC2-->>RC1: UserNotFoundException
        RC1-->>A1: 404 Not Found
    end
    
    Note over RC2: 2. 리뷰 확인
    RC2->>R3: findById(reviewId)
    alt 리뷰가 존재하는 경우
        R3-->>RC2: 리뷰 정보
        
        Note right of RC2: 댓글 생성
        RC2->>RC3: save(comment)
        RC3-->>RC2: 생성된 댓글
        
        RC2-->>RC1: 댓글 정보
        RC1-->>A1: 201 Created<br/>{commentId, content}
    else 리뷰가 없는 경우
        R3-->>RC2: Empty
        RC2-->>RC1: ReviewNotFoundException
        RC1-->>A1: 404 Not Found
    end

---------- 리뷰 댓글 조회 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant RC1 as ReviewCommentController
participant RC2 as ReviewCommentService
participant RC3 as ReviewCommentRepository
participant U3 as UserRepository

    A1->>RC1: GET /reviews/{reviewId}/comments?page=1&size=20
    RC1->>RC2: getComments(reviewId, PageRequest)
    RC2->>RC3: findByReviewId(reviewId, Pageable)
    RC3-->>RC2: Page<Comment> (20개)
    
    loop 각 댓글마다
        RC2->>U3: findById(userId)
        U3-->>RC2: 유저 정보 (닉네임)
    end
    
    RC2-->>RC1: 댓글 목록 + 페이징 정보
    RC1-->>A1: 200 OK<br/>{comments[], page, totalPages}

---------- 리뷰 댓글 수정 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant RC1 as ReviewCommentController
participant RC2 as ReviewCommentService
participant RC3 as ReviewCommentRepository

    A1->>RC1: PUT /comments/{commentId}
    RC1->>RC2: updateComment(commentId, userId, commentDto)
    
    RC2->>RC3: findById(commentId)
    alt 댓글이 존재하는 경우
        RC3-->>RC2: 댓글 정보
        alt 본인이 작성한 댓글인 경우
            Note right of RC2: content 수정
            RC2->>RC3: save(updatedComment)
            RC3-->>RC2: 수정된 댓글
            RC2-->>RC1: 수정 완료
            RC1-->>A1: 200 OK<br/>{commentId, content}
        else 본인이 작성한 댓글이 아닌 경우
            RC2-->>RC1: UnauthorizedException
            RC1-->>A1: 403 Forbidden
        end
    else 댓글이 없는 경우
        RC3-->>RC2: Empty
        RC2-->>RC1: CommentNotFoundException
        RC1-->>A1: 404 Not Found
    end

---------- 리뷰 댓글 삭제 ----------\

sequenceDiagram
actor A1 as 클라이언트
participant RC1 as ReviewCommentController
participant RC2 as ReviewCommentService
participant RC3 as ReviewCommentRepository

    A1->>RC1: DELETE /comments/{commentId}
    RC1->>RC2: deleteComment(commentId, userId)
    
    RC2->>RC3: findById(commentId)
    alt 댓글이 존재하는 경우
        RC3-->>RC2: 댓글 정보
        alt 본인이 작성한 댓글인 경우
            RC2->>RC3: delete(commentId)
            RC3-->>RC2: 삭제 완료
            RC2-->>RC1: 삭제 성공
            RC1-->>A1: 204 No Content
        else 본인이 작성한 댓글이 아닌 경우
            RC2-->>RC1: UnauthorizedException
            RC1-->>A1: 403 Forbidden
        end
    else 댓글이 없는 경우
        RC3-->>RC2: Empty
        RC2-->>RC1: CommentNotFoundException
        RC1-->>A1: 404 Not Found
    end
