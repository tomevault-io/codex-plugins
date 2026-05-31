# DDNS Server - AI Coding Instructions

## Project Overview
This is a secure Dynamic DNS server that interfaces with Virtualmin web panels through session-based authentication. The architecture has evolved from simple API proxy to a complete DNS management system with encryption, logging, and CLI support.

## Architecture & Data Flow
- **Setup Phase**: `setup.php` encrypts Virtualmin credentials using generated API key as encryption password
- **Web API**: `ddns.php` handles HTTP requests, decrypts credentials, updates DNS via Virtualmin sessions
- **CLI Interface**: `cli.php` provides command-line access for automated updates
- **Core Logic**: `virtualmin.php` manages Virtualmin web sessions and DNS record operations

## Key File Patterns

### Configuration Management
- Configuration stored in `.config` with encrypted credentials (5 lines: encrypted_username, encrypted_password, endpoint, logage, nameserver)
- API key serves dual purpose: authentication token and encryption password
- Setup validates all inputs with comprehensive error handling and HTML escaping

### Security Model
- **Encryption**: AES-256-CBC with PBKDF2 key derivation (1000 iterations, SHA256)
- **Session Management**: Cookie-based Virtualmin login via `/session_login.cgi`
- **DNS Validation**: Regex pattern for subdomain format validation
- **Record Verification**: Uses `gethostbyname()` to check existing records before updates

### Virtualmin Integration Patterns
- **Session Flow**: Login → Get cookie → POST to `/save_record.cgi` → Cleanup cookie
- **Record Format**: `dom={domain_id}&id={name}.%2F{type}%2F{old_value}` (URL-encoded)
- **Success Detection**: HTTP 302 redirect indicates successful update
- **Error Handling**: Non-302 responses treated as failures with debug info

### Logging Architecture
- **Daily Log Files**: `logs/YYYY-MM-DD.log` with automatic rotation
- **Log Cleanup**: Automatic deletion based on `logage` configuration
- **Request Tracking**: IP address, timestamp, and full request URI logging
- **Error Context**: Detailed error messages with operation context

## Development Workflow

### Local Testing
- Use `cli.php` for direct command-line testing: `php cli.php <domain_id> <type> <name> <value> <username> <password> [endpoint] [nameserver]`
- Check `logs/` directory for detailed operation logs
- Verify encryption/decryption with test API keys

### Configuration Debugging
- Manual credential testing via `encrypt.php` class methods
- Session debugging by examining `.webminsesh` cookie file (auto-cleaned)
- DNS verification through `gethostbyname()` fallback when dig unavailable

## Critical Dependencies
- **PHP OpenSSL**: Required for AES-256-CBC encryption and PBKDF2
- **cURL**: Session management and HTTP POST operations
- **File Permissions**: Write access for `.config`, `.webminsesh`, and `logs/` directory
- **Virtualmin Panel**: Must support `/session_login.cgi` and `/save_record.cgi` endpoints

## Project-Specific Conventions
- **Domain ID**: Virtualmin's internal domain identifier (not domain name)
- **Record Names**: Must include subdomain + domain (e.g., "test.example.com")
- **TTL Handling**: Uses Virtualmin default TTL (`ttl_def=1`)
- **Cookie Management**: Temporary `.webminsesh` file, always cleaned up
- **Error Responses**: JSON format with `status_code`, `result`, and `error` fields

## Common Modifications
- Update encryption parameters in `Encryption` class constructor
- Modify DNS validation regex in `updateRecord()` for different domain patterns
- Adjust log retention in setup form or `.config` file
- Change session endpoint URLs for different Virtualmin versions
- Add nameserver validation for DNS checking

---
> Source: [silasmontgomery/ddns-server-for-virtualmin](https://github.com/silasmontgomery/ddns-server-for-virtualmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-31 -->
