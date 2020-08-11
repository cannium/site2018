---
title: Import privately hosted Go modules without HTTPS
date: 2020-07-31
---

This might be simple, but it took me hours to figure out the full solution, so I put it here for your reference.

Suppose your internal gitlab host has IP or URL `$GitHost`, then set environment variables below:
```
GO111MODULE="on"
GONOPROXY="$GitHost"
GONOSUMDB="$GitHost"
```
And configure git as:
```bash
git config --global url."git@$GitHost:".insteadOf "https://$GitHost/"
```

When parsing a package import path, say `your.internal.git.host/package`, Go would first try `https`, as `https://your.internal.git.host/package`. This would be caught by the git URL conversion rule, becoming `git@your.internal.git.host/package`, which is serviceable by gitlab via SSH (You should have your SSH keys configured). The "NOPROXY" and "NOSUMDB" tells Go don't try to access through proxies(defined by `GOPROXY`) and don't verify checksum from a global database(defined by `GOSUMDB`).

Somehow the still-ugly part is package names should be ended with `.git`, as defined by gitlab, but I think this is acceptable.