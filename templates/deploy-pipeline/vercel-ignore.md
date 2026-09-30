# Vercel ignore command

Add this to `vercel.json` to restrict deploys to `preview` and `main` only.
Feature branch pushes skip the build (exit 0); production branches build (exit 1).

```json
{
  "ignoreCommand": "if [ \"$VERCEL_GIT_COMMIT_REF\" = \"main\" ] || [ \"$VERCEL_GIT_COMMIT_REF\" = \"preview\" ]; then exit 1; else exit 0; fi"
}
```

Merge this into the project's existing `vercel.json` alongside any headers, rewrites, or other config.

When the hosting target moves off Vercel, replace with the equivalent CI/CD branch gate.
