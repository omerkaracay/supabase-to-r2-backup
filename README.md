# Supabase Backup to Cloudflare R2

Automated daily backups of Supabase databases to Cloudflare R2 storage, using GitHub Actions.

> **Status:** Scheduled backups are **DISABLED**. The workflow requires repository secrets to be configured before it can run. See [Required Secrets](#required-secrets) below. You can trigger a manual run from the Actions tab, but it will fail until secrets are added.

## How It Works

A GitHub Action runs the Supabase CLI to dump your database (roles, schema, and data), compresses it into a zip file, and uploads it to your Cloudflare R2 bucket.

The schedule (`0 2 * * *` UTC) is commented out in `.github/workflows/backup.yml` until secrets are configured. To re-enable, uncomment the schedule section.

## Required Secrets

This workflow will fail without these repository secrets. Go to Settings > Secrets and add:

| Secret | Value |
|--------|-------|
| `SUPABASE_DB_URL` | Your Supabase session pooler connection string |
| `CF_ACCESS_KEY_ID` | Cloudflare R2 API Access Key ID |
| `CF_SECRET_ACCESS_KEY` | Cloudflare R2 API Secret Access Key |
| `CF_BUCKET_NAME` | Your R2 bucket name |
| `CF_BUCKET_ENDPOINT` | Your R2 S3 endpoint URL |

Without these secrets, `pg_dump` falls back to a local socket connection which doesn't exist in the CI runner, causing immediate failure.

## Setup

### 1. Create an R2 Bucket

1. Go to [dash.cloudflare.com](https://dash.cloudflare.com) and select your account
2. Click **R2** in the left sidebar
3. Click **Create bucket**
4. Give it a name (e.g. `supabase-backups`)
5. Click **Create bucket**

#### Set Up Lifecycle Policy (Auto-Delete After 14 Days)

1. In the R2 dashboard, click on your bucket
2. Go to the **Settings** tab
3. Scroll to **Object lifecycle** and click **Create rule**
4. Give it a name (e.g. `delete-old-backups`)
5. Under **Action**, select **Delete**
6. Set **Prefix** to `backup-` (this targets only backup files)
7. Set **Days after object creation** to `14`
8. Click **Create rule**

Old backups will now be automatically deleted after 2 weeks, keeping storage costs low.

#### Get the S3 Endpoint

1. In the R2 dashboard, go to your bucket
2. Look for **S3 API** or **Bucket details**
3. Copy the endpoint URL (format: `https://<account-id>.r2.cloudflarestorage.com`)

### 2. Create R2 API Token

1. In the R2 dashboard, click **Manage R2 API Tokens**
2. Click **Create API token**
3. Give it a name (e.g. `backup-uploader`)
4. Set permissions to **Object Read & Write**
5. Scope it to your bucket (select **Selected buckets** and pick yours)
6. Click **Create API token**
7. **Save the Access Key ID and Secret Access Key immediately** — the secret key won't be shown again

### 3. Get Your Supabase Connection String

In your Supabase project:
- Settings > Database > Connection info
- Select **Session pooler** (port 5432)
- Copy the full PostgreSQL URI

> Use the session pooler URL, not the direct connection. Direct connections often resolve to IPv6 addresses that GitHub Actions runners can't reach.

### 4. Add Repository Secrets

In your GitHub repo, go to Settings > Secrets and add the secrets listed in [Required Secrets](#required-secrets) above.

### 5. Enable the Schedule

After adding secrets, uncomment the schedule in `.github/workflows/backup.yml`:

```yaml
on:
  workflow_dispatch:
  schedule:
    - cron: '0 2 * * *'
```

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

### "No such file or directory" / socket errors

The workflow secrets are not configured. Check that all 5 secrets are set in your repository settings. Without `SUPABASE_DB_URL`, `pg_dump` falls back to a local socket that doesn't exist.

## License

MIT
