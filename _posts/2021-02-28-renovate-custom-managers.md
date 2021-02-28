---
layout: post
title: Renovate Custom Managers
categories: ['dependencies']
tags: ['dependencies']
---

## [Renovate Custom Managers](https://docs.renovatebot.com/modules/manager/regex/)

Tools like Dependabot and PyUp are amazing for automatically keeping dependencies updated. They work fine, as long as your dependencies are distributed via a supported language-specific package manager and specified in a manifest file. This doesn't account for 100% of application dependencies though, particularly when it comes to deployment. This is where Renovate has a killer feature: [Custom Managers](https://docs.renovatebot.com/modules/manager/regex/). The setup is a little fiddly, but this allows Renovate to submit pull requests to arbitrary files like a `Dockerfile` or an ansible playbook bumping applications like nginx, curl or Postgres which aren't specified in a `requirements.txt` or a `package.json`.
