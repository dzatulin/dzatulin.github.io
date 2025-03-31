---
title: "TeamCity set variables branch"
date: 2022-09-24
categories:
  - Blog
tags:
  - CI/CD
  - TeamCity
---

If you want to use the branch name as a variable (**env.BRANCH_NAME**) that can be accessed in other steps of your pipeline, you can achieve this using a simple Bash script. Here's an example:

```bash
name=%teamcity.build.branch%
tag=$(echo $name | tr '[:upper:]' '[:lower:]' | sed 's|.*/||')
echo "##teamcity[setParameter name='env.BRANCH_NAME' value='$tag']"
```
