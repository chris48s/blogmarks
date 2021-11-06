---
layout: post
title: Composite Actions vs Reusable Workflows
categories: ['github']
tags: ['github']
---

## Composite Actions vs Reusable Workflows

A few days after I blogged about GitHub [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action), GitHub launched another similar feature: [Reusable Workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows).

There is a lot of overlap between these features and there are certainly some tasks that could be accomplished with either. Simultaneously, there are some important differences that drive a bit of a wedge between them.

- A composite action is presented as one "step" when it is invoked in a workflow, even if the action yaml contains multiple steps. Invoking a reusable workflow presents each step separately in the summary output. This can make debugging a failed composite action run harder.

- Reusable workflows can use secrets, whereas a composite action can't. You have to pass secrets in as parameters. Reusable workflows are also always implicitly passed `secrets.GITHUB_TOKEN`. This is often convenient, but another way to see this tradeoff would be to say: If you're using a reusable workflow published by someone else, it can always read your `GITHUB_TOKEN` var with whatever scopes that is granted, which may not always be desirable. A composite action can only read what you explicitly pass it.

- Both can only take `string`, `number` or `boolean` as a param. Arrays are not allowed.

- Only a [subset of job keywords](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows#supported-keywords-for-jobs-that-call-a-reusable-workflow) can be used when calling a reusable workflow. This places some restrictions on how they can be used. To give an example, reusable workflows can't be used with a matrix but composite actions can, so

  ```yaml
  jobs:
    build:
      strategy:
        matrix:
          param: ['foo', 'bar']

      uses: chris48s/my-reusable-workflow/.github/workflows/reuse-me.yml@main
      with:
        param: ${% raw %}{{ matrix.param }}{% endraw %}
  ```

  will throw an error, but

  ```yaml
  jobs:
    build:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          param: ['foo', 'bar']
      steps:
        - uses: chris48s/my-shared-action@main
          with:
            param: ${% raw %}{{ matrix.param }}{% endraw %}
  ```

  is valid

- Steps in a composite action can not use `if:` conditions, although there are [workarounds](https://github.com/actions/runner/issues/834).

- A composite action is called as a job step so a job that calls a composite action can have other steps (including calling other composite actions). A job can only call one reusable workflow and can't contain other steps.
