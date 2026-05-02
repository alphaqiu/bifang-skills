# Bifang Skills

AI-powered image generation skills for Claude Code.

## Skills

### sense-infographic

Generate infographic images using the SenseNova U1 Fast model. Supports information diagrams, data visualizations, flowcharts, posters, and other structured visual content. Renders Chinese text within images with high fidelity.

**Trigger examples:**
- `生成一张信息图`
- `create an infographic about X`
- `制作海报`
- `画一个流程图`

**Requirements:** Set the `SENSE_API_KEY` environment variable with your SenseNova API key.

### text-to-image

Generate images from text descriptions using the Token Plan text-to-image model. Supports multiple aspect ratios, art styles, and the qwen-image model family (including Chinese text rendering).

**Trigger examples:**
- `生成一张图片`
- `draw a cat wearing a hat`
- `generate a night view of a futuristic city`
- `根据描述创建图像`

**Requirements:** Set the `TOKEN_PLAN_API_KEY` environment variable with your Token Plan API key.

## Installation

use claude code 

```bash
claude plugin add https://github.com/alphaqiu/bifang-skills
```

use skill cli

```bash
npx skills add alphaqiu/bifang-skills
```

## License

MIT
