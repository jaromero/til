---
layout: post
title:  "Sass Guidelines"
date:   2015-07-10 20:00:00
tags:   scss lint guidelines
---
[SCSS-Lint](https://github.com/brigade/scss-lint) is a nice tool that I'm looking to integrate into my workflow. This is additional to trying to come up with coding guidelines for our projects. Actually, SCSS-Lint's [rules](https://github.com/brigade/scss-lint/blob/master/lib/scss_lint/linter/README.md) are very similar to the rules I was thinking of.

The one thing I would really like though is to have it as an npm plugin (to use with gulp), but having it run in the editor itself is probably a better idea.

Meanwhile, one of my coworkers ([@gatero](https://github.com/gatero)) shared these [Sass Guidelines](http://sass-guidelin.es/) by [Hugo Giraudel](http://hugogiraudel.com/). These are also very similar, and even a config for SCSS-Lint is provided.

I might just take these guidelines and configs and dump them into the [yeoman generator](https://github.com/jaromero/generator-gulp-coffee-webapp) I've been working on. It's something I'd also like to finish (and maybe rename) for our team to use as a standard starting point.
