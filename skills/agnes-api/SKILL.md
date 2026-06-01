---
name: agnes-api
description: Call Agnes AI APIs for image generation, image editing, text-to-video, image-to-video, multi-image video, and keyframe animation. Uses API key from ~/.config/agnes-ai/api_key without printing it.
allowed-tools:
  - Bash
---
# Agnes AI

Use this skill when the user wants to generate images or videos with Agnes AI / Sapiens AI.

Base URL:

```text
https://apihub.agnes-ai.com/v1
```

API key location:

```text
~/.config/agnes-ai/api_key
```

Never print the API key. Always load it into an environment variable:

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
```

## Models

- Image: `agnes-image-2.1-flash`
- Video: `agnes-video-v2.0`

## 1. Image Generation: Text-to-Image

Use this for drawing / creating an image from text.

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "A luminous floating city above a misty canyon at sunrise, cinematic realism, rich details",
    "size": "1024x768",
    "extra_body": {
      "response_format": "url"
    }
  }'
```

Response shape usually includes:

```json
{
  "data": [
    {
      "url": "https://...png"
    }
  ]
}
```

Extract URL:

```bash
python3 - << 'PY'
import json, sys
j=json.load(sys.stdin)
print(j["data"][0]["url"])
PY
```

## 2. Image Editing: Image-to-Image

Use this when the user provides an input image URL and wants to transform/edit it.

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "Transform the scene into a rain-soaked cyberpunk night with neon reflections while preserving the original composition",
    "size": "1024x768",
    "extra_body": {
      "image": ["https://example.com/input-image.png"],
      "response_format": "url"
    }
  }'
```

## 3. Video Generation: Text-to-Video

Video generation is async.

Create task:

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "A cinematic shot of a cat walking on the beach at sunset, soft ocean waves, warm golden lighting, realistic motion",
    "width": 1152,
    "height": 768,
    "num_frames": 121,
    "frame_rate": 24
  }'
```

Response includes `task_id` or `id`.

Poll task:

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
TASK_ID="task_xxx"
curl -s "https://apihub.agnes-ai.com/v1/videos/$TASK_ID" \
  -H "Authorization: Bearer $API_KEY"
```

When `status` is `completed`, read the generated video URL from `video_url`. If `video_url` is missing, fall back to `remixed_from_video_id` — Agnes sometimes returns the final mp4 there.

## 4. Image-to-Video

Use this when animating one image.

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "Animate the character with subtle breathing motion, hair moving gently in the wind, cinematic camera movement, keep face and outfit consistent",
    "image": "https://example.com/image.png",
    "num_frames": 121,
    "frame_rate": 24
  }'
```

## 5. Multi-Image Video

Use multiple input images to guide the video.

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "Create a smooth transformation scene between the two reference images, cinematic lighting, consistent character identity, natural motion",
    "extra_body": {
      "image": [
        "https://example.com/image1.png",
        "https://example.com/image2.png"
      ]
    },
    "num_frames": 121,
    "frame_rate": 24
  }'
```

## 6. Keyframe Animation

Use this for interpolation between keyframes.

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "Generate a smooth cinematic transition between the keyframes, maintaining visual consistency and natural camera movement",
    "extra_body": {
      "image": [
        "https://example.com/keyframe1.png",
        "https://example.com/keyframe2.png"
      ],
      "mode": "keyframes"
    },
    "num_frames": 121,
    "frame_rate": 24
  }'
```

## Parameter Notes

Image parameters:

- `model`: required, `agnes-image-2.1-flash`
- `prompt`: required
- `size`: optional, e.g. `512x512`, `1024x768`
- `extra_body.image`: optional array for image-to-image
- `extra_body.response_format`: optional, use `url`

Video parameters:

- `model`: required, `agnes-video-v2.0`
- `prompt`: required
- `image`: optional string URL for image-to-video
- `width`: optional, default 1152
- `height`: optional, default 768
- `num_frames`: optional, must be `8n + 1` and <= 441, examples: 81, 121, 161, 241, 441
- `frame_rate`: optional, 1-60
- `negative_prompt`: optional
- `seed`: optional
- `extra_body.image`: optional array for multi-image or keyframes
- `extra_body.mode`: optional, use `keyframes` for keyframe animation

## Safer Shell Pattern

When running commands, avoid echoing secrets. Do not use `set -x`.

For quick text-to-image testing:

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
RESPONSE=$(curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "A cute corgi astronaut floating in space, cinematic lighting",
    "size": "512x512",
    "extra_body": {"response_format": "url"}
  }')
printf '%s\n' "$RESPONSE" | python3 -c 'import json,sys; j=json.load(sys.stdin); print(j["data"][0]["url"])'
```

For video polling loop:

```bash
API_KEY=$(cat ~/.config/agnes-ai/api_key)
TASK_ID="task_xxx"
for i in {1..60}; do
  RESPONSE=$(curl -s "https://apihub.agnes-ai.com/v1/videos/$TASK_ID" \
    -H "Authorization: Bearer $API_KEY")
  echo "$RESPONSE" | python3 -c 'import json,sys; j=json.load(sys.stdin); print(j.get("status"), j.get("progress"), j.get("video_url") or j.get("remixed_from_video_id"))'
  STATUS=$(echo "$RESPONSE" | python3 -c 'import json,sys; print(json.load(sys.stdin).get("status"))')
  [ "$STATUS" = "completed" ] && break
  [ "$STATUS" = "failed" ] && break
  sleep 10
done
```

For a cleaner final URL extractor after polling:

```bash
printf '%s\n' "$RESPONSE" | python3 -c 'import json,sys; j=json.load(sys.stdin); print(j.get("video_url") or j.get("remixed_from_video_id") or "")'
```
