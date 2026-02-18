# OmniBox API Reference

Full schema details from OpenAPI 3.0 spec.

## Authentication

- Type: API Key
- Header: `Authorization`
- Format: `Bearer <api-key>`

## Schemas

### OpenCreateResourceDto

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | yes | Resource name/title |
| content | string | yes | Content of the resource (supports #hashtags) |
| tag_ids | string[] | yes | Array of tag UUIDs |
| attrs | object | yes | Additional attributes/metadata (free-form) |
| skip_parsing_tags_from_content | boolean | yes | Skip auto-parsing #hashtags from content |

### OpenCollectRequestDto (gzip)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| url | string | yes | URL of the web page |
| title | string | yes | Title of the web page |
| parentId | string | yes | Parent resource ID (use root if unspecified) |
| html | binary (gzip) | yes | Gzip-compressed HTML file (multipart upload) |

### OpenCollectUrlRequestDto

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| url | string | yes | URL to collect content from |
| parentId | string | no | Parent resource ID |

### OpenAgentRequestDto

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string | yes | Question to ask the AI wizard |
| tools | object[] | yes | Tools available to the AI assistant |
| enable_thinking | boolean | yes | Enable reasoning mode |
| lang | enum | yes | `"简体中文"` or `"English"` |
| parent_message_id | string | yes | Parent message ID for threading |

### APIKeyInfoDto

| Field | Type | Description |
|-------|------|-------------|
| api_key | APIKeyResponseDto | API key details |
| namespace | object | Associated namespace |
| user | object | Associated user |

### APIKeyResponseDto

| Field | Type | Description |
|-------|------|-------------|
| id | string | API key ID |
| value | string | API key value |
| user_id | string | User ID |
| namespace_id | string | Namespace ID |
| attrs | object | Key attributes/settings |
| created_at | datetime | Creation timestamp |
| updated_at | datetime | Last update timestamp |

### CollectUrlResponseDto

| Field | Type | Description |
|-------|------|-------------|
| resource_id | string | ID of the created resource |

## Response Codes

| Code | Meaning |
|------|---------|
| 200 | Success (GET/DELETE) |
| 201 | Created (POST) |
| 400 | Bad request (missing required fields) |
| 401 | Invalid or missing API key |
| 403 | Insufficient permissions |
