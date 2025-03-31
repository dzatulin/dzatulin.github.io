---
title: "TeamCity: Auto-tagging Builds via REST API"
date: 2025-03-31
categories:
  - Blog
tags:
  - CI/CD
  - TeamCity
---

### Auto-tagging TeamCity Builds Using REST API

Tagging builds in TeamCity can be helpful for identifying deployments, grouping builds by environment, or tracking release versions. You can automate this process by using TeamCity’s REST API and a simple Bash script.

Here’s how to assign a tag (e.g. the target environment) to the current build programmatically.

```bash
JSON_BODY=$(cat <<EOF
{
  "tag": [
    {"name": "%deploy-env%"}
  ]
}
EOF
)
echo "JSON Body: $JSON_BODY"

curl -X POST "%teamcity.serverUrl%/app/rest/builds/%teamcity.build.id%/tags" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer %teamcity.access.token%" \
     -d "$JSON_BODY"
```
