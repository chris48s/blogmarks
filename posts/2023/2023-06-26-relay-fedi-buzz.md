<!--
.. title: relay.fedi.buzz
.. slug: relay-fedi-buzz
.. date: 2023-06-26 00:00:00
.. tags: mastodon
.. category: 
.. link: 
.. description: 
.. type: text
-->

I self-host my own mastodon instance. I am [the only user on my server](https://fed.chris-shaw.dev/@chris) and I only follow a handful of accounts. This means I inhabit a somewhat weakly connected corner of the fediverse. For example, following a tag doesn't act as a useful way for me to discover new posts because it almost exclusively shows me posts from people I am already following, which I know about anyway.

This is a problem that is theoretically solved by ActivityPub Relays, although practically I was never really able to get a handle on which (if any) made sense for me to add to my instance.

That changed recently when I found out about [relay.fedi.buzz](https://relay.fedi.buzz/) which allows you to generate ad-hoc follow-only ActivityPub relays for specific tags or instances (the most useful of those being tags IMO). For example, if I want to follow the `#python` tag, I can add [https://relay.fedi.buzz/tag/python](https://relay.fedi.buzz/tag/python) to my instance's relays. That ingests a much wider range of posts featuring that tag into my federated timeline, including from instance I was not already federated with via my follows. Then following that tag from my personal account now allows me to discover new posts on that topic.
