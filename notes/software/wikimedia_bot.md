---
title: 'Wikimedia Bot'
---

[Source Code](https://github.com/helle253/wikimedia-bot)

Python Project, implementing a simple bot that queries wikipedia's 'High Quality Image' tag and posts one to BlueSky, Twitter, and Mastodon.

- [Bsky](https://bsky.app/profile/wikimedia-bot.hellbhoy.net)
- [Twitter](https://twitter.com/wikimedia_bot)
- [Mastodon](https://brinjal.org/@wikimedia)

TODO:

- [x] I think the date filtering is a little borked still/not truly random?
- [x] Gotta figure out why it seems to be posting more and more sporadically (likely related to #1)
- [x] It looks like wikimedia is throwing back a 429 error, which is funky. We aren't sending many requests to download (BUT we might be sending a bunch as part of the lookup?)

It was because the User-Agent for image downloads wasn't descriptive enough - at some point, wikimedia probably changed their policy to better handle scrapers.
