# Java

The layout and names of [SKILL.md](SKILL.md), spelled for Java: folders become packages, and the role suffix lives in the class name.

```
src/main/java/com/acme/shop/
├── ShopApplication.java                  ← composition root (Spring Boot entry)
├── shared/
└── ordering/                             ← module
    ├── domain/
    │   ├── entity/           Order
    │   ├── valueobject/      Money
    │   ├── port/             OrderRepositoryPort, PaymentGatewayPort
    │   ├── event/            OrderPlacedEvent
    │   └── error/            InsufficientStockError
    ├── application/
    │   ├── usecase/          PlaceOrderUseCase
    │   ├── service/          PricingService
    │   └── dto/              PlaceOrderCommand, OrderResult
    └── infrastructure/
        ├── driving/          OrderController, OrderEventListener
        │   └── dto/          PlaceOrderRequest, OrderResponse, OrderWebMapper
        ├── driven/           JpaOrderRepositoryAdapter, StripePaymentGatewayAdapter
        │   └── dto/          JpaOrderModel, StripeChargePayload, OrderJpaMapper
        └── config/           OrderingConfig
```

- **Packages are lowercase and singular, with no separators**: `value-objects/` becomes `valueobject`, `use-cases/` becomes `usecase`. The full name is `<base package>.<module>.<layer>.<folder>`, such as `com.acme.shop.ordering.application.usecase`.
- **Errors extend `RuntimeException`** and keep the `Error` suffix, so the domain's name for the failure survives.
- **DTOs are `record`s.** The JPA `@Entity` is a driven DTO (`JpaOrderModel`), never the domain's `Order`: the domain imports no framework.
- **Spring stereotypes are welcome outside the domain**: `@Service` on use cases and services, `@RestController` and `@Component` on adapters, `@Configuration` in `config/`.
- **A module's interface** is its `application.usecase` and `application.dto` packages; another module imports nothing else from it.
- **Tests mirror the main tree** under `src/test/java`, named `<Class>Test`, since Maven and Gradle keep them apart.

## Enforcing

ArchUnit, as a JUnit test the suite runs: a `layeredArchitecture()` per module (domain accessed by application and infrastructure, application by infrastructure only), plus a rule that `..domain..` classes depend on no `org.springframework..` or `jakarta.persistence..` class. Spring Modulith's `ApplicationModules.verify()` adds the cross-module rule when the app uses it.
