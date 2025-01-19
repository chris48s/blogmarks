<!--
.. title: Three lessons from moving to Renovate
.. slug: moto-lambda-logs
.. date: 2025-01-19 00:00:00
.. tags: python,dependencies
.. category: 
.. link: 
.. description: 
.. type: text
-->

At work recently I have been working on migrating some python projects using UV for package management from dependabot to renovate. Renovate is really impressive software. It can do _all the things_. However with that capability comes some complexity. Whereas dependabot is a go kart, renovate is more like an aeroplane cockpit. Although I've used renovate before, the projects I've been migrating recently are a bit bigger with a few more moving parts.

Here's a random grab-bag of things I have recently learned that were not immediately obvious to me.

## Order of packageRules evaluation

Renovate evaluates `packageRules[]` from top to bottom. The order they are declared in is important. If a package matches a rule, there’s no bail-out or "early return". It keeps going down the array. This means if a package matches multiple rules, later rules override earlier rules in the array.

To give a worked example:

If I have

```toml
[project]
dependencies = [
    "boto3==1.35.98",
    "botocore==1.35.98",
]
```

In my `pyproject.toml`

This renovate config

```json
"packageRules": [
    {
        "matchManagers": ["pep621"],
        "matchDepTypes": ["project.dependencies"],
        "schedule": ["before 4am on monday"], // weekly
    },
    {
        "groupName": "boto",
        "matchManagers": ["pep621"],
        "matchPackageNames": ["/^boto.*$/"],
        "schedule": ["* 0-3 1 * *"], // monthly
    }
]
```

Will bump the two boto packages **monthly**, whereas the same rules in the opposite order

```json
"packageRules": [
    {
        "groupName": "boto",
        "matchManagers": ["pep621"],
        "matchPackageNames": ["/^boto.*$/"],
        "schedule": ["* 0-3 1 * *"], // monthly
    },
    {
        "matchManagers": ["pep621"],
        "matchDepTypes": ["project.dependencies"],
        "schedule": ["before 4am on monday"], // weekly
    }
]
```

will bump them **weekly** because the packages match both rules and the second one "wins".

This means that you broadly want to order your `packageRules` from the most general at the start of the array to most specific at the end.

## VCS Dependencies

UV allows you to specify VCS dependencies in a couple of ways.

One of them is to do it inline in `dependencies` using PEP-508 syntax e.g:

```toml
[project]
dependencies = [
    "arcgis2geojson @ git+https://github.com/chris48s/arcgis2geojson.git@3.0.3"
]
```

The other us to use UV [dependency sources](https://docs.astral.sh/uv/concepts/projects/dependencies/#dependency-sources) e.g:

```toml
[project]
dependencies = [
    "arcgis2geojson"
]

[tool.uv.sources]
arcgis2geojson = { git = "https://github.com/chris48s/arcgis2geojson.git", tag = "3.0.3" }
```

Renovate's UV manager natively knows how to bump VCS dependencies in `pyproject.toml`. However, it will only detect and bump dependencies which are specified with the dependency sources syntax. VCS dependencies specified inline will be ignored, so the syntax you use here matters.

Additionally, the specific syntax you use inside `tool.uv.sources` is also important. These two declarations are essentially identical in terms of what gets installed into your virtual environment:

```toml
[tool.uv.sources]
arcgis2geojson = { git = "https://github.com/chris48s/arcgis2geojson.git", rev = "3.0.3" }
```

```toml
[tool.uv.sources]
arcgis2geojson = { git = "https://github.com/chris48s/arcgis2geojson.git", tag = "3.0.3" }
```

However, renovate will interpret them and bump them differently. Using `rev` here will cause renovate to bump you to the latest commit every time a commit is pushed to the referenced repository, whereas using `tag` will only offer to upgrade the dependency when a new tag is pushed.

## Pre-Commit

I'm going to go on record and say I don't particularly like pre-commit. It is not something I use on my own projects. I do encounter on other people's repositories a lot though, including some of the repos at work. As such, although it is something I prefer not to use, I do need to find coping mechanisms for it. One of my least favourite things about pre-commit is that unless you use repository-local hooks (which is relatively uncommon), pre-commit pushes you towards using custom "hook" repos. This leads to a situation where you end up with the version numbers of your tools duplicated in your package manifest and your `.pre-commit.yaml`. You now no longer have a single source of truth for this information and the two version numbers can end up out of sync.

Usefully, renovate has a [pre-commit manager](https://docs.renovatebot.com/modules/manager/pre-commit/) which can bump version numbers in `.pre-commit.yaml`. This means you can write a `packageRule` like:

```json
{
    "groupName": "ruff",
    "matchManagers": [
        "pep621",
        "pre-commit"
    ],
    "matchPackageNames": [
        "ruff",
        "astral-sh/ruff-pre-commit"
    ],
}
```

for example, which will bump the version of ruff in `pyproject.toml`/`uv.lock` and `.pre-commit.yaml` in a single PR, keeping the two in sync.

Note that renovate's pre-commit manager is **disabled by default** and you must explicitly opt-in to it, as noted in the [docs](https://docs.renovatebot.com/modules/manager/pre-commit/#enabling).
