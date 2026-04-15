# Copilot Instructions for R2 Storage Experiment

## Project Overview

This is a Python experiment project for Cloudflare R2 object storage operations using the boto3 S3-compatible API. The codebase is a single-file utility script demonstrating core R2 operations: upload, download, list, and presigned URL generation.

## Tech Stack

- **Python 3.10+** (see `.python-version`)
- **boto3**: S3-compatible API for R2 operations
- **python-dotenv**: Environment variable management
- **uv**: Package manager (lock file present)

## Architecture

Single entry point: [main.py](main.py)

- `get_r2_client()`: Initializes boto3 client with Cloudflare R2 endpoint (`https://{ACCOUNT_ID}.r2.cloudflarestorage.com`)
- Core operations as standalone functions: `upload_file()`, `download_file()`, `list_files()`, `generate_presigned_url()`
- `if __name__ == "__main__"`: Demo workflow that creates test file, uploads, lists, generates URL, downloads, and cleans up

## Configuration Pattern

Environment variables via `.env` (gitignored):

```python
R2_ACCOUNT_ID=<your-cloudflare-account-id>
R2_ACCESS_KEY_ID=<your-r2-access-key>
R2_SECRET_ACCESS_KEY=<your-r2-secret-key>
R2_BUCKET_NAME=<your-bucket-name>
```

Copy `.env.example` to `.env` and fill with real credentials.

## Key Conventions

- **Indonesian comments/print messages**: User preference, keep error messages and comments in Indonesian
- **Error handling**: Use `try/except ClientError` with descriptive print statements (not logging)
- **Object naming**: Use prefixed paths like `"experiment/test-upload.txt"` for organization
- **Region**: Always `'auto'` for R2 (Cloudflare requirement)

## Running the Project

```bash
# Install dependencies (uses uv)
uv sync

# Run the demo script
python main.py
```

The script auto-generates `test.txt`, uploads it, displays bucket contents, creates presigned URL, downloads to `downloaded_test.txt`, and cleans up `test.txt`.

## Development Patterns

1. **New operations**: Add as standalone functions following existing pattern (client init via `get_r2_client()`)
2. **Testing changes**: Modify `if __name__ == "__main__"` block to test new functionality
3. **File operations**: Always handle file cleanup in demo code (see `os.remove("test.txt")`)

## Assets

`public/` directory contains documentation images (overview.png, create.png, manage.png) - likely for README or external docs.

## Common Pitfalls

- R2 requires exact endpoint format with account ID
- Presigned URLs expire (default 3600s/1 hour) - adjust `expiration` parameter as needed
- boto3 treats R2 as S3 service, but region must be `'auto'`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZulfiFazhar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
