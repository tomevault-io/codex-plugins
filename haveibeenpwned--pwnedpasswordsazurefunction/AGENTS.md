# Copilot Instructions

## Build and test commands

- Build the whole repository from the root with `dotnet build --configuration Release --nologo`.
- Run the full test suite from the root with `dotnet test --configuration Release --nologo`.
- Run a single xUnit test with `dotnet test Functions.Tests\HaveIBeenPwned.PwnedPasswords.Tests.csproj --configuration Release --filter "FullyQualifiedName~HaveIBeenPwned.PwnedPasswords.Tests.RangeTests.Returns_ok_given_valid_hashprefix" --nologo`.
- CI uses `dotnet test --no-build --configuration Release --verbosity normal --logger "trx;LogFileName=test-results.trx" --collect:"XPlat Code Coverage"` after a successful build.
- There is no separate lint command in the repository; follow `.editorconfig` and let the normal build/test flow catch issues.

## High-level architecture

- This solution has four main parts: the Azure Functions app in `Functions`, the shared hash/parsing library in `Shared\HaveIBeenPwned.PwnedPasswords.Shared`, the xUnit test project in `Functions.Tests`, and several one-off data tools in `Tools`.
- The public API is the anonymous `GET /range/{hashPrefix}` function in `Functions\Functions\Range.cs`. It validates a 5-character hex prefix, defaults to SHA-1, switches to NTLM via `?mode=ntlm`, loads `<PREFIX>.txt` from Azure Blob Storage through `IFileStorage`, and streams plaintext results with optional Brotli/gzip/deflate compression.
- Ingestion is a two-step workflow, not a direct blob update. `Functions\Functions\Ingestion\Submit.cs` validates the JSON payload, creates an unconfirmed transaction in table storage, and stores the raw ingestion file in blob storage. `Functions\Functions\Ingestion\Confirm.cs` confirms that transaction and queues it for background processing.
- Background ingestion fans out in two queue-triggered stages. `ProcessTransaction` reads the stored JSON, uppercases both hash types, groups entries by the first 5 hex characters, and pushes batches of up to 500 grouped entries. `ProcessPwnedPasswordEntryBatch` reloads the current prefix blob, merges prevalence counts, retries optimistic-concurrency conflicts using blob ETags, and marks modified prefixes in table storage.
- `Functions\Functions\Ingestion\PurgeCloudflareCache.cs` is a daily timer job. It reads yesterday's modified prefixes from table storage and purges the matching `/range/{prefix}` URLs from Cloudflare in batches of 30.
- `Functions\Program.cs` is the composition root. Azure Blob, Queue, and Table clients plus the Cloudflare client are configured there from app settings such as `PwnedPasswordsConnectionString`, `BlobContainerName`, `TableNamespace`, `PwnedPasswordsBaseUrl`, `CloudflareAPIToken`, and `CloudflareZoneIdentifier`.

## Key conventions

- Keep hash text uppercase once it enters the system. The 5-character uppercase prefix is the primary partitioning key everywhere: HTTP routes, blob filenames, queue batches, and Azure Table storage lookups.
- Reuse the abstractions in `Functions\Abstractions` (`IFileStorage`, `ITableStorage`, `IQueueStorage`, `ICdnStorage`) instead of calling Azure SDK clients directly from function handlers. Storage and CDN details belong in `Functions\Implementations`.
- Prefix files must remain lexicographically sorted and use the text format `SUFFIX:COUNT` with no trailing newline on the final record. `BlobStorage.RenderHashes` and `Functions.Tests\BlobStorageTests.cs` define that contract.
- Prefer the shared low-allocation helpers in `Shared\HaveIBeenPwned.PwnedPasswords.Shared\Hash.cs` and `HashEntry.cs` for parsing, validation, hashing, and serialization. They already encode the repository's SHA-1/NTLM rules, binary/text file formats, pipe readers, and pooled-buffer behavior.
- HTTP functions return plain-text errors through `Functions\HttpRequestDataExtensions.cs` rather than constructing ad-hoc response payloads.
- The append pipeline is intentionally confirm-before-process. Changes to ingestion behavior should preserve the transaction record + stored ingestion file + queue-trigger flow instead of writing blobs directly from the HTTP endpoint.

---
> Source: [HaveIBeenPwned/PwnedPasswordsAzureFunction](https://github.com/HaveIBeenPwned/PwnedPasswordsAzureFunction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-10 -->
