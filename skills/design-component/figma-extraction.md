# Figma Extraction (Step 1 — MANDATORY)

Requires a **Figma MCP server that can execute plugin code** (a `figma_execute`-style tool). Tool names below (`figma_execute`, `figma_navigate`, `figma_capture_screenshot`) are examples — use your server's actual tool names.

Convert node IDs from URLs (`3353-3257`) to API format (`3353:3257`).

## Fast Path (Run First)

**`figma_execute`** with the traversal script below. One call returns full structure; works on private files (REST `figma_get_file_data` / `figma_take_screenshot` often 404).

### 1. Navigate (if new file/session)

```
toolName: "figma_navigate"
arguments: { url: "<full-figma-design-url>" }
```

### 2. Get Full Structure (primary source)

```
toolName: "figma_execute"
arguments: { code: "<script>", timeout: 15000 }
```

Use **`await figma.getNodeByIdAsync('<id>')`** only (sync `getNodeById` throws in dynamic-page mode).

### Minimal Traversal Script

Replace `NODE_ID` with the target node:

```js
const node = await figma.getNodeByIdAsync('NODE_ID');
if (!node) return { error: 'Node not found' };
async function getProps(n, depth) {
  if (depth > 4) return { name: n.name, type: n.type };
  const r = { name: n.name, type: n.type, id: n.id };
  if ('width' in n) { r.width = Math.round(n.width); r.height = Math.round(n.height); }
  if ('fills' in n) r.fills = n.fills;
  if ('cornerRadius' in n) r.cornerRadius = n.cornerRadius;
  if ('paddingLeft' in n) r.padding = { left: n.paddingLeft, right: n.paddingRight, top: n.paddingTop, bottom: n.paddingBottom };
  if ('itemSpacing' in n) r.itemSpacing = n.itemSpacing;
  if ('layoutMode' in n) r.layoutMode = n.layoutMode;
  if ('primaryAxisAlignItems' in n) r.primaryAxisAlignItems = n.primaryAxisAlignItems;
  if ('counterAxisAlignItems' in n) r.counterAxisAlignItems = n.counterAxisAlignItems;
  if ('characters' in n) { r.characters = n.characters; r.fontSize = n.fontSize; r.fontWeight = n.fontWeight; r.fontName = n.fontName; r.lineHeight = n.lineHeight; r.letterSpacing = n.letterSpacing; }
  if ('children' in n) { r.children = []; for (const c of n.children) r.children.push(await getProps(c, depth + 1)); }
  return r;
}
return await getProps(node, 0);
```

Extend `getProps` as needed (strokes, effects, opacity). Log: dimensions, spacing, typography, colors, radius -> map to tokens.

### 3. Visual Reference (optional)

```
toolName: "figma_capture_screenshot"
arguments: { nodeId: "<id>" }
```

## Fallback

- `figma_get_component` (metadata) when REST is available.
- If your Figma MCP server isn't connected, follow its setup steps (for plugin-based servers, enable the plugin under *Figma -> Plugins -> Development*).
