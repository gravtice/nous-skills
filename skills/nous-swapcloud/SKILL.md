---
name: nous-swapcloud
description: Upload a local file to SwapCloud (Tencent COS) and return a temporary signed download URL. Use when users need a shareable temporary link (临时下载链接), want to upload large content before passing URL to external APIs, or need help with COS permissions.
---

# SwapCloud Upload (CLI)

## Quick Start

**IMPORTANT:** If you rely on `.env.*` files, run commands from this skill's base directory so config can be loaded. If you pass runtime env vars (inline/export), working directory is not restricted.

```bash
# 0) Create local config file (first time only)
(cd "<SKILL_BASE_DIR>" && { test -f .env.local || touch .env.local; })

# 1) Edit `.env.local` and fill in required values (see "Supported Environment Variables"):
#    BACKEND, TENCENT_COS_REGION, TENCENT_COS_BUCKET, TENCENT_COS_SECRET_ID, TENCENT_COS_SECRET_KEY

# 2) Basic upload (URL expires in 1 hour)
(cd "<SKILL_BASE_DIR>" && npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "/path/to/file" --expires 3600)

# 3) With object auto-deletion after 7 days
(cd "<SKILL_BASE_DIR>" && npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "/path/to/file" --expires 3600 --object-ttl-days 7)
```

Runtime env vars example (no `.env.local` required):

```bash
BACKEND=TENCENT_COS \
TENCENT_COS_REGION=ap-guangzhou \
TENCENT_COS_BUCKET=your-bucket-name \
TENCENT_COS_SECRET_ID=your-secret-id \
TENCENT_COS_SECRET_KEY=your-secret-key \
npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "/path/to/file" --expires 3600
```

Replace `<SKILL_BASE_DIR>` with the actual "Base directory for this skill" path shown when this skill is loaded.

**Output handling:** Return only the signed URL. If output contains extra lines (for example package-manager warnings), extract the first `http(s)` URL and validate it before returning.

## Supported Environment Variables

### Required

- `BACKEND` (must be `TENCENT_COS`)
- `TENCENT_COS_REGION` (for example `ap-guangzhou`)
- `TENCENT_COS_BUCKET` (bucket name)
- `TENCENT_COS_SECRET_ID` (COS credential secret id)
- `TENCENT_COS_SECRET_KEY` (COS credential secret key)

### Optional

- `TENCENT_COS_PREFIX` (object key prefix inside bucket, no leading slash)

### Env file priority

- `.env.local > .env.production > .env.development > .env.test`
- Runtime env vars override values in `.env.*` files

## Workflow

1) Validate the input file exists.
   - `test -f "<filePath>"`

2) Upload and capture command output.
   - If using `.env.*` config, run from skill base directory.
   - `raw=$(cd "<SKILL_BASE_DIR>" && npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "<filePath>" --expires <seconds> 2>&1)`
   - With object TTL (days): add `--object-ttl-days <days>`

3) Extract and validate URL:
   - `url=$(printf '%s\n' "$raw" | grep -Eo 'https?://[^[:space:]]+' | head -n1)`
   - **Success**: if `url` is non-empty, return this URL only.
   - **Failure**: if `url` is empty, treat as upload failure and use the error output for troubleshooting.

## Common Scenarios

### Share a file as a temporary link

1) Upload the file.
2) Send the returned URL.

### Send large content to an external API (upload first, then pass URL)

When an external API has payload limits, create a file (JSON/text/log), upload it, then pass the signed URL to the API.

- Keep `--expires` small.
- Do not upload secrets/PII unless you understand the risk and access scope.

## TTL Notes

- Link TTL (seconds): set via `--expires` / `expiresInSeconds`.
- Object TTL (days): set via `--object-ttl-days` / `objectTtlDays`.
  - This only writes object tag `swapcloud_ttl_days=<days>`.
  - Auto-deletion requires a COS bucket lifecycle rule that matches that tag and deletes after `<days>` (COS lifecycle runs on a daily schedule, may be delayed).

## Troubleshooting

When upload fails, check the error message and provide appropriate guidance:

### `Unsupported BACKEND=""` or missing environment variables
Environment variables not configured. Tell the user:
> SwapCloud 云存储未配置。请在 skill 目录 (`<SKILL_BASE_DIR>`) 下创建或编辑 `.env.local`，
> 至少填写以下项：`BACKEND`、`TENCENT_COS_REGION`、`TENCENT_COS_BUCKET`、`TENCENT_COS_SECRET_ID`、`TENCENT_COS_SECRET_KEY`。

### `command not found` / install failure
Retry with official registry:
```bash
(cd "<SKILL_BASE_DIR>" && npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "<filePath>" --expires <seconds>)
```

### `AccessDenied` when opening the URL
The signing credential lacks `GetObject` permission. Tell the user to check their COS bucket permissions.

### Network or timeout errors
Retry the upload. If persistent, check network connectivity to Tencent COS.

## Examples

### Upload an image and get a shareable link
```bash
(cd "<SKILL_BASE_DIR>" && npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "/tmp/screenshot.png" --expires 7200)
# Output: https://bucket.cos.region.myqcloud.com/path/to/file?sign=...
```

### Upload a large JSON file for API consumption
```bash
# Create the file first
echo '{"data": [...]}' > /tmp/payload.json

# Upload and get URL (expires in 5 minutes)
(cd "<SKILL_BASE_DIR>" && npx -y --registry=https://registry.npmjs.org/ @gravtice/nous-swapcloud upload --file "/tmp/payload.json" --expires 300)
```
