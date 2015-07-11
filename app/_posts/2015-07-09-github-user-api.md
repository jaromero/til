---
layout: post
title:  "The Github User Info API"
date:   2015-07-09 20:00:00
tags:   github json jekyll
---
**EDIT 15-07-10:** This is [rate-limited](https://developer.github.com/v3/#rate-limiting) to 60 unauthenticated requests per hour, so I've gone with something like the second approach.

Github has an API of sorts for getting information about users.

Well, it's not really an API, since it's just a json file: `https://api.github.com/users/<username>`

For example, this is what [mine](https://api.github.com/users/jaromero) shows:

```json
{
  "login": "jaromero",
  "id": 794031,
  "avatar_url": "https://avatars.githubusercontent.com/u/794031?v=3",
  "gravatar_id": "",
  "url": "https://api.github.com/users/jaromero",
  "html_url": "https://github.com/jaromero",
  "followers_url": "https://api.github.com/users/jaromero/followers",
  "following_url": "https://api.github.com/users/jaromero/following{/other_user}",
  "gists_url": "https://api.github.com/users/jaromero/gists{/gist_id}",
  "starred_url": "https://api.github.com/users/jaromero/starred{/owner}{/repo}",
  "subscriptions_url": "https://api.github.com/users/jaromero/subscriptions",
  "organizations_url": "https://api.github.com/users/jaromero/orgs",
  "repos_url": "https://api.github.com/users/jaromero/repos",
  "events_url": "https://api.github.com/users/jaromero/events{/privacy}",
  "received_events_url": "https://api.github.com/users/jaromero/received_events",
  "type": "User",
  "site_admin": false,
  "name": "Antonio Romero",
  "company": "",
  "blog": "https://plus.google.com/117253402437145808641/about",
  "location": "Mexico City",
  "email": "nsdragon@gmail.com",
  "hireable": false,
  "bio": null,
  "public_repos": 24,
  "public_gists": 7,
  "followers": 9,
  "following": 7,
  "created_at": "2011-05-17T18:05:50Z",
  "updated_at": "2015-07-02T18:37:58Z"
}
```

Actually, I found this while trying to figure out a different issue. We're looking at doing another of these jekyll blogs for our front-end team at [my company](http://vincoorbis.com), and I wanted to see what the best way of showing authorship might be.

[This is one solution I found](https://gist.github.com/ravasthi/1834570), which looks nice and self-contained.

Using the above json however, what I can do instead is add a custom `github-author` field in the front matter, and use that to obtain things like the author's avatar and his/her blog address and other such things.
