# Fiber Capture Snippets

Use these snippets in browser DevTools Console on any React page.

## Get Fiber From DOM Element

```js
function getFiberFromElement(el) {
  if (!el) return null;
  const key = Object.keys(el).find(
    (k) => k.startsWith("__reactFiber$") || k.startsWith("__reactInternalInstance$")
  );
  return key ? el[key] : null;
}
```

## Find Nearest Composite Fiber

```js
function getCompositeFiber(fiber) {
  let node = fiber;
  while (node) {
    const t = node.type;
    if (typeof t === "function" || (typeof t === "object" && t !== null)) return node;
    node = node.return;
  }
  return null;
}
```

## Safe Prop Serialization

```js
function safeSerialize(value, seen = new WeakSet(), depth = 0, maxDepth = 6) {
  if (value === null || value === undefined) return value;
  if (typeof value === "function") return "[Function]";
  if (typeof value === "symbol") return "[Symbol]";
  if (typeof value === "bigint") return String(value);
  if (typeof value !== "object") {
    if (typeof value === "string" && value.length > 500) return value.slice(0, 500) + "...[truncated]";
    return value;
  }
  if (depth >= maxDepth) return "[MaxDepth]";
  if (seen.has(value)) return "[Circular]";
  seen.add(value);

  if (Array.isArray(value)) {
    return value.slice(0, 50).map((v) => safeSerialize(v, seen, depth + 1, maxDepth));
  }

  const out = {};
  for (const [k, v] of Object.entries(value).slice(0, 100)) {
    out[k] = safeSerialize(v, seen, depth + 1, maxDepth);
  }
  return out;
}
```

## Stable Type Identity Key (Per Session)

```js
const __typeIds = new WeakMap();
let __typeSeq = 0;

function getTypeIdentity(type) {
  if (!type || (typeof type !== "function" && typeof type !== "object")) return String(type);
  if (!__typeIds.has(type)) __typeIds.set(type, `type_${++__typeSeq}`);
  return __typeIds.get(type);
}
```

## Capture One Element

```js
function captureElement(el) {
  const fiber = getCompositeFiber(getFiberFromElement(el));
  if (!fiber) return null;

  const componentType = fiber.type;
  const componentName = componentType?.displayName || componentType?.name || "Anonymous";
  const componentIdentity = getTypeIdentity(componentType);
  const source =
    typeof componentType === "function" ? componentType.toString().slice(0, 10000) : null;

  const ownerChain = [];
  let p = fiber.return;
  let depth = 0;
  while (p && depth < 40) {
    const t = p.type;
    const n = (t && (t.displayName || t.name)) || p.elementType || `tag_${p.tag}`;
    ownerChain.push(String(n));
    p = p.return;
    depth++;
  }

  const r = el.getBoundingClientRect();
  return {
    componentIdentity,
    componentName,
    props: safeSerialize(fiber.memoizedProps ?? {}),
    html: el.outerHTML.slice(0, 12000),
    ownerChain,
    source,
    rect: { x: r.x, y: r.y, width: r.width, height: r.height },
  };
}
```

## Capture By Selector

```js
function captureBySelector(selector) {
  return [...document.querySelectorAll(selector)].map(captureElement).filter(Boolean);
}
```

## Minimal Grouping Helper

```js
function groupByIdentity(captures) {
  return captures.reduce((acc, c) => {
    (acc[c.componentIdentity] ||= {
      componentIdentity: c.componentIdentity,
      componentName: c.componentName,
      examples: [],
    }).examples.push({
      props: c.props,
      html: c.html,
      ownerChain: c.ownerChain,
      rect: c.rect,
    });
    return acc;
  }, {});
}
```

Notes:
- Keep captures in one page session to preserve `fiber.type` identity mapping.
- Prefer semantic anchors (headings, labels, landmarks) when selecting elements.
- Record capture provenance separately (URL, timestamp, viewport).
