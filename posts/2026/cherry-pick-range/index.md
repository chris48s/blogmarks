<!--
.. title: Cherry-picking a range of commits in git
.. slug: cherry-pick-range
.. date: 2026-01-27 00:00:00
.. tags: terminal,git
.. category: 
.. link: 
.. description: 
.. type: text
-->

Sometimes it is useful to cherry pick multiple commits in a single operation.

The most frequent way I do this is by cherry-picking a range using the syntax

```
git cherry-pick abc123^..def456
```

This applies that range of commits in order. The `^` is important because git ranges are exclusive on the left and inclusive on the right.

- `abc123..def456` applies commits **after** `abc123`, up to and including `def456`
- `abc123^..def456` applies commits starting with `abc123`, up to and including `def456`

The caret tells git to include the first commit.

I usually check that

```
git log --oneline abc123^..def456
```

shows the list of commits I expect before running the cherry-pick.

I use this one less often, but it is also possible to cherry-pick multiple individual commits in one command. For example:

```
git cherry-pick 7f3a9c2 b81e4d6 d4e8b73
```

Git applies the commits in the order listed, left to right. This is useful when you want to pick a non-continuous range or you want to re-order while picking.
