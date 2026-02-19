# Style Parity Snippets

Use these snippets when structural reconstruction is correct but visuals drift.

## Capture Computed Styles for Anchor Nodes

```js
function collectStyles(map, keys) {
  const out = {};
  for (const [name, sel] of Object.entries(map)) {
    const el = document.querySelector(sel);
    if (!el) {
      out[name] = null;
      continue;
    }
    const cs = getComputedStyle(el);
    const r = el.getBoundingClientRect();
    out[name] = { rect: { w: r.width, h: r.height, x: r.x, y: r.y } };
    for (const k of keys) out[name][k] = cs.getPropertyValue(k);
  }
  return out;
}

const keys = [
  "padding",
  "margin",
  "border",
  "border-radius",
  "background-color",
  "backdrop-filter",
  "box-shadow",
  "gap",
  "font-size",
  "line-height",
  "letter-spacing",
  "font-weight",
  "color",
  "max-width",
  "min-height",
];

const anchors = {
  container: ".component-root",
  title: "h1",
  subtitle: ".subtitle",
  ctaGroup: ".ctas",
  primaryButton: ".primary",
  controls: ".controls",
  activeTab: ".tab.active",
};

console.log(collectStyles(anchors, keys));
```

## Quick Diff Helper (Run in Node or Console)

```js
function diffStyles(target, clone, epsilon = 0.5) {
  const diffs = [];
  for (const key of Object.keys(target)) {
    if (!target[key] || !clone[key]) {
      diffs.push({ node: key, issue: "missing-node" });
      continue;
    }
    for (const prop of Object.keys(target[key])) {
      if (prop === "rect") continue;
      if (target[key][prop] !== clone[key][prop]) {
        diffs.push({ node: key, prop, target: target[key][prop], clone: clone[key][prop] });
      }
    }
    if (target[key].rect && clone[key].rect) {
      const rw = Math.abs(target[key].rect.w - clone[key].rect.w);
      const rh = Math.abs(target[key].rect.h - clone[key].rect.h);
      if (rw > epsilon || rh > epsilon) {
        diffs.push({ node: key, prop: "rect", target: target[key].rect, clone: clone[key].rect });
      }
    }
  }
  return diffs;
}
```

## Check Font Parity (Generic)

```js
const probe = document.querySelector("h1, h2, [data-title], .title") || document.body;
const family = getComputedStyle(probe).fontFamily;
const primary = family.split(",")[0].trim().replace(/^['\"]|['\"]$/g, "");

console.log({
  primaryFamily: primary,
  regularLoaded: document.fonts.check(`400 16px "${primary}"`),
  mediumLoaded: document.fonts.check(`500 16px "${primary}"`),
  semiboldLoaded: document.fonts.check(`600 16px "${primary}"`),
  bodyFont: getComputedStyle(document.body).fontFamily,
});
```

If checks fail, host required font files locally (for example under `public/fonts/`) and use local `@font-face` URLs.

## Extract Relevant Stylesheet Rules

1. In Network tools, filter requests to stylesheets.
2. Save relevant CSS responses.
3. Build class tokens from target subtree:

```js
const nodes = document.querySelectorAll(".component-root, .component-root *");
const classes = [...new Set([...nodes].flatMap((n) => [...n.classList]))];
console.log(classes);
```

4. Build a grep pattern from captured class tokens, then search saved CSS.
5. Port matched rules to local classnames/selectors before manual tuning.

## Visual-Diff Checklist

- Container side padding and max-width match at desktop and mobile breakpoints.
- Typography metrics match (font family, size, line height, letter spacing, weight).
- Glass/translucent surfaces match (background alpha, border, blur, shadow).
- Interactive state styles match (active/inactive tabs, submitted/open sections, disabled states).
- CTA shell and spacing match (padding, border radius, icon geometry, label offsets).
