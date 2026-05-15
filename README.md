# Google Workspace Authenticator

A single-file Google service account authenticator for Google Workspace admin scripts. Drop `authenticator.py` into any project that needs delegated API access.

## Features

- Credential file resolution in four locations (env var → cwd → Linux server path → `Keys/`)
- Pre-built service constructors for every Workspace API: Gmail, Drive, Admin Directory, Calendar, Cloud Identity, Admin Reports, Sheets
- `GmailBatchAuthenticator` — loads credentials and the Gmail discovery doc once, then builds per-user services cheaply for concurrent batch operations (~700 fewer network calls vs. building each service from scratch)
- OAuth2 flow for Admin Reports (prompts browser on first run, caches `token.json`)
- `custom_api_build` for any other Google API

## Setup

```bash
pip install -r requirements.txt
cp .env.example .env   # fill in KEY_PATH and ADMIN_EMAIL
```

Place your service account key at the path set in `KEY_PATH`. The service account must have domain-wide delegation enabled with the appropriate OAuth scopes for each API you use.

## Usage

```python
from authenticator import admin_directory_v1_api, gmail_v1_api, GmailBatchAuthenticator

# Single delegated service
service = admin_directory_v1_api()
users = service.users().list(domain="yourdomain.com").execute()

# Gmail for a specific user
gmail = gmail_v1_api("user@yourdomain.com")

# Batch Gmail (efficient for processing many users)
auth = GmailBatchAuthenticator()
for email in user_list:
    svc = auth.get_service(email)
```

## Available builders

| Function | API |
|---|---|
| `admin_directory_v1_api(user)` | Admin Directory v1 |
| `gmail_v1_api(user)` | Gmail v1 |
| `drive_v3_api(user)` | Drive v3 |
| `drive_v2_api(user)` | Drive v2 |
| `calendar_v3_api(user)` | Calendar v3 |
| `cloud_identity_v1_api(user)` | Cloud Identity v1 |
| `admin_reports_v1_api()` | Admin Reports v1 (OAuth2) |
| `sheets_credentials()` | Sheets / gspread credentials |
| `custom_api_build(name, version, scopes, user)` | Any Google API |
| `GmailBatchAuthenticator` | Cached Gmail batch auth |
