
Table users { 
    id varchar [pk]
    password varchar [not null]
    username varchar [not null, unique] // 실제 로그인에 사용할 Id
    name varchar // 실명
    email varchar [unique] // 이메일
    phone varchar // 전화번호
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    indexes {
        (username)
        (email)
    }
}

Table categories {
    id varchar [pk]
    category_name varchar [not null, unique]
    display_order int [default: 0]
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
}

Table products {
    id varchar [pk]
    name varchar [not null]
    description text
    price decimal(10,2) [not null]
    stock int [not null, default: 0]
    category_id varchar [ref: > categories.id]
    is_active boolean [default: true]
    view_count int [default: 0]  // 조회수 (인기순 정렬용)
    sold_count int [default: 0]  // 판매량 (인기순 정렬용)
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    indexes {
        (category_id)
        (is_active)
        (created_at)
    }
}

Table product_images {
    id varchar [pk]
    product_id varchar [ref: > products.id, not null]
    image_url varchar [not null]
    display_order int [default: 0] // 이미지 순서
    is_thumbnail boolean [default: false] // 대표 이미지 여부
    created_at timestamp [not null, default: `now()`]
    indexes {
        (product_id)
        (display_order)
    }
}

Table orders {
    id varchar [pk]
    user_id varchar [ref: > users.id, not null]
    total_amount decimal(10,2) [not null]
    discount_amount decimal(10,2) [default: 0]
    final_amount decimal(10,2) [not null]
    status varchar [not null, default: 'PENDING'] // PENDING, PAID, CONFIRMED, CANCELLED
    coupon_id varchar [ref: > coupons.id]  // 사용된 쿠폰
    point_amount decimal(10,2) [default: 0]  // 사용된 포인트
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    paid_at timestamp
    cancelled_at timestamp
    indexes {
        (user_id)
        (status)
        (created_at)
    }
}

Table payments {
    id varchar [pk]
    order_id varchar [ref: > orders.id, not null]
    amount decimal(10,2) [not null]
    payment_type varchar [not null]  // PAYMENT, REFUND
    payment_method varchar [not null]  // CARD, BANK_TRANSFER, KAKAO_PAY, TOSS 등
    payment_status varchar [not null, default: 'PENDING']  // PENDING, COMPLETED, FAILED, REFUNDED
    // 결제 정보
    transaction_id varchar [unique]  // PG사 거래 ID
    pg_provider varchar  // 결제 대행사 (토스, 나이스페이 등)
    // 실패 사유
    failure_reason text
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    completed_at timestamp
    failed_at timestamp
    indexes {
        (order_id)
        (transaction_id)
        (payment_status)
    }
}

Table order_items {
    id varchar [pk]
    product_id varchar [ref: > products.id, not null]
    order_id varchar [ref: > orders.id, not null]
    product_name varchar [not null]  // 주문 당시 상품명
    quantity int [not null]
    unit_price decimal(10,2) [not null]
    subtotal decimal(10,2) [not null] // quantity * unit_price
    // 주문 취소
    cancel_status varchar  // null, REQUESTED, APPROVED, REJECTED 주문 취소 단계
    cancel_reason text  // 주문 취소 사유
    // 반품
    return_status varchar  // null, REQUESTED, APPROVED, REJECTED 반품 단계
    return_reason text  // 반품 사유
    // 환불
    refund_status varchar  // null, REQUESTED, APPROVED, REJECTED  // 환불 단계
    refund_reason text  // 환불 사유
    // 구매 확정
    is_confirmed boolean [default: false]
    confirmed_at timestamp
    cancelled_at timestamp
    returned_at timestamp
    refunded_at timestamp
    created_at timestamp [not null, default: `now()`]
    indexes {
        (order_id)
        (product_id)
        (is_confirmed)
    }
}

Table carts {
    id varchar [pk]
    user_id varchar [ref: > users.id, not null]
    product_id varchar [ref: > products.id, not null]
    quantity int [not null, default: 1]
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    indexes {
        (user_id, product_id) [unique]
    }
}

Table deliveries {
    id varchar [pk]
    order_item_id varchar [ref: > order_items.id, not null]
    // 배송지 정보
    receiver_name varchar [not null]
    receiver_phone varchar [not null]
    shipping_address text [not null]
    postal_code varchar
    delivery_memo text  // 배송 요청사항
    // 택배 정보
    parcel_number varchar [unique] // 송장번호
    parcel_corp varchar [not null] // CJ대한통운, 우체국택배 등
    // 배송 상태
    delivery_status varchar [not null, default: 'PREPARING']
    // PREPARING(준비중), SHIPPED(배송중), DELIVERED(배송완료), RETURNED(반송)
    // 시간 정보
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    shipped_at timestamp  // 출고 시간
    delivered_at timestamp  // 배송완료 시간
    indexes {
        (order_item_id)
        (parcel_number)
        (delivery_status)
    }
}

Table points {
    id varchar [pk]
    user_id varchar [ref: > users.id, not null]
    amount decimal(10,2) [not null]
    point_type varchar [not null]  // EARNED, USED, EXPIRED, REFUNDED
    description varchar  // 적립/사용 사유
    order_id varchar [ref: > orders.id]  // 주문 관련 포인트일 경우
    expires_at timestamp  // 포인트 만료일
    created_at timestamp [not null, default: `now()`]
    indexes {
        (user_id)
        (order_id)
        (created_at)
    }
}

Table coupons {
    id varchar [pk]
    name varchar [not null] // "신규가입 쿠폰", "5월 할인 이벤트"
    code varchar [unique] // "WELCOME2025" (입력형 쿠폰)
    // 할인 정보
    discount_type varchar [not null] // PERCENTAGE(% 정률), FIXED(정액)
    discount_value decimal(10,2) [not null] // 10 (10% 또는 10,000원)
    max_discount_amount decimal(10,2) // 최대 할인 금액 (% 쿠폰일 때)
    min_order_amount decimal(10,2) // 최소 주문 금액
    // 수량 관리
    total_quantity int // 총 발급 가능 수량 (null이면 무제한)
    issued_quantity int [default: 0] // 발급된 수량
    usage_count int [default: 0] // 현재까지 사용된 횟수
    per_user_limit int [default: 1] // 1인당 사용 가능 횟수
    // 유효 기간
    valid_from timestamp [not null]
    valid_until timestamp [not null]
    is_active boolean [default: true]
    created_at timestamp [not null, default: `now()`]
    indexes {
        (code)
        (is_active)
    }
}

Table user_coupons {
    id varchar [pk]
    coupon_id varchar [ref: > coupons.id, not null]
    user_id varchar [ref: > users.id, not null]
    status varchar [not null, default: 'AVAILABLE'] // AVAILABLE, USED, EXPIRED
    used_at timestamp  // 사용 시간
    expires_at timestamp [not null]  // 만료 시간
    order_id varchar [ref: > orders.id]  // 사용된 주문
    issued_at timestamp [not null, default: `now()`]  // 발급 시간
    indexes {
        (user_id, coupon_id)
        (user_id, status)
        (expires_at)
    }
}

Table reviews {
    id varchar [pk]
    user_id varchar [ref: > users.id, not null]
    product_id varchar [ref: > products.id, not null]
    order_item_id varchar [ref: > order_items.id, unique, not null]
    rating decimal(2,1) [not null]  // 평점 (1.0 ~ 5.0)
    content text [not null]  // varchar → text로 변경
    // 이미지
    image_urls text  // JSON 배열 또는 쉼표로 구분
    // 관리
    is_visible boolean [default: true]  // 숨김 처리 가능
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    indexes {
        (product_id)
        (user_id)
        (order_item_id)
        (created_at)
    }
}

Table review_comments {
    id varchar [pk]
    user_id varchar [ref: > users.id, not null]
    review_id varchar [ref: > reviews.id, not null]
    parent_comment_id varchar [ref: > review_comments.id]  // 대댓글용
    content text [not null]
    // 판매자 댓글 구분
    is_seller boolean [default: false]  // 판매자가 단 댓글인지
    is_visible boolean [default: true]
    created_at timestamp [not null, default: `now()`]
    updated_at timestamp [not null, default: `now()`]
    indexes {
        (review_id)
        (parent_comment_id)
        (user_id)
        (created_at)
    }
}