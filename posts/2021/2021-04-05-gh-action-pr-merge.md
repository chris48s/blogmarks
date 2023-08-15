<!--
.. title: Running a GitHub action on pull request merge
.. slug: gh-action-pr-merge
.. date: 2021-04-05 00:00:00
.. tags: github,github
.. category: github
.. link: 
.. description: 
.. type: text
-->

Github workflows can be triggered by a variety of events. The `pull_request` event has 14 different [activity types](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#pull_request). I recently wanted to write an action that runs after a pull request is merged and I was surprised to find that `merged` is not one of the supported activity types. This is easy enough to work around using the following snippet, but a surprising omission.


```yaml
on:
  pull_request:
    types: [closed]

jobs:
  my-job:
    if: github.event.pull_request.merged == true
    steps: ...
```
