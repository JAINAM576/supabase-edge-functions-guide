# Complete Guide: Supabase Edge Functions with Database Triggers

This guide walks you through setting up Supabase Edge Functions locally and deploying them with database triggers on Windows. Follow these steps to avoid common pitfalls and save time.

## Prerequisites
- Windows 10/11
- Internet connection
- Browser access to your Supabase project

---

## Step 1: Install Scoop Package Manager

Scoop is a command-line installer for Windows that makes installing developer tools easier.

### Installation Steps:
1. **Open PowerShell as Administrator**
   - Press `Win + X` and select "Windows PowerShell (Admin)"

2. **Set execution policy:**
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

3. **Install Scoop:**
   ```powershell
   irm get.scoop.sh | iex
   ```

4. **Verify installation:**
   ```powershell
   scoop --version
   ```

---

## Step 2: Install Supabase CLI

### Installation Steps:
1. **Open Command Prompt or PowerShell**

2. **Add Supabase bucket:**
   ```bash
   scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
   ```

3. **Install Supabase CLI:**
   ```bash
   scoop install supabase
   ```

4. **Verify installation:**
   ```bash
   supabase --version
   ```

---

## Step 3: Login to Supabase

### Authentication Steps:
1. **Open Command Prompt or PowerShell**

2. **Login to Supabase:**
   ```bash
   supabase login
   ```
   - This opens your browser
   - Authorize the CLI in your browser
   - Copy the verification code back to your terminal

---

## Step 4: Create Edge Function Locally

### Project Setup:
1. **Create project directory:**
   ```bash
   mkdir my-supabase-project
   cd my-supabase-project
   ```

2. **Initialize Supabase project:**
   ```bash
   supabase init
   ```

3. **Link to your Supabase project:**
   ```bash
   supabase link --project-ref YOUR_PROJECT_ID
   ```

   **To find your Project ID:**
   - Go to your Supabase Dashboard
   - Navigate to Project Settings → General
   - Copy the "Project ID"

4. **Create new Edge Function:**
   ```bash
   supabase functions new send-marketing-email
   ```

   This creates the following structure:
   ```
   /supabase/functions/send-marketing-email/
   ├── .npmrc
   ├── deno.json
   └── index.ts
   ```

---

## Step 5: Development and Deployment

### Environment Variables (Optional):
If you need to set environment variables:
```bash
supabase secrets set PROJECT_URL="your_project_url"
```
**⚠️ Important:** Don't prefix environment variable names with "SUPABASE" as it will cause errors.

### Local Development:
**Prerequisites:** Install Docker Desktop on your PC

1. **Run function locally:**
   ```bash
   supabase functions serve send-marketing-email --no-verify-jwt
   ```

2. **Test your function** at the local URL provided (usually `http://localhost:54321/functions/v1/send-marketing-email`)

### Deploy to Production:
```bash
supabase functions deploy send-marketing-email
```

---

## Step 6: Setup Database Triggers

### Enable Required Extensions:
Run these SQL commands in your Supabase SQL Editor:

```sql
CREATE EXTENSION IF NOT EXISTS pg_net;
CREATE EXTENSION IF NOT EXISTS http;
```

### Create Sample Table:
```sql
CREATE TABLE IF NOT EXISTS public.email_waitinglist (
   id SERIAL PRIMARY KEY,
   email_id TEXT,
   log_time TIMESTAMPTZ DEFAULT NOW()
);
```

### Create Trigger Function:
```sql
CREATE OR REPLACE FUNCTION handle_new_email()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM
    net.http_post(
      url := 'YOUR_EDGE_FUNCTION_URL',
      headers := jsonb_build_object(
        'Content-Type', 'application/json',
        'Authorization', 'Bearer YOUR_SERVICE_ROLE_KEY'
      ),
      body := jsonb_build_object('record', row_to_json(NEW))
    );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

**To get your Edge Function URL:**
- Go to your Supabase Dashboard
- Navigate to Edge Functions
- Click on your function name
- Copy the URL from the Details tab

**To get your Service Role Key:**
- Go to Project Settings → API
- Copy the "Service Role" key (not the anon key)

### Create the Trigger:
```sql
-- Remove existing trigger if it exists
DROP TRIGGER IF EXISTS email_inserted_trigger ON public.email_waitinglist;

-- Create new trigger
CREATE TRIGGER email_inserted_trigger
  AFTER INSERT ON public.email_waitinglist
  FOR EACH ROW
  EXECUTE FUNCTION public.handle_new_email();
```

### Enable Trigger (if needed):
```sql
ALTER TABLE public.email_waitinglist ENABLE ALWAYS TRIGGER email_inserted_trigger;
```

---

## Step 7: Testing and Debugging

### Test the Trigger:
```sql
INSERT INTO public.email_waitinglist (email_id) VALUES ('test@example.com');
```

### View Logs:
- Go to your Supabase Dashboard
- Navigate to Logs in the left sidebar
- Check for function execution logs and any errors

### Common Issues:
1. **Missing pg_net extension** - Run the extension commands above
2. **Authentication errors** - Verify your Service Role key is correct
3. **Network permissions** - Ensure your database can make HTTP requests
4. **Docker not running** - Start Docker Desktop for local development

---

## Quick Reference

### Key Commands:
- `supabase login` - Authenticate CLI
- `supabase functions new <name>` - Create new function
- `supabase functions serve <name>` - Run locally
- `supabase functions deploy <name>` - Deploy to production
- `supabase secrets set KEY=value` - Set environment variables

### Important URLs:
- **Edge Function URL:** Dashboard → Edge Functions → [Your Function] → Details
- **Service Role Key:** Dashboard → Project Settings → API → Service Role

### Troubleshooting:
- Check logs in Supabase Dashboard → Logs
- Verify Docker is running for local development
- Ensure all SQL extensions are installed
- Confirm authentication keys are correct

---

**🎉 Congratulations!** You now have a complete Supabase Edge Function setup with database triggers. Your function will automatically execute whenever new rows are inserted into your specified table.

---

## About

**Author:** Jainam Sanghavi

**Support:** If you face any issues or have questions about this guide, feel free to reach out at [sanghavijainam86@gmail.com](mailto:sanghavijainam86@gmail.com)
