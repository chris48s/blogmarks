<!--
.. title: Editing Commit Timestamps in Git
.. slug: commit-timestamps
.. date: 2020-06-30 00:00:00
.. tags: terminal,git
.. category: 
.. link: 
.. description: 
.. type: text
-->

I always forget that [GitHub shows commits in a pull request in timestamp order](https://help.github.com/en/github/committing-changes-to-your-project/why-are-my-commits-in-the-wrong-order) rather than tree/history order until just after I've just pushed a monster rebase and everything's in the wrong order. To force GitHub to show the commits in the correct order, we need to edit the commit timestamps to match the history order. To do this:

```bash
git rebase -i HEAD~N
```

Mark the commits that need re-ordering as `edit`


```text
edit f3b9e40 Reticulate Splines
edit 68f39b8 Adjust Bell Curves
edit d605e5a Dice Models
```

At each stage of the rebase:

```bash
git commit --amend --date=now --no-edit
git rebase --continue
```

Now the timestamp ordering will match the history. When you force-push, GitHub will order the commits correctly in your pull request.
