# Roblox Open Cloud Assets API — Reference

Complete reference for uploading meshes and textures to Roblox via the Open Cloud Assets API.

## Table of Contents
1. Authentication
2. Upload a Mesh (FBX)
3. Upload a Texture (PNG)
4. Poll for Operation Completion
5. Extract Asset ID
6. Batch Upload Script
7. Error Codes and Troubleshooting
8. Rate Limits

---

## 1. Authentication

All requests require the `x-api-key` header with the Open Cloud API key.

```
x-api-key: YOUR_API_KEY
```

The Creator ID is the Roblox User ID (numeric). Find it from the profile URL: `roblox.com/users/CREATOR_ID/profile`

---

## 2. Upload a Mesh (FBX)

**Endpoint:** `POST https://apis.roblox.com/assets/v1/assets`

**Windows cmd:**
```cmd
curl -s -X POST "https://apis.roblox.com/assets/v1/assets" ^
  -H "x-api-key: %ROBLOX_API_KEY%" ^
  -F "request={\"assetType\":\"Model\",\"displayName\":\"MyMesh\",\"description\":\"Exported from Blender\",\"creationContext\":{\"creator\":{\"userId\":\"%ROBLOX_CREATOR_ID%\"}}};type=application/json" ^
  -F "fileContent=@C:\path\to\file.fbx;type=model/fbx"
```

**PowerShell:**
```powershell
$body = @{
    request = '{"assetType":"Model","displayName":"MyMesh","description":"Exported from Blender","creationContext":{"creator":{"userId":"' + $env:ROBLOX_CREATOR_ID + '"}}}'
}
# Use curl.exe (not Invoke-WebRequest alias)
curl.exe -s -X POST "https://apis.roblox.com/assets/v1/assets" `
  -H "x-api-key: $env:ROBLOX_API_KEY" `
  -F "request={`"assetType`":`"Model`",`"displayName`":`"MyMesh`",`"description`":`"From Blender`",`"creationContext`":{`"creator`":{`"userId`":`"$env:ROBLOX_CREATOR_ID`"}}};type=application/json" `
  -F "fileContent=@C:\path\to\file.fbx;type=model/fbx"
```

**Python (recommended for Claude Code — more reliable on Windows):**
```python
import requests
import os

api_key = os.environ["ROBLOX_API_KEY"]
creator_id = os.environ["ROBLOX_CREATOR_ID"]

url = "https://apis.roblox.com/assets/v1/assets"

request_payload = {
    "assetType": "Model",
    "displayName": "MyMesh",
    "description": "Exported from Blender",
    "creationContext": {
        "creator": {
            "userId": creator_id
        }
    }
}

import json
files = {
    "request": (None, json.dumps(request_payload), "application/json"),
    "fileContent": ("mesh.fbx", open("C:/path/to/file.fbx", "rb"), "model/fbx"),
}

headers = {"x-api-key": api_key}
response = requests.post(url, headers=headers, files=files)
result = response.json()
print(json.dumps(result, indent=2))
```

**Response (operation started):**
```json
{
  "path": "operations/some-operation-id",
  "done": false
}
```

---

## 3. Upload a Texture (PNG)

Same endpoint, different assetType and content type.

**Python:**
```python
request_payload = {
    "assetType": "Decal",
    "displayName": "MyTexture",
    "description": "Texture from Blender",
    "creationContext": {
        "creator": {
            "userId": creator_id
        }
    }
}

files = {
    "request": (None, json.dumps(request_payload), "application/json"),
    "fileContent": ("texture.png", open("C:/path/to/texture.png", "rb"), "image/png"),
}

response = requests.post(url, headers=headers, files=files)
result = response.json()
print(json.dumps(result, indent=2))
```

---

## 4. Poll for Operation Completion

After uploading, the API returns an operation ID. Poll until `done: true`.

```python
import time

operation_path = result.get("path", "")
# Extract operation ID
op_id = operation_path.split("/")[-1] if "/" in operation_path else operation_path

poll_url = f"https://apis.roblox.com/assets/v1/operations/{op_id}"

for attempt in range(30):
    time.sleep(2)
    poll_response = requests.get(poll_url, headers=headers)
    poll_data = poll_response.json()

    if poll_data.get("done"):
        print("Upload complete!")
        print(json.dumps(poll_data, indent=2))
        break
    else:
        print(f"Polling attempt {attempt + 1}... not done yet")
else:
    print("ERROR: Upload timed out after 60 seconds")
```

---

## 5. Extract Asset ID

When the operation completes, the response contains the asset ID.

```python
# From the completed poll response:
if poll_data.get("done"):
    # The response structure may be:
    response_data = poll_data.get("response", {})

    # Asset ID is in the assetId field or can be derived from the path
    asset_id = response_data.get("assetId")

    # If not directly available, check:
    # poll_data["response"]["path"] might be "assets/ASSET_ID"
    if not asset_id:
        asset_path = response_data.get("path", "")
        asset_id = asset_path.split("/")[-1] if "/" in asset_path else None

    if asset_id:
        roblox_uri = f"rbxassetid://{asset_id}"
        print(f"Asset ID: {asset_id}")
        print(f"Roblox URI: {roblox_uri}")
    else:
        print("WARNING: Could not extract asset ID from response")
        print(json.dumps(poll_data, indent=2))
```

---

## 6. Batch Upload Script

Full Python script to upload all files from the export manifest.

```python
import requests
import json
import time
import os

api_key = os.environ["ROBLOX_API_KEY"]
creator_id = os.environ["ROBLOX_CREATOR_ID"]
headers = {"x-api-key": api_key}
upload_url = "https://apis.roblox.com/assets/v1/assets"

# Load manifest
export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
manifest_path = os.path.join(export_dir, "manifest.json")
with open(manifest_path) as f:
    manifest = json.load(f)

results = []

for entry in manifest:
    name = entry["name"]
    asset_ids = {"name": name, "mesh_id": None, "texture_id": None}

    # Upload FBX
    if entry.get("fbx_path") and os.path.exists(entry["fbx_path"]):
        print(f"Uploading mesh: {name}...")
        payload = {
            "assetType": "Model",
            "displayName": name,
            "description": "Exported from Blender",
            "creationContext": {"creator": {"userId": creator_id}}
        }
        files = {
            "request": (None, json.dumps(payload), "application/json"),
            "fileContent": (f"{name}.fbx", open(entry["fbx_path"], "rb"), "model/fbx"),
        }
        resp = requests.post(upload_url, headers=headers, files=files)
        op_data = resp.json()

        # Poll
        op_id = op_data.get("path", "").split("/")[-1]
        poll_url = f"https://apis.roblox.com/assets/v1/operations/{op_id}"
        for i in range(30):
            time.sleep(2)
            pr = requests.get(poll_url, headers=headers).json()
            if pr.get("done"):
                aid = pr.get("response", {}).get("assetId")
                if not aid:
                    p = pr.get("response", {}).get("path", "")
                    aid = p.split("/")[-1] if "/" in p else None
                asset_ids["mesh_id"] = aid
                print(f"  Mesh uploaded: rbxassetid://{aid}")
                break
        else:
            print(f"  ERROR: Mesh upload timed out for {name}")

    # Upload texture
    if entry.get("texture_path") and os.path.exists(entry["texture_path"]):
        print(f"Uploading texture: {name}...")
        payload = {
            "assetType": "Decal",
            "displayName": f"{name}_texture",
            "description": "Texture from Blender",
            "creationContext": {"creator": {"userId": creator_id}}
        }
        files = {
            "request": (None, json.dumps(payload), "application/json"),
            "fileContent": (f"{name}.png", open(entry["texture_path"], "rb"), "image/png"),
        }
        resp = requests.post(upload_url, headers=headers, files=files)
        op_data = resp.json()

        op_id = op_data.get("path", "").split("/")[-1]
        poll_url = f"https://apis.roblox.com/assets/v1/operations/{op_id}"
        for i in range(30):
            time.sleep(2)
            pr = requests.get(poll_url, headers=headers).json()
            if pr.get("done"):
                aid = pr.get("response", {}).get("assetId")
                if not aid:
                    p = pr.get("response", {}).get("path", "")
                    aid = p.split("/")[-1] if "/" in p else None
                asset_ids["texture_id"] = aid
                print(f"  Texture uploaded: rbxassetid://{aid}")
                break
        else:
            print(f"  ERROR: Texture upload timed out for {name}")

    results.append(asset_ids)

# Save results
results_path = os.path.join(export_dir, "upload_results.json")
with open(results_path, 'w') as f:
    json.dump(results, f, indent=2)
print(f"\nAll uploads complete. Results: {results_path}")
print(json.dumps(results, indent=2))
```

---

## 7. Error Codes and Troubleshooting

| HTTP Code | Error | Cause | Fix |
|---|---|---|---|
| 401 | Unauthorized | Invalid or expired API key | Regenerate key |
| 403 | Forbidden | Key lacks Assets permission | Recreate with Assets read+write |
| 400 | InvalidAsset | FBX malformed or too large | Re-export, ensure triangulated, check file size under 50MB |
| 400 | InvalidImage | PNG corrupt or wrong format | Re-export as standard PNG (no interlacing) |
| 429 | TooManyRequests | Rate limited | Wait 60 seconds, retry |
| 500 | InternalError | Roblox server issue | Retry after 10 seconds |
| Moderation failure | Asset flagged | Name or content flagged | Rename to something generic, simplify mesh |

**Common gotchas:**
- File paths on Windows: use forward slashes in Python (`C:/Users/...`) or raw strings (`r"C:\Users\..."`)
- Large meshes (over 10K tris) may take longer to process — increase poll timeout
- Asset names cannot contain special characters — sanitize names before upload
- The API has a 50MB file size limit per upload

---

## 8. Rate Limits

- **Uploads:** ~60 requests per minute per API key
- **Polls:** No strict limit, but keep to 1 poll per 2 seconds per operation
- **Batch strategy:** Upload sequentially with a 1-second delay between uploads to avoid 429s
- If hit 429, back off exponentially (2s, 4s, 8s, etc.)
