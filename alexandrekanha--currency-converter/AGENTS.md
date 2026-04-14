# Copilot Instructions: Currency Converter API

## Project Overview

A RESTful currency conversion API (Java 17 + Spring Boot 4.0.2) with a React/Vite frontend. The backend consumes external exchange rates (open.er-api.com), implements smart two-tier caching for resilience, and provides detailed OpenAPI/Swagger documentation.

Stack: **Java 17 | Spring Boot 4.0.2 | Maven | React 19 | Vite | Railway Deployment**

---

## Architecture & Critical Patterns

### Two-Tier Caching Strategy (Core Feature)
The service uses a resilience pattern with two cache TTL levels:
- **Fresh cache (300s)**: Use cached rates without API calls
- **Fallback cache (3600s)**: If API fails, serve stale data up to 1 hour old
- **Beyond 1 hour**: Return error only if no cached value exists

**Reference**: [CurrencyConverterService.java](backend/src/main/java/com/alexandre/currency_converter/service/CurrencyConverterService.java#L42-L65)
- Cache key: `"USD-BRL"` (from-to)
- `ConcurrentHashMap<String, ExchangeRateCache>` for thread-safety
- Timestamp stored in `ExchangeRateCache` objects

### External API Integration
- **Provider**: `https://open.er-api.com/v6/latest/{currency}`
- **Response parsing**: Jackson `ObjectMapper` to extract rates from JSON
- **Error handling**: Catches API failures gracefully, logs extensively
- Custom exception: `ExternalApiException` for API-specific errors

### REST Endpoint Design
Single conversion endpoint: `GET /api/convert?from=USD&to=BRL&amount=100`
- Query parameters (not path variables)
- `BigDecimal` for amounts to ensure precision
- Validates: amount must be > 0 → throws `InvalidAmountException` (400 BAD_REQUEST)

### Data Precision
- Monetary values use `BigDecimal` (never `double` for money)
- Decimal scale: **4 places** (`MONETARY_SCALE = 4`)
- Applied in `ConvertController.java` using `setScale(MONETARY_SCALE, RoundingMode.HALF_UP)`

---

## Key Files & Responsibilities

| File | Purpose |
|------|---------|
| [CurrencyConverterService.java](backend/src/main/java/com/alexandre/currency_converter/service/CurrencyConverterService.java) | Core conversion logic, caching, external API calls |
| [ConvertController.java](backend/src/main/java/com/alexandre/currency_converter/controller/ConvertController.java) | HTTP endpoint, validation, response formatting |
| [GlobalExceptionHandler.java](backend/src/main/java/com/alexandre/currency_converter/exception/GlobalExceptionHandler.java) | Centralized exception handling → JSON ErrorResponse |
| [CorsConfig.java](backend/src/main/java/com/alexandre/currency_converter/config/CorsConfig.java) | CORS enabled for all origins (dev/test only) |
| [ExchangeRateCache.java](backend/src/main/java/com/alexandre/currency_converter/service/ExchangeRateCache.java) | Simple immutable cache entry with timestamp |
| [api.js](frontend/currency-converter-ui/src/api.js) | Frontend fetch wrapper for `/api/convert` |

---

## Common Developer Tasks

### Build & Run Backend
```bash
cd backend
mvn clean package          # Full build
mvn spring-boot:run        # Run locally (port 8080)
mvn test                   # Run all tests
```

### Build & Run Frontend
```bash
cd frontend/currency-converter-ui
npm install                # Dependencies (once)
npm run dev                # Dev server (Vite, HMR)
npm run build              # Production build
npm run lint               # ESLint check
```

### Testing Strategy
Prefer **unit tests with mock API** over integration tests to avoid real external calls:
- Use `Mockito.spy()` to mock `fetchRateFromExternalApi()`
- Verify cache hits with `verify(service, times(1)).fetchRateFromExternalApi()`
- Test performance: second call should be faster (cache hit)
- Use `@DisplayName` for test documentation

**Reference**: [CurrencyConverterServiceTest.java](backend/src/test/java/com/alexandre/currency_converter/service/CurrencyConverterServiceTest.java)

---

## Code Conventions

### Backend (Java)

1. **Exception Handling**
   - Custom exceptions inherit from `RuntimeException` (unchecked)
   - `@RestControllerAdvice` handles all domain exceptions globally
   - Return `ErrorResponse` DTO with `HttpStatus` codes

2. **Logging**
   - Use `LoggerFactory.getLogger(ClassName.class)` per class
   - Log cache hits/misses, API calls, fallback decisions
   - Example: `log.info("Cache HIT (fresh)! ... for {} (age: {}s)", key, cacheAge)`

3. **DTOs**
   - Immutable (final fields) or use explicit getters
   - No business logic
   - Use for request/response serialization
   - Include all fields needed for API contract

4. **Protected Methods for Testing**
   - `fetchRateFromExternalApi()` is `protected` → allows mocking in tests via `spy()`

### Frontend (JavaScript/React)

1. **API Client**
   - Place fetch logic in `api.js` (centralized base URL: `localhost:8080/api`)
   - Export functions, not classes
   - Handle errors: throw on non-2xx responses

2. **Configuration**
   - Vite config: `vite.config.js`
   - ESLint: `eslint.config.js`
   - Environment: Set `BASE_URL` for production in `api.js`

---

## Deployment Notes

- **Platform**: Railway.app
- **Java version**: Explicitly set to 17 in `pom.xml`
- **Port**: Environment variable `PORT` (defaults to 8080)
- **CORS**: Configured to allow all origins → should tighten for production
- **Forward headers**: `server.forward-headers-strategy=framework` for HTTPS behind proxy

---

## Common Mistakes to Avoid

1. ❌ Use `double` for monetary values → ✅ Use `BigDecimal`
2. ❌ Assume cache is always valid → ✅ Check TTL age, fall back gracefully
3. ❌ Lose precision in decimal places → ✅ Use `.setScale(4, RoundingMode.HALF_UP)`
4. ❌ Log sensitive API keys/tokens → ✅ Log only sanitized request/response info
5. ❌ Test with real external API → ✅ Mock `fetchRateFromExternalApi()` in unit tests

---

## Performance Considerations

- Cache reduces API calls by ~95% in typical usage (5min fresh TTL)
- `ConcurrentHashMap` allows concurrent reads without blocking
- `BigDecimal` operations are slower than `double` but necessary for accuracy
- Frontend fetch is synchronous—streaming/pagination not currently needed for single conversions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlexandreKanha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
