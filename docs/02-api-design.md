# API 설계서

## 1. 상품  (Product)

### 1.1 전체 상품 조회
```
GET /api/products/{kind}
```
**Query Parameters:**
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)
- `sort`: "latest" | "orderCount" | "rating" (optional)
  - `latest`: 최신순
  - `orderCount`: 주문많은순
  - `rating`: 평점높은순

**Response (200 OK):**
```json
{
  "products": [
    {
      "productId": "string",
      "name": "string",
      "price": number,
      "stock": number,
      "category": "string",
      "rating": number,
      "orderCount": number,
      "createdAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number,
  "totalElements": number
}
```

### 1.2 상품 상세 조회
```
GET /api/products/{id}
```

**Response (200 OK):**
```json
{
  "productId": "string",
  "name": "string",
  "description": "string",
  "price": number,
  "stock": number,
  "category": {
    "categoryId": "string",
    "name": "string"
  },
  "rating": number,
  "reviewCount": number,
  "orderCount": number,
  "saleStatus": "ON_SALE" | "SOLD_OUT" | "DISCONTINUED",
  "createdAt": "datetime",
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.3 상품 등록
```
POST /api/products
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "price": number,
  "stock": number,
  "categoryId": "string"
}
```

**Response (201 Created):**
```json
{
  "productId": "string",
  "name": "string",
  "description": "string",
  "price": number,
  "stock": number,
  "categoryId": "string",
  "createdAt": "datetime"
}
```

### 1.4 상품 수정
```
PUT /api/products/{id}
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "price": number,
  "stock": number,
  "categoryId": "string"
}
```

**Response (200 OK):**
```json
{
  "productId": "string",
  "name": "string",
  "description": "string",
  "price": number,
  "stock": number,
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.5 상품 삭제
```
DELETE /api/products/{id}
```

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.6 재고 수정
```
PATCH /api/products/{id}/stock
```

**Request Body:**
```json
{
  "quantity": number
}
```

**Response (200 OK):**
```json
{
  "productId": "string",
  "stock": number,
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.7 가격 수정
```
PATCH /api/products/{id}/price
```

**Request Body:**
```json
{
  "price": number
}
```

**Response (200 OK):**
```json
{
  "productId": "string",
  "price": number,
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.8 품절 상태 변경
```
PATCH /api/products/{id}/stock-status
```

**Request Body:**
```json
{
  "status": "IN_STOCK" | "OUT_OF_STOCK"
}
```

**Response (200 OK):**
```json
{
  "productId": "string",
  "stockStatus": "IN_STOCK" | "OUT_OF_STOCK",
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.9 판매 상태 변경
```
PATCH /api/products/{id}/sale-status
```

**Request Body:**
```json
{
  "status": "ON_SALE" | "SOLD_OUT" | "DISCONTINUED"
}
```

**Response (200 OK):**
```json
{
  "productId": "string",
  "saleStatus": "ON_SALE" | "SOLD_OUT" | "DISCONTINUED",
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

### 1.10 상품 검색
```
GET /api/products/search
```

**Query Parameters:**
- `keyword`: string (required) - 검색어 (상품명, 카테고리명)
- `categoryId`: string (optional) - 카테고리 필터
- `minPrice`: number (optional) - 최소 가격
- `maxPrice`: number (optional) - 최대 가격
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)
- `sort`: "latest" | "price_low" | "price_high" | "orderCount" | "rating" (optional)

**Response (200 OK):**
```json
{
  "products": [
    {
      "productId": "string",
      "name": "string",
      "price": number,
      "stock": number,
      "category": {
        "categoryId": "string",
        "name": "string"
      },
      "rating": number,
      "orderCount": number,
      "thumbnailUrl": "string"
    }
  ],
  "keyword": "string",
  "page": number,
  "totalPages": number,
  "totalElements": number
}
```

---

## 2. 카테고리 (Category)

### 2.1 카테고리 전체 조회
```
GET /api/categories
```

**Response (200 OK):**
```json
{
  "categories": [
    {
      "categoryId": "string",
      "name": "string",
      "description": "string",
      "productCount": number
    }
  ]
}
```

### 2.2 카테고리 단건 조회
```
GET /api/categories/{id}
```

**Response (200 OK):**
```json
{
  "categoryId": "string",
  "name": "string",
  "description": "string",
  "productCount": number,
  "createdAt": "datetime"
}
```

### 2.3 카테고리 생성
```
POST /api/categories
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string"
}
```

**Response (201 Created):**
```json
{
  "categoryId": "string",
  "name": "string",
  "description": "string",
  "createdAt": "datetime"
}
```

### 2.4 카테고리 수정
```
PUT /api/categories/{id}
```

**Request Body:**
```json
{
  "name": "string",
  "description": "string"
}
```

**Response (200 OK):**
```json
{
  "categoryId": "string",
  "name": "string",
  "description": "string",
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 카테고리를 찾을 수 없음

### 2.5 카테고리 삭제
```
DELETE /api/categories/{id}
```

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 카테고리를 찾을 수 없음

### 2.6 카테고리별 상품 목록 조회
```
GET /api/categories/{categoryId}/products
```

**Query Parameters:**
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)
- `sort`: "latest" | "orderCount" | "rating" (optional)

**Response (200 OK):**
```json
{
  "category": {
    "categoryId": "string",
    "name": "string"
  },
  "products": [
    {
      "productId": "string",
      "name": "string",
      "price": number,
      "stock": number,
      "rating": number
    }
  ],
  "page": number,
  "totalPages": number,
  "totalElements": number
}
```

**Error Response:**
- `404 Not Found`: 카테고리를 찾을 수 없음

---

## 3. 찜하기 / 위시리스트 (Wishlist)

### 3.1 상품 찜하기
```
POST /api/wishlists
```

**Request Body:**
```json
{
  "userId": "string",
  "productId": "string"
}
```

**Response (201 Created):**
```json
{
  "wishlistId": "string",
  "productId": "string",
  "addedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 유저 또는 상품을 찾을 수 없음
- `400 Bad Request`: 이미 찜한 상품

### 3.2 찜 목록 조회
```
GET /api/users/{userId}/wishlists
```

**Query Parameters:**
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)
- `sort`: "latest" | "oldest" (optional, default: "latest")

**Response (200 OK):**
```json
{
  "wishlists": [
    {
      "wishlistId": "string",
      "product": {
        "productId": "string",
        "name": "string",
        "price": number,
        "stock": number,
        "thumbnailUrl": "string",
        "saleStatus": "ON_SALE" | "SOLD_OUT" | "DISCONTINUED"
      },
      "addedAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number,
  "totalElements": number
}
```

### 3.3 찜 삭제
```
DELETE /api/wishlists/{wishlistId}
```

**Query Parameters:**
- `userId`: string

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 찜을 찾을 수 없음
- `403 Forbidden`: 본인 찜이 아님

### 3.4 상품 찜 여부 확인
```
GET /api/wishlists/check
```

**Query Parameters:**
- `userId`: string
- `productId`: string

**Response (200 OK):**
```json
{
  "isWishlisted": boolean,
  "wishlistId": "string"
}
```

---

## 4. 최근 본 상품 (Recent Products)

### 4.1 최근 본 상품 조회
```
GET /api/users/{userId}/recent-products
```

**Query Parameters:**
- `limit`: number (optional, default: 20, max: 50)

**Response (200 OK):**
```json
{
  "products": [
    {
      "productId": "string",
      "name": "string",
      "price": number,
      "stock": number,
      "thumbnailUrl": "string",
      "viewedAt": "datetime"
    }
  ]
}
```

### 4.2 최근 본 상품 추가
```
POST /api/users/{userId}/recent-products
```

**Request Body:**
```json
{
  "productId": "string"
}
```

**Response (201 Created):**
```json
{
  "productId": "string",
  "viewedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 상품을 찾을 수 없음

---

## 5. 주문 / 결제 (Order / Payment)

### 5.1 주문 생성
```
POST /api/orders
```

**Request Body:**
```json
{
  "userId": "string",
  "items": [
    {
      "productId": "string",
      "quantity": number
    }
  ],
  "usePoint": number,
  "couponId": "string" (optional)
}
```

**Response (201 Created):**
```json
{
  "orderId": "string",
  "totalPrice": number,
  "discountedPrice": number,
  "finalPrice": number,
  "status": "PENDING",
  "createdAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`:  유저 또는 상품을 찾을 수 없음
- `400 Bad Request`: 재고 부족, 포인트 부족

### 3.2 결제 처리
```
POST /api/orders/{orderId}/payment
```

**Request Body:**
```json
{
  "paymentMethod": "CARD" | "BANK_TRANSFER" | "VIRTUAL_ACCOUNT"
}
```

**Response (200 OK):**
```json
{
  "paymentId": "string",
  "orderId": "string",
  "status": "PAID",
  "paidAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 주문을 찾을 수 없음
- `400 Bad Request`: 결제 실패 (재고/포인트 복구됨)

### 3.3 장바구니에서 주문 생성
```
POST /api/orders/from-cart
```

**Request Body:**
```json
{
  "userId": "string",
  "cartId": "string",
  "usePoint": number,
  "couponId": "string" (optional)
}
```

**Response (201 Created):**
```json
{
  "orderId": "string",
  "finalPrice": number,
  "status": "PENDING",
  "createdAt": "datetime"
}
```

**Error Response:**
- `400 Bad Request`: 상품 없음, 재고 부족
- `404 Not Found`: 장바구니를 찾을 수 없음

### 3.4 주문 조회 (단건)
```
GET /api/orders/{orderId}
```

**Response (200 OK):**
```json
{
  "orderId": "string",
  "userId": "string",
  "items": [
    {
      "productId": "string",
      "productName": "string",
      "quantity": number,
      "price": number
    }
  ],
  "totalPrice": number,
  "finalPrice": number,
  "status": "PENDING" | "PAID" | "SHIPPING" | "DELIVERED" | "CONFIRMED" | "FAILED" | "CANCELLED",
  "createdAt": "datetime",
  "paidAt": "datetime",
  "deliveredAt": "datetime",
  "confirmedAt": "datetime"
}
```

### 3.5 사용자별 주문 목록 조회
```
GET /api/users/{userId}/orders
```

**Query Parameters:**
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)
- `status`: "PENDING" | "PAID" | "SHIPPING" | "DELIVERED" | "CONFIRMED" | "CANCELLED" | "FAILED" (optional)
- `sort`: "latest" | "oldest" (optional, default: "latest")

**Response (200 OK):**
```json
{
  "orders": [
    {
      "orderId": "string",
      "items": [
        {
          "productId": "string",
          "productName": "string",
          "quantity": number,
          "price": number,
          "thumbnailUrl": "string"
        }
      ],
      "totalPrice": number,
      "finalPrice": number,
      "status": "PENDING" | "PAID" | "SHIPPING" | "DELIVERED" | "CONFIRMED" | "CANCELLED" | "FAILED",
      "createdAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number,
  "totalElements": number
}
```

### 3.6 주문 취소
```
POST /api/orders/{orderId}/cancel
```

**Request Body:**
```json
{
  "userId": "string",
  "reason": "string"
}
```

**Response (200 OK):**
```json
{
  "orderId": "string",
  "status": "CANCELLED",
  "refundedPoint": number,
  "refundedAmount": number,
  "cancelledAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 주문을 찾을 수 없음
- `403 Forbidden`: 본인 주문이 아님
- `400 Bad Request`: 취소 불가 상태 (배송 시작 후 취소 불가)

### 3.7 구매 확정
```
POST /api/orders/{orderId}/confirm
```

**Request Body:**
```json
{
  "userId": "string"
}
```

**Response (200 OK):**
```json
{
  "orderId": "string",
  "status": "CONFIRMED",
  "canReview": true,
  "confirmedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 주문을 찾을 수 없음
- `403 Forbidden`: 본인 주문이 아님
- `400 Bad Request`: 배송 미완료 (배송 완료 후 구매 확정 가능)

---

## 4. 장바구니 (Cart)

### 4.1 장바구니 아이템 추가
```
POST /api/cart/items
```

**Request Body:**
```json
{
  "userId": "string",
  "productId": "string",
  "quantity": number
}
```

**Response (201 Created):**
```json
{
  "cartItemId": "string",
  "productId": "string",
  "quantity": number,
  "addedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 유저 또는 상품을 찾을 수 없음

### 4.2 장바구니 조회
```
GET /api/{userId}/cart
```

**Query Parameters:**
- `userId`: string

**Response (200 OK):**
```json
{
  "cartId": "string",
  "userId": "string",
  "cartItems": [
    {
      "cartItemId": "string",
      "product": {
        "productId": "string",
        "name": "string",
        "price": number,
        "stock": number
      },
      "quantity": number,
      "subtotal": number
    }
  ],
  "totalPrice": number
}
```

### 4.3 장바구니 아이템 수량 변경
```
PATCH /api/cart/items/{itemId}
```

**Request Body:**
```json
{
  "quantity": number
}
```

**Response (200 OK):**
```json
{
  "cartItemId": "string",
  "quantity": number,
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 장바구니 아이템을 찾을 수 없음

### 4.4 장바구니 아이템 삭제
```
DELETE /api/cart/items/{itemId}
```

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 장바구니 아이템을 찾을 수 없음

### 4.5 장바구니 전체 삭제
```
DELETE /api/users/{userId}/cart
```

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 장바구니를 찾을 수 없음

---

## 5. 포인트 (Point)

### 5.1 포인트 충전
```
POST /api/points/charge
```

**Request Body:**
```json
{
  "userId": "string",
  "amount": number,
  "paymentMethod": "CARD" | "BANK_TRANSFER"
}
```

**Response (200 OK):**
```json
{
  "paymentId": "string",
  "chargedAmount": number,
  "totalPoint": number,
  "chargedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 유저를 찾을 수 없음
- `400 Bad Request`: 결제 실패

### 5.2 포인트 잔액 조회
```
GET /api/points
```

**Query Parameters:**
- `userId`: string

**Response (200 OK):**
```json
{
  "userId": "string",
  "point": number,
  "updatedAt": "datetime"
}
```

### 5.3 포인트 내역 조회 (전체)
```
GET /api/points/history
```

**Query Parameters:**
- `userId`: string
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)

**Response (200 OK):**
```json
{
  "history": [
    {
      "historyId": "string",
      "type": "CHARGE" | "USE" | "EARN" | "REFUND",
      "amount": number,
      "balance": number,
      "description": "string",
      "createdAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number
}
```

### 5.4 포인트 내역 조회 (타입별)
```
GET /api/points/history
```

**Query Parameters:**
- `userId`: string
- `type`: "CHARGE" | "USE" | "EARN" | "REFUND"
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)

**Response (200 OK):**
```json
{
  "history": [
    {
      "historyId": "string",
      "type": "CHARGE" | "USE" | "EARN" | "REFUND",
      "amount": number,
      "balance": number,
      "description": "string",
      "createdAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number
}
```

### 5.5 포인트 내역 조회 (기간별)
```
GET /api/points/history
```

**Query Parameters:**
- `userId`: string
- `startDate`: date (YYYY-MM-DD)
- `endDate`: date (YYYY-MM-DD)
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)

**Response (200 OK):**
```json
{
  "history": [
    {
      "historyId": "string",
      "type": "CHARGE" | "USE" | "EARN" | "REFUND",
      "amount": number,
      "balance": number,
      "description": "string",
      "createdAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number
}
```

### 5.6 포인트 통계 조회
```
GET /api/points/summary
```

**Query Parameters:**
- `userId`: string

**Response (200 OK):**
```json
{
  "currentPoint": number,
  "totalCharged": number,
  "totalUsed": number,
  "totalEarned": number,
  "totalRefunded": number
}
```

---

## 6. 쿠폰 (Coupon)

### 6.1 쿠폰 이벤트 생성
```
POST /api/coupons/events
```

**Request Body:**
```json
{
  "name": "string",
  "totalCount": number,
  "discountType": "PERCENTAGE" | "FIXED_AMOUNT",
  "discountValue": number,
  "startAt": "datetime",
  "endAt": "datetime"
}
```

**Response (201 Created):**
```json
{
  "eventId": "string",
  "name": "string",
  "totalCount": number,
  "remainingCount": number,
  "status": "ACTIVE",
  "createdAt": "datetime"
}
```

### 6.2 대기열 진입
```
POST /api/coupons/events/{eventId}/queue
```

**Request Body:**
```json
{
  "userId": "string"
}
```

**Response (200 OK):**
```json
{
  "eventId": "string",
  "userId": "string",
  "position": number,
  "estimatedWaitTime": number (seconds)
}
```

### 6.3 대기 순번 조회
```
GET /api/coupons/events/{eventId}/queue/position
```

**Query Parameters:**
- `userId`: string

**Response (200 OK):**
```json
{
  "eventId": "string",
  "userId": "string",
  "position": number,
  "estimatedWaitTime": number (seconds)
}
```

### 6.4 쿠폰 발급 결과 조회
```
GET /api/coupons/events/{eventId}/result
```

**Query Parameters:**
- `userId`: string

**Response (200 OK):**
```json
{
  "eventId": "string",
  "userId": "string",
  "issued": boolean,
  "coupon": {
    "couponId": "string",
    "name": "string",
    "discountType": "PERCENTAGE" | "FIXED_AMOUNT",
    "discountValue": number,
    "expiresAt": "datetime"
  }
}
```

### 6.5 보유 쿠폰 조회
```
GET /api/coupons/my
```

**Query Parameters:**
- `userId`: string

**Response (200 OK):**
```json
{
  "coupons": [
    {
      "couponId": "string",
      "name": "string",
      "discountType": "PERCENTAGE" | "FIXED_AMOUNT",
      "discountValue": number,
      "isUsed": boolean,
      "expiresAt": "datetime"
    }
  ]
}
```

---

## 7. 배송 (Delivery)

### 7.1 배송 조회
```
GET /api/orders/{orderId}/delivery
```

**Response (200 OK):**
```json
{
  "deliveryId": "string",
  "orderId": "string",
  "trackingNumber": "string",
  "status": "READY" | "SHIPPING" | "DELIVERED",
  "address": {
    "zipCode": "string",
    "street": "string",
    "detail": "string"
  },
  "currentLocation": "string",
  "estimatedDeliveryAt": "datetime",
  "deliveredAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 주문 또는 배송을 찾을 수 없음

### 7.2 배송 추적
```
GET /api/deliveries/{trackingNumber}/track
```

**Response (200 OK):**
```json
{
  "trackingNumber": "string",
  "status": "READY" | "SHIPPING" | "DELIVERED",
  "currentLocation": "string",
  "estimatedDeliveryAt": "datetime",
  "history": [
    {
      "location": "string",
      "status": "string",
      "timestamp": "datetime"
    }
  ]
}
```

---

## 8. 리뷰 (Review)

### 8.1 리뷰 작성
```
POST /api/reviews
```

**Request Body:**
```json
{
  "userId": "string",
  "productId": "string",
  "orderId": "string",
  "rating": number (1-5),
  "content": "string"
}
```

**Response (201 Created):**
```json
{
  "reviewId": "string",
  "productId": "string",
  "userId": "string",
  "rating": number,
  "content": "string",
  "createdAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 유저 또는 상품을 찾을 수 없음
- `400 Bad Request`: 구매 확정 전 리뷰 작성 불가, 이미 리뷰 작성 완료

### 8.2 상품별 리뷰 조회
```
GET /api/products/{productId}/reviews
```

**Query Parameters:**
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)
- `sort`: "latest" | "rating_high" | "rating_low" (optional)

**Response (200 OK):**
```json
{
  "reviews": [
    {
      "reviewId": "string",
      "user": {
        "userId": "string",
        "nickname": "string"
      },
      "rating": number,
      "content": "string",
      "createdAt": "datetime"
    }
  ],
  "avgRating": number,
  "totalReviews": number,
  "page": number,
  "totalPages": number
}
```

### 8.3 리뷰 수정
```
PUT /api/reviews/{reviewId}
```

**Request Body:**
```json
{
  "userId": "string",
  "rating": number (1-5),
  "content": "string"
}
```

**Response (200 OK):**
```json
{
  "reviewId": "string",
  "rating": number,
  "content": "string",
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 리뷰를 찾을 수 없음
- `403 Forbidden`: 본인 리뷰가 아님

### 8.4 리뷰 삭제
```
DELETE /api/reviews/{reviewId}
```

**Query Parameters:**
- `userId`: string

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 리뷰를 찾을 수 없음
- `403 Forbidden`: 본인 리뷰가 아님

---

## 9. 리뷰 댓글 (Review Comment)

### 9.1 댓글 작성
```
POST /api/reviews/{reviewId}/comments
```

**Request Body:**
```json
{
  "userId": "string",
  "content": "string"
}
```

**Response (201 Created):**
```json
{
  "commentId": "string",
  "reviewId": "string",
  "userId": "string",
  "content": "string",
  "createdAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 유저 또는 리뷰를 찾을 수 없음

### 9.2 댓글 조회
```
GET /api/reviews/{reviewId}/comments
```

**Query Parameters:**
- `page`: number (optional, default: 1)
- `size`: number (optional, default: 20)

**Response (200 OK):**
```json
{
  "comments": [
    {
      "commentId": "string",
      "user": {
        "userId": "string",
        "nickname": "string"
      },
      "content": "string",
      "createdAt": "datetime"
    }
  ],
  "page": number,
  "totalPages": number
}
```

### 9.3 댓글 수정
```
PUT /api/comments/{commentId}
```

**Request Body:**
```json
{
  "userId": "string",
  "content": "string"
}
```

**Response (200 OK):**
```json
{
  "commentId": "string",
  "content": "string",
  "updatedAt": "datetime"
}
```

**Error Response:**
- `404 Not Found`: 댓글을 찾을 수 없음
- `403 Forbidden`: 본인 댓글이 아님

### 9.4 댓글 삭제
```
DELETE /api/comments/{commentId}
```

**Query Parameters:**
- `userId`: string

**Response:**
- `204 No Content`: 삭제 성공

**Error Response:**
- `404 Not Found`: 댓글을 찾을 수 없음
- `403 Forbidden`: 본인 댓글이 아님

---

## 공통 응답 형식

### Error Response
모든 에러는 다음과 같은 공통 형식으로 응답합니다:

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "timestamp": "datetime"
  }
}
```

### HTTP Status Codes
- `200 OK`: 요청 성공
- `201 Created`: 생성 성공
- `204 No Content`: 삭제 성공
- `400 Bad Request`: 잘못된 요청
- `403 Forbidden`: 권한 없음
- `404 Not Found`: 리소스를 찾을 수 없음
- `500 Internal Server Error`: 서버 오류