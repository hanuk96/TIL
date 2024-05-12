도메인 주도 개발 시작하기를 보고 정리한다.

Directory 구조
└── src
      ├── main
      │   ├── java
      │   │   └── com
      │   │       └── myshop
      │   │           └── order
      │   │               └── command
      │   │                   └── application (application layer)
      │   │                       ├── Request (value)
      │   │                       ├── Response (value)
      │   │                       └── OrderService
      │   │                   └── domain (domain layer)
      │   │                       ├── OrderState (value)
      │   │                       ├── Order (entity)
      │   │                       └── OrderRepository (respository - domain model이 속함)
      │   │               └── query      
      │   │                   └── application (application layer)
      │   │                       ├── Request (value)
      │   │                       ├── Response (value)
      │   │                       └── OrderDetailService
      │   │                   └── dao - data access 용도
      │   │                       └── OrderSummaryDao (dao)
      │   │                   └── dto
      │   │                       └── OrderSummary (OrderSummaryDao에서 사용할 dto)
      │   │               └── infra
      │   │                   ├── domain (domain layer)
      │   │                   └── application (presentation layer)
      │   │               └── ui
      │   │                   ├── exception (exception, excetion handler)
      │   │                   └── controller (presentation layer)
      │   │           ├── board
      │   │           ├── member
      │   │           ├── catalog
      │   │           └── common
      │   │               ├── event (event handler)
      │   │               ├── jpa (sort, pagination, spec builder ...)
      │   │               └── model (공통 model 이메일, 주소 ...)
      │   │           ├── lock
      │   │           └── config (spring과 관련된 여러 설정, 공통 bean ...)
└── resource
