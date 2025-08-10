---
title: "TeamCity: Docker Image Tag with Commit Hash and Environment"
date: 2025-03-31
categories:
  - Blog
tags:
  - Docker
  - TeamCity
---

### Creating Docker Tags with Git Commit Hash and Environment in TeamCity

When building Docker images in CI pipelines, a good practice is to use a combination of an environment variable and the Git commit hash to create image tags.

Here’s a simple example of how to set a custom image tag in **TeamCity** using a Bash script:

```bash
SHORT_COMMIT_HASH=$(echo %build.vcs.number% | cut -c1-7)
echo "##teamcity[setParameter name='img-tag' value='$SHORT_COMMIT_HASH-%deploy-env%']"
```

### Explanation

`%build.vcs.number%` — a built-in TeamCity variable that contains the current Git commit hash.

`cut -c1-7` — trims the hash to the first 7 characters (short hash).

`%deploy-env%` — your custom TeamCity parameter (e.g. `dev`, `stage`, `prod`).

The final image tag might look like: **a1b2c3d-dev**, **f4e5g6h-stage**, etc.

`##teamcity[setParameter ...]` — sets a new build parameter img-tag that you can use in subsequent build steps, such as Docker build and push.
