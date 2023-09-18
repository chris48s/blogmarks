<!--
.. title: Querying GitHub Repos in Bulk
.. slug: bulk-querying-github
.. date: 2023-09-18 00:00:00
.. tags: github
.. category: 
.. link: 
.. description: 
.. type: text
-->

I've recently been working on a tool called [pip-abandoned](https://github.com/chris48s/pip-abandoned), which allows you to search for abandoned and deprecated python packages.

In order to make this, one of the things I needed to do was fetch the `archived` property for lots of GitHub repositories. Using the GitHub v3 (rest) API, this would need one round-trip per repo. So if I want to fetch the `archived` property for

- [https://github.com/pygments/pygments](https://github.com/pygments/pygments)
- [https://github.com/pycqa/mccabe](https://github.com/pycqa/mccabe)
- [https://github.com/readthedocs/commonmark.py](https://github.com/readthedocs/commonmark.py)

then I need to make three API calls:

- [https://api.github.com/repos/pygments/pygments](https://api.github.com/repos/pygments/pygments)
- [https://api.github.com/repos/pycqa/mccabe](https://api.github.com/repos/pycqa/mccabe)
- [https://api.github.com/repos/readthedocs/commonmark.py](https://api.github.com/repos/readthedocs/commonmark.py)

..and if I had a list of 200 GitHub repos, that would require 200 individual network requests.

Fortunately, GitHub also has a GraphQL API. Using the GraphQL API, we can query more than one repo in a single request. To fetch the same data as above using the GraphQL API, I can make the following query

```graphql
query {
  pygments: repository(
    owner: "pygments", name: "pygments"
  ) {
    isArchived
  }
  mccabe: repository(
    owner: "pycqa", name: "mccabe"
  ) {
    isArchived
  }
  commonmark: repository(
    owner: "readthedocs", name: "commonmark.py"
  ) {
    isArchived
  }
}
```

to retrieve the `archived` property for those three repos in a single round-trip over the network. When querying a large number of repos this is a big benefit.
