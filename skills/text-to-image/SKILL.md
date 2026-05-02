---
name: text-to-image
description: |
  Generates images from text descriptions using the Token Plan text-to-image model.
  Use this skill when asked to "draw a picture," "generate an image," or "create an image based on a description,"
  such as "生成一张图片" or "根据描述创建图像" or "draw a cat wearing a hat" or "generate a night view of a futuristic city."
  Accepts plain text prompts and outputs images.
mode: subagent
version: 1.1.0
tools:
  bash: true
  write: false
  edit: false
---

Generate an image by calling the Token Plan text-to-image API based on the text description.

## Execution Steps

**Step-0**: Check that the environment variable `$TOKEN_PLAN_API_KEY` exists. Keep it secret. If not set, stop and prompt the user to configure it.

**Step-1**: Extract parameters from user input:

- `prompt` (required): The image description.
- `model` (optional, default: `qwen-image-2.0-pro`): See Available Models.
- `size` (optional, default: `2688*1152`): Image dimensions in `WIDTH*HEIGHT` format. See Available Sizes.
- `asratio` (optional): Aspect ratio like `16:9`, `9:16`, `1:1`. If provided, convert to `size` using the Aspect Ratio table. Takes precedence over `size`.

**Step-2**: Execute the curl command to generate the image:

```bash
RESPONSE=$(curl -s -X POST "https://token-plan.cn-beijing.maas.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation" \
  -H "Authorization: Bearer $TOKEN_PLAN_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"<model>","input":{"messages":[{"role":"user","content":[{"text":"<prompt>"}]}]},"parameters":{"size":"<size>"}}')
echo "$RESPONSE"|jq -r '{code, message, images: [.output.choices[]?.message.content[]?.image]}'
```

**Step-3**: Extract the `jq` result — `code` is the error code, `message` is the error description, `images` is a list of image URLs from `output.choices[*].message.content[*].image`.

- If `code` is not null, stop and tell the user the error with `message`.
- If `images` has entries, download them one by one to the current directory.
- If the user has explicitly specified a save path, save there instead.

**Result Example**

_**Successful response**_

```json
{
  "code": null,
  "message": null,
  "images": [
    "https://dashscope-7c2c.oss-accelerate.aliyuncs.com/..."
  ]
}
```

_**Failed response — invalid parameter**_

```json
{
  "code": "InvalidParameter",
  "message": "Image area must be between 262144 (512x512) and 4194304 (2048x2048) pixels.",
  "images": []
}
```

**Step-4**: Return the file path(s) of the downloaded image(s).

## Aspect Ratio

When the user provides an aspect ratio, convert it to the corresponding `size`. `asratio` takes precedence over `size` — if both are given, use the ratio.

| Aspect Ratio | Size | Best For |
|-------------|------|----------|
| `2.35:1` | `2688*1152` | Ultrawide cinematic (default) |
| `1:1` | `1024*1024` | Square format |
| `1:1:high` | `2048*2048` | High-resolution square |
| `16:9` | `2688*1536` | Landscape / widescreen |
| `9:16` | `1536*2688` | Mobile / vertical |
| `4:3` | `2368*1728` | Standard landscape |
| `3:4` | `1728*2368` | Standard portrait |
| `21:9` | `2368*1024` | Ultrawide banner |

## Available Sizes

Can also be specified directly in `WIDTH*HEIGHT` format:

`1024*1024`, `720*1280`, `1280*720`, `2048*2048`, `2688*1536`, `1536*2688`, `2368*1728`, `1728*2368`, `2368*1024`, `2688*1152`

## Available Models

| Model | Description |
|-------|-------------|
| `qwen-image-2.0` | General-purpose, versatile, proficient in Chinese text rendering |
| `qwen-image-2.0-pro` | Higher quality (default) |
| `wan2.7-image` | Multiple styles, defaults to 4 images |
| `wan2.7-image-pro` | Supports 4K resolution |

## Prompt Writing Tips

The qwen-image models handle detailed prompts well. For best results:

- **Be specific about subjects**: describe the main subject clearly — appearance, pose, expression, clothing.
- **Describe the scene**: background, lighting, weather, time of day, camera angle.
- **Specify the art style**: photorealistic, oil painting, watercolor, anime, pixel art, 3D render, etc.
- **Mention color palette**: e.g., "暖色调" (warm tones), "冷色调蓝紫色" (cool blue-purple tones).
- **Include composition hints**: close-up, wide shot, bird's-eye view, centered, rule of thirds.
- **State the mood**: dreamy, dramatic, serene, vibrant, moody.

A good image prompt is typically 50-200 words, rich in visual detail.
