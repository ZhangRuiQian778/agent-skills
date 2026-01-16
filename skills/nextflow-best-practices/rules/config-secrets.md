---
title: Keep Secrets Out of Versioned Config
impact: MEDIUM-HIGH
impactDescription: Reduces credential leakage
tags: config, secrets, security
---

## Keep Secrets Out of Versioned Config

Use Nextflow secrets or environment variables rather than hardcoding secrets in config or params. Secrets should not be assigned to pipeline parameters.

**Incorrect (hardcoded secret):**

```nextflow
params.api_key = 'super-secret-token'
```

**Correct (use secrets or env vars):**

```groovy
// nextflow.config
aws {
  accessKey = secrets.MY_ACCESS_KEY
  secretKey = secrets.MY_SECRET_KEY
}
```

**Process usage (inject as env vars):**

```nextflow
process UPLOAD {
  secret 'MY_ACCESS_KEY'
  secret 'MY_SECRET_KEY'

  script:
  """
  upload --access \$MY_ACCESS_KEY --secret \$MY_SECRET_KEY
  """
}
```

Reference: Nextflow secrets documentation
