<!--
.. title: GitHub Composite Actions
.. slug: composite-actions
.. date: 2021-09-12 00:00:00
.. tags: github,github
.. category: github
.. link: 
.. description: 
.. type: text
-->

GitHub quietly sneaked out a new feature in the last few weeks: [Composite Actions](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action). This solves one of my biggest issues with GH Actions which is that yaml workflow steps couldn't be re-used. This means I have lots of repos with the exact same yaml workflow, or roughly the same workflow with slight variations. For example these 3 repos (and others) currently use the exact same workflow for CI:

* [datapackage-to-datasette](https://github.com/chris48s/datapackage-to-datasette/blob/1db8e836f588d0d1b65fbef56bcfa4ccc79d871b/.github/workflows/test.yml)
* [commitment](https://github.com/chris48s/commitment/blob/fa634b9e6c4291052af5160421274c194d5eef3e/.github/workflows/test.yml)
* [datasette-mailto-links](https://github.com/chris48s/datasette-mailto-links/blob/705aae608f839fbdff787971957291ec01f30d3f/.github/workflows/test.yml)

Composite actions essentially allows yaml workflow steps to be packaged and re-used in the same way that javascript actions or docker actions can be, so I can package this up as a shared action and use it in lots of repos. This is a feature I have been hoping would be added to Actions for some time and I'm looking forward to DRYing up my workflows across repos over the coming weeks.
