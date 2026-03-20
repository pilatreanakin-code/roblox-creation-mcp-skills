# Setup Guide — Roblox Open Cloud API Key

This guide walks the user through creating a Roblox Open Cloud API key for programmatic asset uploads.

## What is it?

The Roblox Open Cloud API lets external tools (like Claude Code) upload meshes and textures directly to Roblox's servers without manually dragging files into Studio. The API key authenticates these requests.

## Step-by-Step Setup

### 1. Find your Roblox Creator ID

1. Go to https://www.roblox.com/users/profile
2. Look at the URL — it will be something like `https://www.roblox.com/users/123456789/profile`
3. The number (`123456789`) is your Creator ID
4. Save this — you'll need it

### 2. Create the API Key

1. Go to https://create.roblox.com/dashboard/credentials
2. If you don't see "API Keys" in the left sidebar, go directly to: https://create.roblox.com/dashboard/credentials?activeTab=ApiKeysTab
3. Click **Create API Key**
4. Fill in:
   - **Name**: `BlenderExport` (or any name you want)
   - **Description**: optional
5. Under **Access Permissions**:
   - Click **Add API System**
   - Select **Assets**
   - Enable: **Read** and **Write**
6. Under **Accepted IP Addresses**:
   - Add `0.0.0.0/0` (this allows requests from any IP — simplest for local development)
   - If you want to restrict it later, add your specific public IP instead
7. Under **Expiration**:
   - Choose "No Expiration" for convenience, or set a date
8. Click **Save and Generate Key**
9. **COPY THE KEY IMMEDIATELY** — it will only be shown once. If you lose it, you'll need to create a new one.

### 3. Set Environment Variables

In a terminal (or Claude Code), run:

**Windows (Command Prompt):**
```cmd
set ROBLOX_API_KEY=your_api_key_here
set ROBLOX_CREATOR_ID=your_creator_id_here
```

**Windows (PowerShell):**
```powershell
$env:ROBLOX_API_KEY = "your_api_key_here"
$env:ROBLOX_CREATOR_ID = "your_creator_id_here"
```

**Mac/Linux:**
```bash
export ROBLOX_API_KEY="your_api_key_here"
export ROBLOX_CREATOR_ID="your_creator_id_here"
```

To make these permanent, add them to your shell profile (~/.bashrc, ~/.zshrc) or Windows Environment Variables.

### 4. Test the Key

Run this to verify the key works:
```bash
curl -s -X GET "https://apis.roblox.com/cloud/v2/users/YOUR_CREATOR_ID" \
  -H "x-api-key: YOUR_API_KEY"
```

If you get a JSON response with user info, the key works. If you get a 401 or 403, double-check the key and permissions.

## Troubleshooting

| Problem | Solution |
|---|---|
| "Invalid API Key" | Copy the key again — make sure no trailing spaces |
| "Insufficient permissions" | Re-create the key with Assets read+write |
| "IP not allowed" | Add `0.0.0.0/0` to accepted IPs |
| Can't find API Keys page | Make sure you're logged in and are 13+ with a verified account |
| Key disappeared | Keys are shown once. Create a new one. |

## Security Notes

- Never share your API key publicly (GitHub, Discord, etc.)
- The `0.0.0.0/0` IP allowance is fine for local dev but restrict it for production
- You can revoke a key anytime from the same dashboard
- Each key can be scoped to specific permissions — only grant what you need
