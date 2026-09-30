# Cattr User Manual

Screenshots and source PSD files are stored as regular Git files; the site does
not depend on Git LFS. The GitHub Pages workflow publishes the contents of
`docs/` to `docs.cattr.app` when changes reach `main`. Pull requests build the
site without deploying it.

## Run locally

```bash
npm ci
npm run serve
```
