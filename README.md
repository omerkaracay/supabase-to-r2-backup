# Supabase Backup to Cloudflare R2

Automated daily backups of Supabase databases to Cloudflare R2 storage, using GitHub Actions.

## How It Works

Every day at 2:00 AM UTC, a GitHub Action runs the Supabase CLI to dump your database (roles, schema, and data), compresses it into a zip file, and uploads it to your Cloudflare R2 bucket.

## Setup

### 1. Create an R2 Bucket

Go to the Cloudflare dashboard and create an R2 bucket for your backups. Note:
- Bucket name
- S3 endpoint URL

### 2. Create R2 API Token

In Cloudflare R2, create an API token with **Object Read & Write** permissions scoped to your bucket. Save the Access Key ID and Secret Access Key.

### 3. Get Your Supabase Connection String

In your Supabase project:
- Settings > Database > Connection info
- Select **Session pooler** (port 5432)
- Copy the full PostgreSQL URI

> Use the session pooler URL, not the direct connection. Direct connections often resolve to IPv6 addresses that GitHub Actions runners can't reach.

### 4. Add Repository Secrets

In your GitHub repo, go to Settings > Secrets and add:

| Secret | Value |
|--------|-------|
| `SUPABASE_DB_URL` | Your Supabase session pooler connection string |
| `CF_ACCESS_KEY_ID` | Cloudflare R2 API Access Key ID |
| `CF_SECRET_ACCESS_KEY` | Cloudflare R2 API Secret Access Key |
| `CF_BUCKET_NAME` | Your R2 bucket name |
| `CF_BUCKET_ENDPOINT` | Your R2 S3 endpoint URL |

### 5. Done

The workflow runs automatically on a daily schedule. You can also trigger it manually from the Actions tab.

## Backup Output

Each backup is a zip file containing:
- `roles.sql` - Database roles and permissions
- `schema.sql` - Full database schema
- `data.sql` - All table data

Files are named `backup-YYYY-MM-DD.zip` and stored in your R2 bucket.

## Troubleshooting

### "Network is unreachable" / IPv6 errors

Your Supabase connection string uses a direct connection (`db.<ref>.supabase.co`) that resolves to an IPv6 address. Switch to the **session pooler** URL in your Supabase project settings.

### "Wrong password" errors

- Verify your database password is correct
- If you forgot it, reset it in Supabase Settings > Database
- After resetting, copy the full session pooler connection string (it includes the new password)
- Update the `SUPABASE_DB_URL` secret

## License

MIT
