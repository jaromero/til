---
layout: post
title:  "Switching Markdown renderers"
date:   2015-07-08 20:00:00
categories: update jekyll markdown
---
By default, jekyll uses the [kramdown](http://kramdown.gettalong.org/) parser for Markdown files. I haven't actually tried it yet as standalone, but one thing I couldn't seem to get it to work with right, is GFM-style code blocks:

    ```coffee
    console.log 'Hello, everyone!'
    bgColor = '#ccf'
    body = document.getElementsByTagName('body')[0]
    body.style.backgroundColor = bgColor
    ```

This doesn't work out of the box. Instead, these are required:

```
{{ "{%" }} highlight coffee %}
console.log 'Hello, everyone!'
bgColor = '#ccf'
body = document.getElementsByTagName('body')[0]
body.style.backgroundColor = bgColor
{{ "{%" }} endhighlight %}
```

But this means that GFM syntax plugins in editors don't quite work.

After a quick google search, however, I found [this post](http://ajoz.github.io/2014/06/29/i-want-my-github-flavored-markdown/) that suggests using [redcarpet](https://github.com/vmg/redcarpet) instead, which actually works (except for the fact that jekyll still attempts to parse liquid tags).

So now I can have fenced code blocks, Github style, with syntax highlighting on the resulting jekyll build:

```coffee
console.log 'Hello, everyone!'
bgColor = '#ccf'
body = document.getElementsByTagName('body')[0]
body.style.backgroundColor = bgColor
```

    ```coffee
    console.log 'Hello, everyone!'
    bgColor = '#ccf'
    body = document.getElementsByTagName('body')[0]
    body.style.backgroundColor = bgColor
    ```

(Additionally, fencing a code block with `~` would also be acceptable, and GFM also supports it.)

As an additional fun fact, I had to write the second code block above like this, to avoid liquid parsing:

```
{{ "{{" }} {{ '"{%"' }} }} highlight coffee %}
console.log 'Hello, everyone!'
bgColor = '#ccf'
body = document.getElementsByTagName('body')[0]
body.style.backgroundColor = bgColor
{{ "{{" }} {{ '"{%"' }} }} endhighlight %}
```

[Jekyll is _greedy_ when it comes to parsing](http://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags).

Of course, we always have the option of [embedding Github gists](https://github.com/blog/122-embedded-gists).
