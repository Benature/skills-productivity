---
name: omnibox-api
description: Interact with OmniBox API to create resources, upload files, collect web content from URLs, and ask questions to the AI wizard. Use when the user mentions OmniBox, omnibox, collecting URLs, saving web pages, or querying the OmniBox assistant.
---

# OmniBox API

Base URL: `https://api.omnibox.pro`

Auth: `Authorization: Bearer <API_KEY>` header on all requests.

## Endpoints Overview

| Method | Path | Description |
|--------|------|-------------|
| GET | /v1/api-keys/info | Get API key info |
| DELETE | /v1/api-keys | Delete API key |
| POST | /v1/resources | Create a resource/document |
| POST | /v1/resources/upload | Upload a file as resource |
| POST | /v1/wizard/collect/gzip | Collect web content (gzip HTML) |
| POST | /v1/wizard/collect/url | Collect content from a URL |
| POST | /v1/wizard/ask | Ask AI wizard a question |

## Common Usage

### Create a Resource

```bash
curl -X POST 'https://api.omnibox.pro/v1/resources' \
  -H 'Authorization: Bearer $OMNIBOX_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "My Document",
    "content": "Document content here. #tag1 #tag2",
    "tag_ids": [],
    "attrs": {},
    "skip_parsing_tags_from_content": false
  }'
```

Response: `{ "id": "...", "name": "..." }`

### Upload a File

```bash
curl -X POST 'https://api.omnibox.pro/v1/resources/upload' \
  -H 'Authorization: Bearer $OMNIBOX_API_KEY' \
  -F 'file=@/path/to/file.pdf' \
  -F 'parsed_content=Optional pre-parsed text'
```

Response: `{ "id": "...", "name": "..." }`

### Collect from URL

Simplest way to save a web page:

```bash
curl -X POST 'https://api.omnibox.pro/v1/wizard/collect/url' \
  -H 'Authorization: Bearer $OMNIBOX_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "url": "https://example.com/article",
    "parentId": "optional-parent-resource-id"
  }'
```

Response: `{ "resource_id": "..." }`

### Collect with Gzip HTML

For saving a page with its full HTML content:

```bash
echo '<html>...</html>' | gzip > /tmp/html.gz

curl -X POST 'https://api.omnibox.pro/v1/wizard/collect/gzip' \
  -H 'Authorization: Bearer $OMNIBOX_API_KEY' \
  -F 'url=https://example.com/page' \
  -F 'title=Page Title' \
  -F 'html=@/tmp/html.gz;type=application/gzip;filename=html.gz'
```

### Ask the AI Wizard

```bash
curl -X POST 'https://api.omnibox.pro/v1/wizard/ask' \
  -H 'Authorization: Bearer $OMNIBOX_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "What are the main topics in my recent articles?",
    "tools": [],
    "enable_thinking": true,
    "lang": "简体中文",
    "parent_message_id": ""
  }'
```

`lang` options: `"简体中文"` | `"English"`

## Implementation Notes

- Use environment variable `OMNIBOX_API_KEY` for the API key. Never hardcode.
- `parentId` in collect endpoints is optional; omit to use root resource.
- `tag_ids` accepts UUID strings for existing tags.
- Content supports `#hashtag` syntax for auto-tagging (disable with `skip_parsing_tags_from_content: true`).
- For detailed schemas, see [reference.md](reference.md).
