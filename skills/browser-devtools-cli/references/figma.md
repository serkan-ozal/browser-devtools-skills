# Figma Tools

Design comparison and validation commands.

## Prerequisites

- Figma API access token (environment variable `FIGMA_ACCESS_TOKEN`)
- Cloud configuration for image/text embeddings (optional, for semantic comparison)

## compare-page-with-design

Compare live page UI against a Figma design.

```bash
browser-devtools-cli figma compare-page-with-design --figma-file-key <key> --figma-node-id <id> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--figma-file-key` | string | Yes | - | Figma file key (from URL: part after `/file/`) |
| `--figma-node-id` | string | Yes | - | Figma node id (frame/component, e.g. `12:34`) |
| `--selector` | string | No | - | CSS selector to compare only a region (omit = full page) |
| `--full-page` | boolean | No | `true` | Compare full scrollable page; ignored when selector set |
| `--mssim-mode` | enum | No | `semantic` | MSSIM mode: `raw` (pixel-strict) or `semantic` (layout-oriented) |
| `--weights` | object | No | - | JSON: `{"mssim":0.4,"imageEmbedding":0.3,"textEmbedding":0.3}` for signal weights |
| `--figma-scale` | number | No | - | Figma raster export scale (e.g. 1, 2) |
| `--figma-format` | enum | No | - | Figma export format: `png` or `jpg` |
| `--max-dim` | number | No | - | Max dimension for comparison images |
| `--jpeg-quality` | number | No | - | JPEG quality 50–100 for preprocessing |

**Comparison Modes:**

- **`semantic`**: More tolerant of text/data differences, focuses on layout and visual structure
- **`raw`**: Pixel-level comparison, expects near-identical output

**Examples:**

```bash
# Compare viewport with Figma frame (file key from URL part after /file/, node id e.g. 1:2)
browser-devtools-cli figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:2"

# Compare specific component (JSON for parsing)
browser-devtools-cli --json figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:2" \
  --selector "#header"

# Raw comparison for static designs
browser-devtools-cli --json figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:2" \
  --mssim-mode raw

# Full page comparison
browser-devtools-cli --json figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:2" \
  --full-page

# Custom weights (focus on layout) — pass as JSON object
browser-devtools-cli --json figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:2" \
  --weights '{"mssim":0.7,"imageEmbedding":0.2,"textEmbedding":0.1}'
```

**Output (JSON):** Returns a combined similarity score and notes describing which signals were used. Notes explain skipped signals (e.g. missing cloud config).

```json
{
  "score": 0.87,
  "notes": [
    "MSSIM score indicates good structural similarity",
    "Minor text differences detected"
  ],
  "meta": {
    "pageUrl": "https://...",
    "pageTitle": "...",
    "figmaFileKey": "...",
    "figmaNodeId": "1:2",
    "selector": null,
    "fullPage": true,
    "pageImageType": "png",
    "figmaImageType": "png"
  }
}
```

## Design QA Workflow

```bash
# Set Figma API token
export FIGMA_ACCESS_TOKEN="your-token-here"

SESSION="--session-id design-qa --json"

# Navigate to implementation
browser-devtools-cli $SESSION navigation go-to --url "http://localhost:3000/dashboard"
browser-devtools-cli $SESSION sync wait-for-network-idle

# Compare header
HEADER_RESULT=$(browser-devtools-cli $SESSION figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:header" \
  --selector "header")
echo "Header: $HEADER_RESULT"

# Compare hero section
HERO_RESULT=$(browser-devtools-cli $SESSION figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:hero" \
  --selector ".hero-section")
echo "Hero: $HERO_RESULT"

# Compare full page
PAGE_RESULT=$(browser-devtools-cli $SESSION figma compare-page-with-design \
  --figma-file-key "xxx" --figma-node-id "1:page" \
  --full-page)
echo "Full page: $PAGE_RESULT"

# Screenshot for reference
browser-devtools-cli $SESSION content take-screenshot --name "implementation" --full-page
```

## CI/CD Design Validation

```bash
#!/bin/bash
set -e

export FIGMA_ACCESS_TOKEN="$FIGMA_TOKEN"
CLI="browser-devtools-cli --json --quiet --session-id design-ci-$$"

# Navigate
$CLI navigation go-to --url "$APP_URL"
$CLI sync wait-for-network-idle

# Compare with design (set FIGMA_FILE_KEY and FIGMA_NODE_ID from your design URL)
RESULT=$($CLI figma compare-page-with-design \
  --figma-file-key "$FIGMA_FILE_KEY" --figma-node-id "$FIGMA_NODE_ID" \
  --mssim-mode semantic)

# Check score (0–1, higher = more similar)
SCORE=$(echo $RESULT | jq '.score')
echo "Design comparison: score=$SCORE"

if [ "$(echo $RESULT | jq '.score >= 0.8')" != "true" ]; then
  echo "Design mismatch detected!"
  echo "Details: $RESULT"
  exit 1
fi

echo "Design validation passed (score=$SCORE)"
```

## Similarity Signals

| Signal | Description | Weight (default) |
|--------|-------------|------------------|
| MSSIM | Structural similarity (layout, shapes) | 0.4 |
| Image Embedding | Visual feature similarity | 0.3 |
| Text Embedding | Semantic text similarity | 0.3 |

**Weight Configurations:** Use `--weights` with a JSON object (keys: `mssim`, `imageEmbedding`, `textEmbedding`).

```bash
# Focus on layout
--weights '{"mssim":0.7,"imageEmbedding":0.2,"textEmbedding":0.1}'

# Focus on visual appearance
--weights '{"mssim":0.3,"imageEmbedding":0.5,"textEmbedding":0.2}'

# Focus on content
--weights '{"mssim":0.2,"imageEmbedding":0.2,"textEmbedding":0.6}'
```

## Comparison Modes

### Semantic Mode (Default)

Use when:
- Comparing designs with dynamic/real data
- Testing layout and visual structure
- Allowing for text content differences

```bash
browser-devtools-cli --json figma compare-page-with-design \
  --figma-file-key "FILE_KEY" --figma-node-id "1:2" \
  --mssim-mode semantic
```

### Raw Mode

Use when:
- Comparing static designs
- Expecting pixel-perfect match
- Testing specific visual details

```bash
browser-devtools-cli --json figma compare-page-with-design \
  --figma-file-key "FILE_KEY" --figma-node-id "1:2" \
  --mssim-mode raw
```

## Troubleshooting

### Low Similarity Scores

1. **Check viewport size:** Ensure browser viewport matches Figma frame size

```bash
# Set viewport to match Figma
browser-devtools-cli interaction resize-viewport --width 1440 --height 900
```

2. **Wait for content:** Use `sync wait-for-network-idle` before comparing

3. **Check selector:** Ensure selector targets the correct element

4. **Mode selection:** Use `semantic` mode for dynamic content

### Missing Signals

- Image/text embedding may be skipped without cloud configuration
- Check `notes` field in output for skipped signals
- MSSIM is always available locally

### Figma Access Issues

- Verify `FIGMA_ACCESS_TOKEN` environment variable is set
- Check Figma URL format includes `node-id` parameter
- Ensure you have access to the Figma file
