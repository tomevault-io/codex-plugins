# GitHub Copilot Instructions for 2BA OpenAPI Specification

## Copilot Behavior Instructions

When working with this repository:

1. **Always follow the established naming conventions**:
   
   **File Naming (for organization)**:
   - **PascalCase** for schema files: `ErrorResponse.yaml`, `Price.yaml`, `NetPriceResponse.yaml`
   - **kebab-case** for parameter/response files: `page-number.yaml`, `400-bad-request.yaml`
   
   **Component Names (for $ref references)**:
   - **PascalCase** for ALL component references: `Price`, `PageNumber`, `BadRequest`, `NotFound`
   - Component keys must match regex: `^[a-zA-Z0-9\.\-_]+$`
   - References always use component names, not filenames
   
   **Schema Properties & Parameters**:
   - **camelCase** for properties: `netPrice`, `errorCode`, `pageNumber`
   - **camelCase** for parameter names: `supplierGln`, `tradeItemId`, `quantity`
   - **camelCase** for operationIds: `getTradeItemNetPrice`, `getProducts`
   
   **API Paths**:
   - **kebab-case** for path segments: `/netprices`, `/trade-items`
   
  **Response Envelope Pattern (Single-Item and Bulk)**:
  - All responses wrap content in a `data` property — never return domain objects directly
  - The `data` property **must always use a named `$ref`** — NEVER define `data` as an inline anonymous `type: object`
  - Inline anonymous objects cause code generators (NSwag) to produce ambiguous types (`Data`, `Data2`, `Data3`)
  - For single-item responses: `data: { $ref: ./XxxResponseData.yaml }` where the `*ResponseData` schema contains composite key + domain data
  - For bulk responses: `data: { type: array, items: { $ref: ... } }` with `meta: { $ref: CursorPaginationMetadata }`
  - Naming: `{Entity}{Aspect}ResponseData.yaml` (e.g., `ProductDetailsResponseData`, `TradeItemPricingsResponseData`)
  - `*ResponseData` files go in `schemas/responses/` and must be registered in `openapi.yaml` under `components/schemas`
  - Use `meta` for cursor pagination information (reuse `CursorPaginationMetadata` with `cursor`, `prevCursor`, `hasNext`, `hasPrev`, `limit`, `estimatedTotal`)

2. **OpenAPI 3.1 / JSON Schema 2020-12 Requirements**:
   - Optional ETIM scalars use a non-null type and are omitted from `required`; absent means no value
   - Optional ETIM objects use a direct `$ref` and are omitted from `required`
   - Use JSON Schema null types only where the API intentionally gives `null` distinct semantics (NOT deprecated `nullable: true`)
   - Use `type: array` (required, never nullable) for collection properties in sub-resource response schemas — empty collections use `[]`, never `null`
   - **Exception**: Aggregate root response schemas (`ProductResponseData`, `TradeItemResponseData`) use `type: ["array", "null"]` for collection properties to support partial inclusion (`null` = not included in this response, `[]` = included but empty)
   - **Exception**: Aggregate root response schemas also use `anyOf: [$ref, type: "null"]` + optional (not in `required`) for singular object properties (`details`, `lcaEnvironmental`, `ordering`) to support three-state field selection (absent = not requested, `null` = requested but no data, object = has data)
   - Use `anyOf`, `oneOf`, `allOf` for composition (avoid deprecated patterns)
  - Allow `additionalProperties` throughout the object model, including nested models, to preserve backward compatibility when optional fields are added
   - Use `examples` array (plural) in schemas, not `example` (singular, deprecated)
   - Include `format` for type hints: `uri`, `email`, `date-time`, `uuid`, etc.
   - Use `format: decimal` on all ETIM-converted number fields (code-gen hint for NSwag/.NET `decimal`; see [design decisions](../docs/product-data-openapi-design-decisions.md#format-decimal--code-generator-hint))
   - Use `minLength`, `maxLength`, `minimum`, `maximum` for validation
   - Prefer `const` over single-value `enum` for literal values

3. **Maintain the directory structure** - Place schemas in appropriate domain folders
4. **Use $ref extensively** - Don't repeat schema definitions; create reusable components
5. **Create separate files** for reusable enumerations and shared types
6. **Follow the DTO pattern** - Keep response schemas separate from domain schemas
7. **Include comprehensive examples** - Every schema should have realistic examples
8. **Document business context** - Add descriptions explaining purpose and constraints
9. **Support flexible identifiers** - Use `anyOf`/`oneOf` patterns for GLN/DUNS/alternative IDs
10. **Maintain consistent error responses** - Use RFC 9457 Problem Details format (`application/problem+json`). Clients branch on `type` and `status`, never on `detail` or `title` strings. No exception-message matching.

11. **Security scheme convention**:
   - All APIs use **OAuth 2.0 Client Credentials** (`clientCredentials` flow)
   - Token URL: `https://identity.2ba.nl/connect/token` (production)
   - Scope naming: `read:{resource}` (e.g., `read:products`, `read:tradeitems`, `read:netprices`, `read:stock`)
   - Support **RFC 7523 client assertion** (`client_assertion` + `client_assertion_type`) as an alternative to `client_secret`
   - Do NOT use `bearerAuth` (HTTP Bearer) or `apiKeyAuth` — all auth goes through OAuth2

12. **Tag and grouping conventions**:
   - Each API defines exactly 2 tags: `{Resource} single` and `{Resource} bulk`
   - Use `x-tagGroups` with a single group named after the resource (e.g., `Products`, `Trade Items`)
   - Every endpoint must have exactly one tag assigned

13. **Component registration**:
   - ALL shared parameters, schemas, and responses used by an API MUST be registered in that API's `components/` section
   - This includes shared parameters like `Language`, `Cursor`, `Limit` — not just `ResponseData` schemas
   - Component names use **PascalCase** regardless of the file's kebab-case name

14. **YAML `$ref` style**:
   - Use **unquoted** file-path `$ref` values: `$ref: ../../shared/schemas/common/ProblemDetails.yaml`
   - Single quotes are only needed for internal refs starting with `#`: `$ref: '#/components/schemas/Pet'`
   - Per OpenAPI 3.1.0 spec examples and YAML 1.2.2 plain scalar convention

15. **Generated bundles**:
   - Each API has a `generated/` folder with a bundled `{api}-api.yaml` file (git-tracked)
   - Regenerate after ANY source spec change: `npx @redocly/cli bundle --config openapi/redocly.yaml {api}@v1 -o openapi/apis/{api}/generated/{api}-api.yaml`
   - Always commit regenerated bundles alongside the source changes that triggered them

16. **Mermaid class diagram conventions**:
   - Do **not** use UML visibility prefixes (`+`, `-`, `#`, `~`) — all API properties are public by definition
   - Append `✱` (U+2731 Heavy Asterisk) to property names that are **required** (e.g., `number amount✱`)
   - Do **not** use ASCII `*` — Mermaid interprets it as the UML abstract classifier and strips it
   - Leave optional properties unmarked (e.g., `string description`)
   - Add a legend below the diagram: `> **Legend**: Properties marked with \`✱\` are required.`

17. **Domain model specifications** (`openapi-domain.yaml`):
   - Product and TradeItem APIs each have a separate `openapi-domain.yaml` for domain model documentation
   - Contains a single dummy GET endpoint referencing the aggregate root domain schema (`Product.yaml` / `TradeItem.yaml`)
   - Registered in `redocly.yaml` as `{api}-domain@v1` (e.g., `product-domain@v1`, `tradeitem-domain@v1`)
   - Used by `scripts/generate-model-docs.mjs` to generate Mermaid class diagrams and interactive treeview HTML
   - Must be bundled (`npm run bundle`) before running `npm run build:models`
   - Domain specs are documentation-only — the REST API specs (`openapi.yaml`) remain the source of truth for actual API contracts

18. **Domain vs Response schema relationship**:
   - Domain schemas (`/schemas/domain/`) are the SOURCE definitions for business entities
   - Response schemas (`/schemas/responses/`) `$ref` into domain schemas — they are NOT duplicates
   - Domain schemas define the entity structure; response schemas add the API envelope (`data`, `meta`) and composite keys

19. **Response conventions (404, nullability, errors)**:
   - **404 only from root endpoints**: `GET /products/{gln}/{num}` and `GET /trade-items/{gln}/{num}` return 404 when the entity doesn't exist. Sub-resource endpoints (`/descriptions`, `/attachments`, `/pricings`, etc.) **never** return 404 — they return 200 with an empty collection. Sub-resource 200 does not confirm parent entity existence.
   - **Sub-resource collections always `[]`**: In sub-resource response schemas, array properties use `type: array` (required), never `type: ["array", "null"]`. Empty = `[]`, never `null`. Code-gen: `required List<T>` with default `[]`.
   - **Aggregate root collections support partial inclusion**: In `ProductResponseData` and `TradeItemResponseData`, collection properties use `type: ["array", "null"]` (required + nullable) to support three-state semantics:
     - `[...]` — requested, has data
     - `[]` — requested, but empty (no data exists)
     - `null` — not included in this response (not requested)
     Code-gen: `required ICollection<T>?` (nullable collection).
   - **Optional objects are absent**: Singular optional ETIM objects use a direct `$ref` and are omitted when unavailable. Required objects remain in `required`. Use nullable objects only for explicitly documented API-specific three-state semantics.
   - **Required-nullable singular sub-resource `data` (exactly four)**: `ProductDetailsResponseData.details`, `ProductLcaEnvironmentalResponseData.lcaEnvironmental`, `TradeItemDetailsResponseData.details`, and `TradeItemOrderingResponseData.ordering` are required and nullable (`anyOf: [$ref, type: "null"]`). `null` signals the singular resource is unavailable (parent entity not found, or no data of that kind); the endpoint returns `200`, never `404`.
   - **Optional-nullable singular objects in aggregate root (four total)**: `ProductResponseData.details`, `ProductResponseData.lcaEnvironmental`, `TradeItemResponseData.details`, and `TradeItemResponseData.ordering` are optional (not in `required`) and nullable (`anyOf: [$ref, type: "null"]`). Three-state semantics: object = has data, `null` = requested but no data exists, absent = not included in this response (not requested). Code-gen: `T?` (nullable, optional).
   - **Intentional `null` allowlist**: JSON `null` is permitted **only** in (1) aggregate response arrays in `ProductResponseData`/`TradeItemResponseData`, (2) the four required-nullable singular sub-resource `data` properties above, (3) the four optional-nullable singular objects in aggregate root responses above, and (4) pagination metadata members `cursor`/`prevCursor`/`estimatedTotal` in `CursorPaginationMetadata`. Every other optional value is omitted (Option B), enforced by `scripts/validate-option-b.mjs`.
   - **Flattened pricing conditional absence**: In `TradeItemPricingSummary`, rows without an allowance/surcharge omit all seven `allowanceSurcharge*` fields (never `null`). `dependentRequired` couples `allowanceSurchargeIndicator` and `allowanceSurchargeType` and requires both whenever any other allowance/surcharge field is present.
   - **Errors RFC 9457 ProblemDetails**: All errors use Problem Details with `application/problem+json`. Branch on `type`/`status`, never on `detail`/`title`. Validation errors include `errors` extension member.

## Naming Convention Details

### Schema Components (PascalCase)
- File: `ErrorResponse.yaml` → Component: `#/components/schemas/ErrorResponse`
- File: `Price.yaml` → Component: `#/components/schemas/Price`
- File: `NetPriceResponse.yaml` → Component: `#/components/schemas/NetPriceResponse`

### Parameter Components (PascalCase)
- File: `cursor.yaml` → Component: `#/components/parameters/Cursor`
- File: `limit.yaml` → Component: `#/components/parameters/Limit`
- File: `sort-order.yaml` → Component: `#/components/parameters/SortOrder`

### Response Components (PascalCase)
- File: `400-bad-request.yaml` → Component: `#/components/responses/BadRequest`
- File: `404-not-found.yaml` → Component: `#/components/responses/NotFound`
- File: `500-internal-server-error.yaml` → Component: `#/components/responses/InternalServerError`

## JSON Schema Best Practices

### Nullable Fields (OpenAPI 3.1 / JSON Schema 2020-12)
```yaml
# ✅ Optional ETIM scalar — absent when unavailable
propertyName:
  type: string

# ❌ INCORRECT — do not model ordinary optionality as null
propertyName:
  type: ["string", "null"]

# ✅ Sub-resource arrays — required, never nullable (empty = [])
descriptions:
  type: array
  items:
    $ref: '#/components/schemas/ProductDescription'

# ✅ Aggregate root arrays — required, nullable (null = not included)
# Only in ProductResponseData.yaml and TradeItemResponseData.yaml
descriptions:
  type: ["array", "null"]
  items:
    $ref: '#/components/schemas/ProductDescription'

# ❌ INCORRECT — Do NOT use nullable arrays in sub-resource schemas
descriptions:
  type: ["array", "null"]
  items:
    $ref: '#/components/schemas/ProductDescription'

# ✅ Optional objects — direct reference, not in required (domain schemas)
ordering:
  $ref: '#/components/schemas/TradeItemOrdering'

# ✅ Aggregate root singular objects — optional + nullable (three-state field selection)
# Only in ProductResponseData.yaml and TradeItemResponseData.yaml
details:
  anyOf:
    - $ref: '#/components/schemas/ProductDetails'
    - type: "null"
```

### Examples (OpenAPI 3.1+)

**For Schemas** - Use `examples` array (plural):
```yaml
# ✅ CORRECT - Schema examples array (plural)
type: object
properties:
  name:
    type: string
examples:
  - name: "Example 1"
  - name: "Example 2"

# ❌ DEPRECATED - Avoid example (singular) for schemas
example:
  name: "Example"
```

**For Parameters** - Use `example` (singular) OR `examples` (plural object):
```yaml
# ✅ CORRECT - Parameter example (simple, singular)
name: cursor
in: query
schema:
  type: string
example: "eyJpZCI6MTIzfQ=="

# ✅ ALSO CORRECT - Parameter examples (named examples object)
name: status
in: query
schema:
  type: string
examples:
  active:
    value: "active"
    summary: Active status
  pending:
    value: "pending"
    summary: Pending status
```

### Type Validation
```yaml
# ✅ CORRECT - Include validation constraints
productId:
  type: string
  minLength: 1
  maxLength: 35
  pattern: "^[A-Z0-9-]+$"
  examples:
    - "PROD-12345"

# ETIM-converted number field: use format: decimal and multipleOf: 0.0001
price:
  type: number
  format: decimal
  minimum: 0
  multipleOf: 0.0001
  maximum: 99999999999.9999
  examples:
    - 19.99
```

### Extensible Object Models
```yaml
# ✅ CORRECT - Keep objects open for additive evolution
type: object
required:
  - id
  - name
properties:
  id:
    type: string
  name:
    type: string

# Clients must accept and ignore unknown properties.
```

### Response Envelope — Named `$ref` for `data` (Code Generation)

The `data` property in response envelopes **must** use a named `$ref` — never an inline anonymous object. This is critical for code generators like NSwag.

```yaml
# ❌ INCORRECT — inline object → NSwag generates "Data", "Data2", "Data3"
type: object
properties:
  data:
    type: object
    properties:
      manufacturerIdGln: ...
      details:
        $ref: ../domain/ProductDetails.yaml

# ✅ CORRECT — named $ref → NSwag generates "ProductDetailsResponseData"
type: object
properties:
  data:
    $ref: ./ProductDetailsResponseData.yaml
```

**Single-item response** — `data` references a `*ResponseData` schema:
```yaml
# ProductDetailsResponse.yaml
type: object
required:
  - data
properties:
  data:
    $ref: ./ProductDetailsResponseData.yaml
```

**Bulk response** — `data` is an array with named `items`, plus `meta`:
```yaml
# BulkProductDetailsResponse.yaml
type: object
required:
  - data
  - meta
properties:
  data:
    type: array
    items:
      $ref: ../domain/ProductDetailsSummary.yaml
  meta:
    $ref: ../../../../shared/schemas/common/CursorPaginationMetadata.yaml
```

Always prioritize clarity, reusability, and adherence to OpenAPI 3.1+ and JSON Schema 2020-12 standards while respecting the established project patterns and business domain requirements.

---
> Source: [2BABV/product-data-openapi](https://github.com/2BABV/product-data-openapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-21 -->
