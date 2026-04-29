For CRUD, like authentication, uses Wazuh API endpoints and follow the API documentation for request and response formats. Ensure that all API interactions are properly authenticated and handle errors gracefully. Use appropriate HTTP methods (GET, POST, PUT, DELETE) for each operation and validate input data before making API calls. But for data, use Wazuh OpenSearch endpoints and follow the OpenSearch documentation for request and response formats. Ensure that all OpenSearch interactions are properly authenticated and handle errors gracefully. Use appropriate HTTP methods (GET, POST, PUT, DELETE) for each operation and validate input data before making OpenSearch calls.

Any exception regarding of Wazuh should be logged and handled properly, providing meaningful error messages to the user. Additionally, ensure that all API interactions are secure and follow best practices for authentication and data handling.

---
> Source: [OcatZ0/oneforall](https://github.com/OcatZ0/oneforall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
