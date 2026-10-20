# Take Out Backend

A Spring Boot backend learning project for a food delivery application, covering both customer ordering and restaurant administration. I used the `sky-take-out` scaffold to practice building a multi-module Java application with authentication, business workflows, caching, background tasks, and reporting.

## My work

| Area | What I implemented | Code |
| --- | --- | --- |
| Authentication | Developed employee login and management APIs, integrated WeChat login for customers, and added separate JWT interceptors for admin and customer routes. | [EmployeeController](sky-server/src/main/java/com/sky/controller/admin/EmployeeController.java), [UserServiceImpl](sky-server/src/main/java/com/sky/service/impl/UserServiceImpl.java), [interceptors](sky-server/src/main/java/com/sky/interceptor) |
| Menu administration | Implemented category, dish, flavor, and set-meal management, including pagination, batch deletion, availability changes, and checks on related records. | [DishServiceImpl](sky-server/src/main/java/com/sky/service/impl/DishServiceImpl.java), [SetmealServiceImpl](sky-server/src/main/java/com/sky/service/impl/SetmealServiceImpl.java), [CategoryServiceImpl](sky-server/src/main/java/com/sky/service/impl/CategoryServiceImpl.java) |
| Shared backend behavior | Added an annotation and AOP aspect to fill audit fields, custom JSON conversion, exception handling, and an Aliyun OSS upload endpoint. | [AutofillAspect](sky-server/src/main/java/com/sky/aspect/AutofillAspect.java), [WebMvcConfiguration](sky-server/src/main/java/com/sky/config/WebMvcConfiguration.java), [CommonController](sky-server/src/main/java/com/sky/controller/admin/CommonController.java) |
| Redis and caching | Configured Redis key/value serialization, stored shop opening status in Redis, cached customer dish queries, and added Spring Cache annotations for set-meal data. Added cache updates or eviction around menu changes. | [RedisConfiguration](sky-server/src/main/java/com/sky/config/RedisConfiguration.java), [customer controllers](sky-server/src/main/java/com/sky/controller/user), [admin controllers](sky-server/src/main/java/com/sky/controller/admin) |
| Customer ordering | Implemented shopping-cart operations, address-book management, order submission, order history and details, cancellation, and ordering again from an existing order. Order submission writes the order and its line items and clears the cart within a transaction. | [ShoppingCartServiceImpl](sky-server/src/main/java/com/sky/service/impl/ShoppingCartServiceImpl.java), [AddressBookServiceImpl](sky-server/src/main/java/com/sky/service/impl/AddressBookServiceImpl.java), [OrderServiceImpl](sky-server/src/main/java/com/sky/service/impl/OrderServiceImpl.java) |
| Restaurant orders | Added order search, confirmation, rejection, cancellation, delivery, and completion operations. Implemented WebSocket notifications for new orders and customer reminders. | [admin OrderController](sky-server/src/main/java/com/sky/controller/admin/OrderController.java), [OrderServiceImpl](sky-server/src/main/java/com/sky/service/impl/OrderServiceImpl.java), [WebSocketServer](sky-server/src/main/java/com/sky/webSocketServer/WebSocketServer.java) |
| Scheduled processing | Added a job that cancels unpaid orders older than 15 minutes and a nightly job that marks eligible delivery orders as completed. | [OrderTask](sky-server/src/main/java/com/sky/task/OrderTask.java) |
| Reports and dashboard APIs | Built date-range reports for turnover, new and total users, order counts and completion rate, and top-selling products. Added workspace overview APIs and a template-based Excel export for the previous 30 days using Apache POI. | [ReportServiceImpl](sky-server/src/main/java/com/sky/service/impl/ReportServiceImpl.java), [WorkSpaceServiceImpl](sky-server/src/main/java/com/sky/service/impl/WorkSpaceServiceImpl.java) |

## Technical focus

This project gave me practice connecting REST endpoints to service logic and MyBatis persistence across a larger business domain. The ordering workflow combines validation, related database writes, cart cleanup, order status changes, and notifications. Reporting adds a separate view of that same data through aggregation queries and Excel output.

I also worked on reducing repeated code through AOP-based audit filling, shared DTOs and response objects, reusable exception handling, and common authentication infrastructure.

## Stack

Java, Spring Boot 2.7.3, Spring MVC, MyBatis, PageHelper, MySQL, Redis, Spring Cache, JWT, Spring AOP, WebSocket, Spring scheduling, Apache POI, Knife4j/Swagger, Aliyun OSS, and Maven.

## Repository structure

```text
sky-common/       Shared utilities, constants, exceptions, and configuration properties
sky-pojo/         Entities, request DTOs, and response VOs
sky-server/
  src/main/java/com/sky/
    controller/   Admin, customer, and payment-notification endpoints
    service/      Business logic
    mapper/       Persistence interfaces
    aspect/       Audit-field filling
    task/         Scheduled order processing
    webSocketServer/  Admin notifications
  src/main/resources/
    mapper/       MyBatis XML mappings
    businessReportTemplate.xlsx
```

## Local setup requirements

The checked-in repository contains the backend source, MyBatis mappings, and an Excel report template. It does **not** include an application YAML/properties file or a database initialization script, so a fresh clone needs local configuration and a matching database schema before it can run.

Provide the following settings through your local Spring configuration:

| Configuration | Purpose |
| --- | --- |
| `spring.datasource.*` | MySQL connection and credentials |
| `spring.redis.*` | Redis connection |
| `mybatis.mapper-locations` | Load the XML mappings under `classpath:mapper/*.xml` |
| `mybatis.configuration.map-underscore-to-camel-case` | Map database names such as `order_time` to Java fields |
| `sky.jwt.*` | Separate admin/customer signing keys, TTLs, and token header names |
| `sky.wechat.*` | WeChat application settings and payment credentials where those integrations are used |
| `sky.alioss.*` | OSS endpoint, bucket, and upload credentials |

For Redis-backed Spring Cache behavior, configure the cache provider accordingly. The exact JWT, WeChat, and OSS property fields are defined in [sky-common properties](sky-common/src/main/java/com/sky/properties).

After preparing the database and configuration, build from the repository root and run the server module:

```sh
mvn install -DskipTests
mvn -pl sky-server spring-boot:run
```

Knife4j API documentation is mapped to `/doc.html` on your configured server port.

## Current scope

- This repository contains backend APIs; the restaurant UI and customer mini-program are separate.
- The current order-submission path simulates successful payment by marking orders as paid immediately. Payment utilities and callback code are present, but this is not a verified live payment integration.
- Excel export currently reads the template from a machine-specific absolute path in `ReportServiceImpl`; update that path to your local template before using export.
