---
layout: post
title: Vendoring Ansible Galaxy Roles
categories: ['terminal']
tags: ['terminal', 'ansible']
---

## Vendoring Ansible Galaxy Roles

By default `ansible-galaxy` installs all roles in a single global location. If you use ansible a lot, this can lead to a situation where you have a conflict because one project requires version X of a dependency and another requires version Y. I usually guard against this by setting each of my ansible projects up to install dependencies in a local vendor directory inside the project (the NPM/composer model) using the [roles-path setting](https://galaxy.ansible.com/docs/using/installing.html#determining-where-roles-are-installed).

This can be specified on-the-fly with a flag or env var, but I usually set this in my `ansible.cfg` before creating a `requirements.yml`.

```ini
[defaults]
roles_path = ./vendor/
```
