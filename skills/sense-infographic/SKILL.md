---
name: sense-infographic
description: |
  Generates infographic images using the SenseNova U1 Fast model, specialized for information-rich visual content.
  Use this skill when asked to create infographics, information diagrams, data visualizations,
  flowchart images, process diagrams, or any structured visual representation of information,
  such as "生成一张信息图" or "制作信息图" or "create an infographic about X" or "generate a data visualization"
  or "画一个流程图" or "制作知识图谱" or "生成海报" or "信息可视化".
  Excellent at rendering text-heavy visuals — posters, educational graphics, comparison charts, and step-by-step guides.
  Also use when the user wants to visualize complex information with Chinese text in the image.
mode: subagent
version: 1.0.0
tools:
  bash: true
  write: false
  edit: false
---

Generate infographic images by calling the SenseNova U1 Fast API. This model is purpose-built for information graphics — it handles long structured prompts and renders Chinese text within images exceptionally well.

## Execution Steps

**Step-0**: Check that the environment variable `$SENSE_API_KEY` exists. Keep it secret. If not set, stop and prompt the user to configure it.

**Step-1**: Extract parameters from user input:

- `prompt` (required): The infographic description. U1 Fast supports prompts up to 4096 tokens, so be generous with detail — describe the layout, sections, text content, colors, icons, and visual style.
- `size` (optional, default: `2752x1536`): Image dimensions in `WIDTHxHEIGHT` format. See Available Sizes.
- `aspect_ratio` (optional): Aspect ratio like `16:9`, `9:16`, `1:1`. If provided, convert to `size` using the Aspect Ratio table. Takes precedence over `size`.
- `n` (optional, default: `1`): Number of images to generate.

**Step-2**: Execute the curl command to generate the image:

```bash
RESPONSE=$(curl -s -X POST "https://token.sensenova.cn/v1/images/generations" \
  -H "Authorization: Bearer $SENSE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"sensenova-u1-fast","prompt":"<prompt>","size":"<size>","n":<n>}')
echo "$RESPONSE" | jq -r '{error: .error.message, images: [.data[].url]}'
```

**Step-3**: Extract the `jq` result — `error` is the error message text, `images` is a list of image URLs from `data[].url`.

- If `error` is not null, stop and tell the user the error message.
- If `images` has entries, download them one by one to the current directory.
- If the user has explicitly specified a save path, save there instead.

**Result Example**

_**Successful response**_

```json
{
  "error": null,
  "images": [
    "https://cdn.sensenova.dev/gen/..."
  ]
}
```

_**Failed response — invalid parameter**_

```json
{
  "error": "field Size invalid, should be one of: 1664x2496, 2496x1664, ...",
  "images": []
}
```

_**Failed response — auth failure**_

```json
{
  "error": "Forbidden",
  "images": []
}
```

**Step-4**: Return the file path(s) of the downloaded image(s).

## Aspect Ratio

When the user provides an aspect ratio, convert it to the corresponding `size`:

| Aspect Ratio | Size | Best For |
|-------------|------|----------|
| `16:9` | `2752x1536` | Landscape infographic (default) |
| `9:16` | `1536x2752` | Mobile / vertical infographic |
| `1:1` | `2048x2048` | Square format |
| `3:2` | `2496x1664` | Standard landscape |
| `2:3` | `1664x2496` | Standard portrait |
| `4:3` | `2368x1760` | Wide landscape |
| `3:4` | `1760x2368` | Tall portrait |
| `5:4` | `2272x1824` | Near-square landscape |
| `4:5` | `1824x2272` | Near-square portrait |
| `21:9` | `3072x1376` | Ultrawide banner |
| `9:21` | `1344x3136` | Ultra-tall strip |

## Available Sizes

Can also be specified directly in `WIDTHxHEIGHT` format (lowercase `x`):

`2752x1536`, `1536x2752`, `2048x2048`, `2496x1664`, `1664x2496`, `2368x1760`, `1760x2368`, `2272x1824`, `1824x2272`, `3072x1376`, `1344x3136`

## Prompt Writing Tips

U1 Fast is designed for information graphics and renders text (including Chinese) inside images with high fidelity. To get the best results:

- **Describe the layout**: specify whether sections flow left-to-right, top-to-bottom, in a grid, or along a timeline.
- **Specify color themes**: e.g., "以柔和的粉色、淡黄色和浅蓝色为主色调" (soft pink, light yellow, and light blue palette).
- **Include all text verbatim**: every label, heading, and caption you want rendered should appear in the prompt exactly as you want it.
- **Mention visual elements**: icons, illustrations, decorative elements (e.g., "配有可爱的拟人化图标").
- **State the visual style**: flat design, hand-drawn, minimalist, corporate, etc.

The richer and more structured the prompt, the better the output. A good infographic prompt is often 200-800 words long.
