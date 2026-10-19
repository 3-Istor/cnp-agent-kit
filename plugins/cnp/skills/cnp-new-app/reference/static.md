# Static app (template-html-css)

```
index.html          the page
styles/style.css    the styles
Dockerfile          nginx-unprivileged, serves on 8080 as a non-root user
deploy/values.yaml  deployment settings (platform-owned)
.github/workflows/ci.yml   builds and publishes the image on vX.Y.Z tags
ai-context.md       the template's own rules for assistants: read it
```

- Plain HTML5 and CSS, no framework, no build step. JavaScript only when the
  feature needs it, in a separate file under `scripts/`.
- New files must be copied into the image: the Dockerfile copies
  `index.html` and `styles/`. Adding `scripts/` or `images/` means a
  Dockerfile change, which is a sensitive path: keep everything under
  `styles/` when you can, or tell the user it needs their confirmation.
- Reuse CSS values through tokens in `:root`, keep the existing breakpoint
  (`max-width: 600px`), use semantic tags and `alt` on images.
- Images: small (under 200 KB), committed as files, or inline SVG.
- No form that stores answers: a static site has nowhere to keep them. Link
  to the company's form tool, or move to a fullstack app.
